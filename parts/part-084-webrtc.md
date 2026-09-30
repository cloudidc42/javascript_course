# Part 84: WebRTC (Web Real-Time Communication)
## Steps 1651-1670

WebRTC เป็น API ที่ช่วยให้ browsers สามารถสื่อสารกันแบบ peer-to-peer (P2P) ได้โดยตรง
รองรับ video, audio, และการส่งข้อมูลโดยไม่ต้องผ่าน server กลาง

---

## Step 1651: What is WebRTC

```
WebRTC (Web Real-Time Communication) คืออะไร?

WebRTC เป็น open-source project ที่ให้ browsers และ mobile apps มีความสามารถ:
- Real-time audio communication
- Real-time video communication  
- Data transfer แบบ peer-to-peer

ใช้ใน:
- Video conferencing (Zoom, Google Meet)
- Online gaming
- File sharing
- Live streaming
- IoT applications
- Remote desktop

ข้อดีของ P2P:
✓ Latency ต่ำ (ข้อมูลไม่ต้องผ่าน server กลาง)
✓ Server cost น้อย (เฉพาะ signaling)
✓ Privacy ดีกว่า (ข้อมูลไม่ผ่าน third party)

ข้อจำกัด:
✗ บางครั้งต้องใช้ TURN server (ค่าใช้จ่ายสูง)
✗ Scalability จำกัด (สำหรับ group calls ต้องใช้ SFU/MCU)
✗ NAT traversal ซับซ้อน
✗ เครือข่ายบางประเภทบล็อก WebRTC

WebRTC Components หลัก:
1. MediaStream - จัดการ audio/video streams
2. RTCPeerConnection - จัดการ P2P connections
3. RTCDataChannel - ส่งข้อมูล arbitrary data
```

---

## Step 1652: ICE, STUN, TURN Servers

```
ICE (Interactive Connectivity Establishment)
- Framework สำหรับ NAT traversal
- รวบรวม "candidates" (วิธีที่ peer สามารถเชื่อมต่อได้)
- ลอง candidates ทีละอย่างจนกว่าจะเชื่อมต่อได้

Types of ICE candidates:
1. Host candidate - Local IP address
2. Server reflexive candidate - Public IP จาก STUN server
3. Relay candidate - IP จาก TURN server

STUN (Session Traversal Utilities for NAT)
- บอก client ว่า public IP/port ของตัวเองคืออะไร
- ฟรี, ใช้บาน bandwidth น้อย
- ไม่สามารถใช้กับ symmetric NAT ได้

Public STUN servers:
- stun.l.google.com:19302
- stun1.l.google.com:19302
- stun.cloudflare.com:3478

TURN (Traversal Using Relays around NAT)
- Relay server สำหรับกรณีที่ P2P ไม่ได้
- ต้องจ่ายเงิน (bandwidth cost สูง)
- รองรับ symmetric NAT
- ใช้เป็น fallback

SDP (Session Description Protocol)
- Format สำหรับอธิบาย multimedia session
- บอก codec, resolution, bitrate, etc.
- ใช้ใน offer/answer mechanism

Signaling Server
- Server กลางสำหรับแลกเปลี่ยน SDP และ ICE candidates
- ไม่มี standard protocol (ใช้ WebSocket, HTTP, etc.)
- ไม่ส่ง media data (เฉพาะ metadata สำหรับ setup)
```

---

## Step 1653: getUserMedia() - เข้าถึง Camera และ Microphone

```javascript
// getUserMedia - ขอสิทธิ์เข้าถึง camera/microphone

// Basic usage
async function startCamera() {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({
      video: true,
      audio: true
    });
    
    const video = document.getElementById("localVideo");
    video.srcObject = stream;
    video.play();
    
    return stream;
  } catch (error) {
    if (error.name === "NotAllowedError") {
      console.error("ผู้ใช้ปฏิเสธการเข้าถึง camera/microphone");
    } else if (error.name === "NotFoundError") {
      console.error("ไม่พบ camera/microphone");
    } else {
      console.error("Error:", error.message);
    }
    throw error;
  }
}

// Constraints ที่กำหนดได้
async function startCameraWithConstraints() {
  const constraints = {
    video: {
      width: { ideal: 1280, max: 1920 },
      height: { ideal: 720, max: 1080 },
      frameRate: { ideal: 30, max: 60 },
      facingMode: "user",          // "user" หรือ "environment"
      deviceId: { exact: "xxxxx" } // เลือก specific camera
    },
    audio: {
      echoCancellation: true,
      noiseSuppression: true,
      autoGainControl: true,
      sampleRate: 44100,
      channelCount: 2
    }
  };
  
  return navigator.mediaDevices.getUserMedia(constraints);
}

// ดูรายการ devices
async function listDevices() {
  const devices = await navigator.mediaDevices.enumerateDevices();
  
  const videoDevices = devices.filter(d => d.kind === "videoinput");
  const audioInputDevices = devices.filter(d => d.kind === "audioinput");
  const audioOutputDevices = devices.filter(d => d.kind === "audiooutput");
  
  console.log("Cameras:", videoDevices.map(d => ({ id: d.deviceId, label: d.label })));
  console.log("Microphones:", audioInputDevices.map(d => ({ id: d.deviceId, label: d.label })));
  console.log("Speakers:", audioOutputDevices.map(d => ({ id: d.deviceId, label: d.label })));
  
  return { videoDevices, audioInputDevices, audioOutputDevices };
}

// Switch camera
async function switchCamera(currentStream, newDeviceId) {
  // หยุด tracks เดิม
  currentStream.getVideoTracks().forEach(track => track.stop());
  
  // เริ่ม stream ใหม่ด้วย device อื่น
  const newStream = await navigator.mediaDevices.getUserMedia({
    video: { deviceId: { exact: newDeviceId } },
    audio: false // เก็บ audio track เดิม
  });
  
  return newStream;
}

// MediaTrack controls
function setupTrackControls(stream) {
  const videoTrack = stream.getVideoTracks()[0];
  const audioTrack = stream.getAudioTracks()[0];
  
  // Toggle video
  function toggleVideo() {
    videoTrack.enabled = !videoTrack.enabled;
  }
  
  // Toggle audio (mute/unmute)
  function toggleAudio() {
    audioTrack.enabled = !audioTrack.enabled;
  }
  
  // หยุด stream ทั้งหมด
  function stopStream() {
    stream.getTracks().forEach(track => track.stop());
  }
  
  // ปรับ constraints
  async function setVideoQuality(quality) {
    const constraints = {
      low: { width: 320, height: 240, frameRate: 15 },
      medium: { width: 640, height: 480, frameRate: 30 },
      high: { width: 1280, height: 720, frameRate: 30 }
    };
    
    await videoTrack.applyConstraints(constraints[quality]);
  }
  
  return { toggleVideo, toggleAudio, stopStream, setVideoQuality };
}
```

---

## Step 1654: Screen Sharing with getDisplayMedia()

```javascript
// Screen sharing

async function startScreenShare() {
  try {
    const stream = await navigator.mediaDevices.getDisplayMedia({
      video: {
        displaySurface: "monitor", // "monitor", "window", "browser", "application"
        logicalSurface: true,
        cursor: "always",          // "never", "motion", "always"
        width: { ideal: 1920 },
        height: { ideal: 1080 },
        frameRate: { max: 30 }
      },
      audio: {
        echoCancellation: false,
        noiseSuppression: false,
        autoGainControl: false
      }
    });
    
    // ตรวจสอบว่า user หยุด sharing
    stream.getVideoTracks()[0].addEventListener("ended", () => {
      console.log("ผู้ใช้หยุด screen sharing");
      stopScreenShare(stream);
    });
    
    return stream;
  } catch (error) {
    if (error.name === "NotAllowedError") {
      console.log("ผู้ใช้ยกเลิก screen sharing");
    }
    throw error;
  }
}

// เพิ่ม screen share track เข้าไปใน peer connection
async function addScreenShareToPeer(peerConnection, screenStream) {
  const videoTrack = screenStream.getVideoTracks()[0];
  
  // หา video sender ที่มีอยู่
  const sender = peerConnection.getSenders()
    .find(s => s.track?.kind === "video");
  
  if (sender) {
    // Replace existing video track
    await sender.replaceTrack(videoTrack);
  } else {
    // Add new track
    peerConnection.addTrack(videoTrack, screenStream);
  }
}

// ผสม camera + screen share
async function combineStreams(cameraStream, screenStream) {
  const canvas = document.createElement("canvas");
  const ctx = canvas.getContext("2d");
  canvas.width = 1920;
  canvas.height = 1080;
  
  const cameraVideo = document.createElement("video");
  cameraVideo.srcObject = cameraStream;
  cameraVideo.play();
  
  const screenVideo = document.createElement("video");
  screenVideo.srcObject = screenStream;
  screenVideo.play();
  
  // วาด screen และ camera (picture-in-picture)
  function draw() {
    // วาด screen ใหญ่
    ctx.drawImage(screenVideo, 0, 0, canvas.width, canvas.height);
    
    // วาด camera เล็กที่มุมล่างขวา
    const camWidth = 320;
    const camHeight = 180;
    const camX = canvas.width - camWidth - 20;
    const camY = canvas.height - camHeight - 20;
    
    ctx.save();
    ctx.beginPath();
    ctx.roundRect(camX, camY, camWidth, camHeight, 10);
    ctx.clip();
    ctx.drawImage(cameraVideo, camX, camY, camWidth, camHeight);
    ctx.restore();
    
    requestAnimationFrame(draw);
  }
  draw();
  
  // ส่งคืน stream จาก canvas
  const combinedStream = canvas.captureStream(30);
  
  // เพิ่ม audio tracks
  const audioTracks = cameraStream.getAudioTracks();
  audioTracks.forEach(track => combinedStream.addTrack(track));
  
  return combinedStream;
}
```

---

## Step 1655: Setting up Peer Connection

```javascript
// RTCPeerConnection - หัวใจของ WebRTC

class PeerConnection {
  #pc = null;
  #localStream = null;
  #remoteStream = null;
  
  constructor(config) {
    const defaultConfig = {
      iceServers: [
        { urls: "stun:stun.l.google.com:19302" },
        { urls: "stun:stun1.l.google.com:19302" },
        // TURN server (ถ้ามี)
        {
          urls: "turn:your-turn-server.com:3478",
          username: "user",
          credential: "password"
        }
      ],
      iceCandidatePoolSize: 10,
      bundlePolicy: "max-bundle",
      rtcpMuxPolicy: "require"
    };
    
    this.#pc = new RTCPeerConnection(config || defaultConfig);
    this.#remoteStream = new MediaStream();
    
    this.#setupEventHandlers();
  }
  
  #setupEventHandlers() {
    // เมื่อมี ICE candidate
    this.#pc.onicecandidate = (event) => {
      if (event.candidate) {
        this.onIceCandidate?.(event.candidate);
      }
    };
    
    // เมื่อ ICE connection state เปลี่ยน
    this.#pc.oniceconnectionstatechange = () => {
      console.log("ICE state:", this.#pc.iceConnectionState);
      this.onConnectionStateChange?.(this.#pc.iceConnectionState);
    };
    
    // เมื่อได้รับ remote tracks
    this.#pc.ontrack = (event) => {
      event.streams[0].getTracks().forEach(track => {
        this.#remoteStream.addTrack(track);
      });
      this.onRemoteStream?.(this.#remoteStream);
    };
    
    // Connection state
    this.#pc.onconnectionstatechange = () => {
      console.log("Connection state:", this.#pc.connectionState);
      
      if (this.#pc.connectionState === "failed") {
        this.onError?.(new Error("Connection failed"));
      }
    };
    
    // Negotiation needed (เมื่อ tracks เปลี่ยน)
    this.#pc.onnegotiationneeded = async () => {
      try {
        await this.#createAndSendOffer();
      } catch (error) {
        this.onError?.(error);
      }
    };
  }
  
  async addLocalStream(stream) {
    this.#localStream = stream;
    stream.getTracks().forEach(track => {
      this.#pc.addTrack(track, stream);
    });
  }
  
  async #createAndSendOffer() {
    const offer = await this.#pc.createOffer({
      offerToReceiveAudio: true,
      offerToReceiveVideo: true
    });
    
    await this.#pc.setLocalDescription(offer);
    this.onOffer?.(offer);
  }
  
  async createOffer() {
    await this.#createAndSendOffer();
  }
  
  async handleOffer(offer) {
    await this.#pc.setRemoteDescription(new RTCSessionDescription(offer));
    
    const answer = await this.#pc.createAnswer();
    await this.#pc.setLocalDescription(answer);
    
    this.onAnswer?.(answer);
  }
  
  async handleAnswer(answer) {
    await this.#pc.setRemoteDescription(new RTCSessionDescription(answer));
  }
  
  async addIceCandidate(candidate) {
    await this.#pc.addIceCandidate(new RTCIceCandidate(candidate));
  }
  
  close() {
    this.#pc.close();
    this.#localStream?.getTracks().forEach(track => track.stop());
  }
  
  get stats() {
    return this.#pc.getStats();
  }
}

// Event callbacks (to be set by user)
// peerConn.onIceCandidate = (candidate) => { ... }
// peerConn.onOffer = (offer) => { ... }
// peerConn.onAnswer = (answer) => { ... }
// peerConn.onRemoteStream = (stream) => { ... }
// peerConn.onConnectionStateChange = (state) => { ... }
// peerConn.onError = (error) => { ... }
```

---

## Step 1656: SDP Offer and Answer Exchange

```javascript
// SDP Offer/Answer - การต่อรอง media capabilities

// Signaling via WebSocket
class SignalingClient {
  #ws;
  #roomId;
  #peerId;
  
  constructor(serverUrl) {
    this.#ws = new WebSocket(serverUrl);
    this.#setupHandlers();
  }
  
  #setupHandlers() {
    this.#ws.onmessage = (event) => {
      const message = JSON.parse(event.data);
      
      switch (message.type) {
        case "offer":
          this.onOffer?.(message.offer, message.from);
          break;
        case "answer":
          this.onAnswer?.(message.answer, message.from);
          break;
        case "ice-candidate":
          this.onIceCandidate?.(message.candidate, message.from);
          break;
        case "user-joined":
          this.onUserJoined?.(message.userId);
          break;
        case "user-left":
          this.onUserLeft?.(message.userId);
          break;
      }
    };
    
    this.#ws.onopen = () => {
      console.log("เชื่อมต่อ signaling server แล้ว");
      this.onConnected?.();
    };
    
    this.#ws.onclose = () => {
      console.log("disconnect จาก signaling server");
      this.onDisconnected?.();
    };
  }
  
  joinRoom(roomId, userId) {
    this.#roomId = roomId;
    this.#peerId = userId;
    this.#send({ type: "join", roomId, userId });
  }
  
  sendOffer(offer, to) {
    this.#send({ type: "offer", offer, to, from: this.#peerId, roomId: this.#roomId });
  }
  
  sendAnswer(answer, to) {
    this.#send({ type: "answer", answer, to, from: this.#peerId, roomId: this.#roomId });
  }
  
  sendIceCandidate(candidate, to) {
    this.#send({ type: "ice-candidate", candidate, to, from: this.#peerId, roomId: this.#roomId });
  }
  
  #send(data) {
    if (this.#ws.readyState === WebSocket.OPEN) {
      this.#ws.send(JSON.stringify(data));
    }
  }
  
  disconnect() {
    this.#ws.close();
  }
}

// Signaling server (Node.js + WebSocket)
// ไฟล์: server/signaling.js
const WebSocket = require("ws");
const http = require("http");

const server = http.createServer();
const wss = new WebSocket.Server({ server });

const rooms = new Map(); // roomId -> Map(userId -> ws)

wss.on("connection", (ws) => {
  let currentRoom = null;
  let currentUser = null;
  
  ws.on("message", (data) => {
    const message = JSON.parse(data);
    
    switch (message.type) {
      case "join":
        currentRoom = message.roomId;
        currentUser = message.userId;
        
        if (!rooms.has(currentRoom)) {
          rooms.set(currentRoom, new Map());
        }
        
        const room = rooms.get(currentRoom);
        
        // แจ้ง users ที่อยู่ใน room แล้ว
        room.forEach((peerWs, peerId) => {
          // แจ้ง peer ที่มีอยู่ว่ามี user ใหม่
          peerWs.send(JSON.stringify({
            type: "user-joined",
            userId: currentUser
          }));
          
          // แจ้ง user ใหม่ว่ามี peers อยู่แล้ว
          ws.send(JSON.stringify({
            type: "user-joined",
            userId: peerId
          }));
        });
        
        room.set(currentUser, ws);
        break;
      
      case "offer":
      case "answer":
      case "ice-candidate":
        // Forward message to target peer
        const targetRoom = rooms.get(message.roomId);
        const targetWs = targetRoom?.get(message.to);
        
        if (targetWs && targetWs.readyState === WebSocket.OPEN) {
          targetWs.send(JSON.stringify(message));
        }
        break;
    }
  });
  
  ws.on("close", () => {
    if (currentRoom && currentUser) {
      const room = rooms.get(currentRoom);
      room?.delete(currentUser);
      
      // แจ้ง peers ที่เหลือ
      room?.forEach(peerWs => {
        peerWs.send(JSON.stringify({
          type: "user-left",
          userId: currentUser
        }));
      });
      
      if (room?.size === 0) {
        rooms.delete(currentRoom);
      }
    }
  });
});

server.listen(8080, () => {
  console.log("Signaling server ทำงานที่ port 8080");
});
```

---

## Step 1657: ICE Candidate Exchange

```javascript
// ICE candidate exchange

class WebRTCManager {
  #peerConnections = new Map(); // peerId -> PeerConnection
  #signalingClient;
  #localStream;
  
  constructor(signalingServerUrl) {
    this.#signalingClient = new SignalingClient(signalingServerUrl);
    this.#setupSignalingHandlers();
  }
  
  #setupSignalingHandlers() {
    // เมื่อมี user ใหม่เข้า room
    this.#signalingClient.onUserJoined = async (userId) => {
      console.log(`User ${userId} เข้าร่วม`);
      await this.#initiateConnection(userId);
    };
    
    // เมื่อได้รับ offer
    this.#signalingClient.onOffer = async (offer, from) => {
      const pc = this.#createPeerConnection(from);
      
      await pc.handleOffer(offer);
    };
    
    // เมื่อได้รับ answer
    this.#signalingClient.onAnswer = async (answer, from) => {
      const pc = this.#peerConnections.get(from);
      await pc?.handleAnswer(answer);
    };
    
    // เมื่อได้รับ ICE candidate
    this.#signalingClient.onIceCandidate = async (candidate, from) => {
      const pc = this.#peerConnections.get(from);
      await pc?.addIceCandidate(candidate);
    };
    
    // เมื่อ user ออก
    this.#signalingClient.onUserLeft = (userId) => {
      this.#closePeerConnection(userId);
      this.onUserLeft?.(userId);
    };
  }
  
  #createPeerConnection(peerId) {
    const pc = new PeerConnection();
    
    // ส่ง ICE candidates ผ่าน signaling
    pc.onIceCandidate = (candidate) => {
      this.#signalingClient.sendIceCandidate(candidate, peerId);
    };
    
    pc.onOffer = (offer) => {
      this.#signalingClient.sendOffer(offer, peerId);
    };
    
    pc.onAnswer = (answer) => {
      this.#signalingClient.sendAnswer(answer, peerId);
    };
    
    pc.onRemoteStream = (stream) => {
      this.onRemoteStream?.(peerId, stream);
    };
    
    pc.onConnectionStateChange = (state) => {
      if (state === "connected") {
        console.log(`เชื่อมต่อกับ ${peerId} สำเร็จ`);
        this.onPeerConnected?.(peerId);
      }
    };
    
    if (this.#localStream) {
      pc.addLocalStream(this.#localStream);
    }
    
    this.#peerConnections.set(peerId, pc);
    return pc;
  }
  
  async #initiateConnection(peerId) {
    const pc = this.#createPeerConnection(peerId);
    await pc.createOffer();
  }
  
  #closePeerConnection(peerId) {
    this.#peerConnections.get(peerId)?.close();
    this.#peerConnections.delete(peerId);
  }
  
  async joinRoom(roomId, userId, stream) {
    this.#localStream = stream;
    this.#signalingClient.joinRoom(roomId, userId);
  }
  
  leaveRoom() {
    this.#peerConnections.forEach(pc => pc.close());
    this.#peerConnections.clear();
    this.#signalingClient.disconnect();
  }
}
```

---

## Step 1658: RTCDataChannel - Data Transfer

```javascript
// RTCDataChannel - ส่งข้อมูล arbitrary data แบบ P2P

class DataChannelManager {
  #pc;
  #dataChannels = new Map();
  
  constructor(peerConnection) {
    this.#pc = peerConnection;
    
    // รับ data channels ที่ฝั่งตรงข้ามสร้าง
    peerConnection.ondatachannel = (event) => {
      this.#setupDataChannel(event.channel);
    };
  }
  
  createChannel(name, options = {}) {
    const channel = this.#pc.createDataChannel(name, {
      ordered: true,           // รับประกัน ordering
      maxRetransmits: undefined, // จำนวน retransmits สูงสุด
      protocol: "",
      negotiated: false,
      id: undefined,
      ...options
    });
    
    return this.#setupDataChannel(channel);
  }
  
  #setupDataChannel(channel) {
    channel.onopen = () => {
      console.log(`DataChannel "${channel.label}" เปิดแล้ว`);
      this.onChannelOpen?.(channel.label);
    };
    
    channel.onclose = () => {
      console.log(`DataChannel "${channel.label}" ปิดแล้ว`);
      this.#dataChannels.delete(channel.label);
      this.onChannelClose?.(channel.label);
    };
    
    channel.onmessage = (event) => {
      this.onMessage?.(channel.label, event.data);
    };
    
    channel.onerror = (error) => {
      console.error(`DataChannel error:`, error);
    };
    
    this.#dataChannels.set(channel.label, channel);
    return channel;
  }
  
  send(channelName, data) {
    const channel = this.#dataChannels.get(channelName);
    
    if (!channel || channel.readyState !== "open") {
      throw new Error(`Channel "${channelName}" ไม่พร้อมใช้งาน`);
    }
    
    if (typeof data === "string") {
      channel.send(data);
    } else if (data instanceof ArrayBuffer || ArrayBuffer.isView(data)) {
      channel.send(data);
    } else {
      channel.send(JSON.stringify(data));
    }
  }
  
  sendJSON(channelName, data) {
    this.send(channelName, JSON.stringify(data));
  }
  
  closeChannel(name) {
    this.#dataChannels.get(name)?.close();
  }
}

// ตัวอย่าง: Chat application
class ChatApp {
  #dcManager;
  #messages = [];
  
  constructor(dcManager) {
    this.#dcManager = dcManager;
    
    const chatChannel = dcManager.createChannel("chat", {
      ordered: true // ต้องรับข้อความตามลำดับ
    });
    
    dcManager.onMessage = (channelName, data) => {
      if (channelName === "chat") {
        const message = JSON.parse(data);
        this.#messages.push(message);
        this.onNewMessage?.(message);
      }
    };
  }
  
  sendMessage(text) {
    const message = {
      id: Date.now(),
      text,
      timestamp: new Date().toISOString(),
      sender: "me"
    };
    
    this.#dcManager.sendJSON("chat", message);
    this.#messages.push({ ...message, sender: "me" });
    this.onNewMessage?.(message);
  }
  
  get messages() {
    return [...this.#messages];
  }
}
```

---

## Step 1659: Building a Video Chat App

```javascript
// Video Chat App สมบูรณ์

// HTML:
// <video id="localVideo" autoplay muted playsinline></video>
// <video id="remoteVideo" autoplay playsinline></video>
// <button id="callBtn">โทร</button>
// <button id="hangupBtn" disabled>วางสาย</button>
// <button id="muteBtn">Mute</button>
// <button id="videoBtn">ปิด Video</button>

class VideoChat {
  #localStream = null;
  #peerConnection = null;
  #signalingClient = null;
  #isMuted = false;
  #isVideoOff = false;
  
  constructor() {
    this.#signalingClient = new SignalingClient("wss://signaling.example.com");
    this.#setupUI();
  }
  
  #setupUI() {
    document.getElementById("callBtn").onclick = () => this.call();
    document.getElementById("hangupBtn").onclick = () => this.hangup();
    document.getElementById("muteBtn").onclick = () => this.toggleMute();
    document.getElementById("videoBtn").onclick = () => this.toggleVideo();
    
    // ฟังก์ชั่น UI helper
    this.#signalingClient.onConnected = () => {
      document.getElementById("status").textContent = "เชื่อมต่อแล้ว";
    };
  }
  
  async initialize(roomId) {
    // ขอสิทธิ์ camera/microphone
    this.#localStream = await navigator.mediaDevices.getUserMedia({
      video: { width: 1280, height: 720 },
      audio: { echoCancellation: true, noiseSuppression: true }
    });
    
    document.getElementById("localVideo").srcObject = this.#localStream;
    
    // เข้าร่วม signaling room
    const userId = Math.random().toString(36).slice(2);
    
    this.#signalingClient.onOffer = async (offer, from) => {
      await this.#handleIncomingCall(offer, from);
    };
    
    this.#signalingClient.onAnswer = async (answer) => {
      await this.#peerConnection?.handleAnswer(answer);
    };
    
    this.#signalingClient.onIceCandidate = async (candidate) => {
      await this.#peerConnection?.addIceCandidate(candidate);
    };
    
    this.#signalingClient.joinRoom(roomId, userId);
  }
  
  async call() {
    this.#setupPeerConnection();
    
    // เพิ่ม local tracks
    this.#localStream.getTracks().forEach(track => {
      this.#peerConnection.addTrack(track, this.#localStream);
    });
    
    // สร้าง offer
    const offer = await this.#peerConnection.createOffer();
    await this.#peerConnection.setLocalDescription(offer);
    
    this.#signalingClient.sendOffer(offer, "remote"); // ส่งไปยัง remote peer
    
    document.getElementById("callBtn").disabled = true;
    document.getElementById("hangupBtn").disabled = false;
  }
  
  async #handleIncomingCall(offer, from) {
    this.#setupPeerConnection();
    
    await this.#peerConnection.setRemoteDescription(offer);
    
    // เพิ่ม local tracks
    this.#localStream.getTracks().forEach(track => {
      this.#peerConnection.addTrack(track, this.#localStream);
    });
    
    // สร้าง answer
    const answer = await this.#peerConnection.createAnswer();
    await this.#peerConnection.setLocalDescription(answer);
    
    this.#signalingClient.sendAnswer(answer, from);
    
    document.getElementById("callBtn").disabled = true;
    document.getElementById("hangupBtn").disabled = false;
  }
  
  #setupPeerConnection() {
    this.#peerConnection = new RTCPeerConnection({
      iceServers: [
        { urls: "stun:stun.l.google.com:19302" }
      ]
    });
    
    this.#peerConnection.onicecandidate = (event) => {
      if (event.candidate) {
        this.#signalingClient.sendIceCandidate(event.candidate, "remote");
      }
    };
    
    this.#peerConnection.ontrack = (event) => {
      document.getElementById("remoteVideo").srcObject = event.streams[0];
    };
    
    this.#peerConnection.onconnectionstatechange = () => {
      const state = this.#peerConnection.connectionState;
      document.getElementById("status").textContent = 
        state === "connected" ? "กำลังโทรอยู่" : state;
    };
  }
  
  hangup() {
    this.#peerConnection?.close();
    this.#peerConnection = null;
    
    document.getElementById("remoteVideo").srcObject = null;
    document.getElementById("callBtn").disabled = false;
    document.getElementById("hangupBtn").disabled = true;
    document.getElementById("status").textContent = "สาย วาง";
  }
  
  toggleMute() {
    this.#isMuted = !this.#isMuted;
    this.#localStream.getAudioTracks().forEach(track => {
      track.enabled = !this.#isMuted;
    });
    document.getElementById("muteBtn").textContent = 
      this.#isMuted ? "Unmute" : "Mute";
  }
  
  toggleVideo() {
    this.#isVideoOff = !this.#isVideoOff;
    this.#localStream.getVideoTracks().forEach(track => {
      track.enabled = !this.#isVideoOff;
    });
    document.getElementById("videoBtn").textContent = 
      this.#isVideoOff ? "เปิด Video" : "ปิด Video";
  }
  
  async shareScreen() {
    const screenStream = await navigator.mediaDevices.getDisplayMedia({
      video: true
    });
    
    const screenTrack = screenStream.getVideoTracks()[0];
    
    // Replace video track ใน peer connection
    const sender = this.#peerConnection.getSenders()
      .find(s => s.track?.kind === "video");
    
    await sender?.replaceTrack(screenTrack);
    
    // เมื่อหยุด screen share ให้ใช้ camera อีกครั้ง
    screenTrack.onended = async () => {
      const cameraTrack = this.#localStream.getVideoTracks()[0];
      await sender?.replaceTrack(cameraTrack);
    };
  }
}

// การใช้งาน
const chat = new VideoChat();
await chat.initialize("room-123");
```

---

## Step 1660: Building a File Sharing App

```javascript
// File sharing ผ่าน WebRTC DataChannel

class FileShare {
  #pc;
  #signalingClient;
  #sendChannel;
  #receiveChannel;
  #receivedChunks = [];
  #receivedSize = 0;
  #fileMetadata = null;
  
  static CHUNK_SIZE = 16384; // 16KB chunks
  
  constructor() {
    this.#setupConnection();
  }
  
  #setupConnection() {
    this.#pc = new RTCPeerConnection({
      iceServers: [{ urls: "stun:stun.l.google.com:19302" }]
    });
    
    // สร้าง data channel สำหรับส่งไฟล์
    this.#sendChannel = this.#pc.createDataChannel("fileTransfer", {
      ordered: true,
      maxRetransmits: 10
    });
    
    this.#sendChannel.binaryType = "arraybuffer";
    
    // รับ data channel จาก peer
    this.#pc.ondatachannel = (event) => {
      this.#receiveChannel = event.channel;
      this.#receiveChannel.binaryType = "arraybuffer";
      this.#setupReceiver();
    };
    
    this.#pc.onicecandidate = (event) => {
      if (event.candidate) {
        this.onIceCandidate?.(event.candidate);
      }
    };
  }
  
  async sendFile(file) {
    // ส่ง metadata ก่อน
    const metadata = {
      type: "metadata",
      name: file.name,
      size: file.size,
      fileType: file.type,
      chunks: Math.ceil(file.size / FileShare.CHUNK_SIZE)
    };
    
    this.#sendChannel.send(JSON.stringify(metadata));
    
    // ส่งไฟล์เป็น chunks
    const arrayBuffer = await file.arrayBuffer();
    let offset = 0;
    let chunkIndex = 0;
    
    const sendNextChunk = () => {
      if (offset >= arrayBuffer.byteLength) {
        // ส่ง end signal
        this.#sendChannel.send(JSON.stringify({ type: "end" }));
        this.onSendComplete?.();
        return;
      }
      
      // ตรวจสอบ buffer ก่อนส่ง
      if (this.#sendChannel.bufferedAmount > FileShare.CHUNK_SIZE * 8) {
        setTimeout(sendNextChunk, 50);
        return;
      }
      
      const chunk = arrayBuffer.slice(offset, offset + FileShare.CHUNK_SIZE);
      this.#sendChannel.send(chunk);
      
      offset += FileShare.CHUNK_SIZE;
      chunkIndex++;
      
      // รายงาน progress
      const progress = Math.min(100, Math.floor((offset / arrayBuffer.byteLength) * 100));
      this.onSendProgress?.(progress, chunkIndex, metadata.chunks);
      
      // ส่ง chunk ถัดไป
      setTimeout(sendNextChunk, 0);
    };
    
    sendNextChunk();
  }
  
  #setupReceiver() {
    this.#receiveChannel.onmessage = (event) => {
      if (typeof event.data === "string") {
        const message = JSON.parse(event.data);
        
        if (message.type === "metadata") {
          this.#fileMetadata = message;
          this.#receivedChunks = [];
          this.#receivedSize = 0;
          this.onReceiveStart?.(message);
        } else if (message.type === "end") {
          this.#assembleFile();
        }
      } else {
        // Binary chunk
        this.#receivedChunks.push(event.data);
        this.#receivedSize += event.data.byteLength;
        
        const progress = Math.floor((this.#receivedSize / this.#fileMetadata.size) * 100);
        this.onReceiveProgress?.(progress);
      }
    };
  }
  
  #assembleFile() {
    const blob = new Blob(this.#receivedChunks, {
      type: this.#fileMetadata.fileType
    });
    
    // Download file
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = this.#fileMetadata.name;
    a.click();
    URL.revokeObjectURL(url);
    
    this.onReceiveComplete?.(blob, this.#fileMetadata);
    
    // Reset
    this.#receivedChunks = [];
    this.#receivedSize = 0;
    this.#fileMetadata = null;
  }
}

// UI สำหรับ File Sharing
async function setupFileShareUI() {
  const fileShare = new FileShare();
  
  // Setup signaling และ connection
  // ... (ละเอียดเหมือน video chat)
  
  // Progress UI
  fileShare.onSendProgress = (percent, chunk, total) => {
    document.getElementById("sendProgress").value = percent;
    document.getElementById("sendStatus").textContent = 
      `ส่งแล้ว ${chunk}/${total} chunks (${percent}%)`;
  };
  
  fileShare.onReceiveProgress = (percent) => {
    document.getElementById("receiveProgress").value = percent;
    document.getElementById("receiveStatus").textContent = 
      `รับแล้ว ${percent}%`;
  };
  
  fileShare.onReceiveComplete = (blob, metadata) => {
    console.log(`รับไฟล์ ${metadata.name} สำเร็จ (${metadata.size} bytes)`);
  };
  
  // Drop zone
  const dropZone = document.getElementById("dropZone");
  
  dropZone.addEventListener("dragover", (e) => {
    e.preventDefault();
    dropZone.classList.add("dragover");
  });
  
  dropZone.addEventListener("drop", (e) => {
    e.preventDefault();
    dropZone.classList.remove("dragover");
    
    const file = e.dataTransfer.files[0];
    if (file) fileShare.sendFile(file);
  });
}
```

---

## Step 1661: WebRTC Stats and Monitoring

```javascript
// WebRTC Statistics API

class WebRTCMonitor {
  #pc;
  #statsInterval = null;
  #previousStats = {};
  
  constructor(peerConnection) {
    this.#pc = peerConnection;
  }
  
  startMonitoring(intervalMs = 1000) {
    this.#statsInterval = setInterval(async () => {
      await this.#collectStats();
    }, intervalMs);
  }
  
  stopMonitoring() {
    clearInterval(this.#statsInterval);
    this.#statsInterval = null;
  }
  
  async #collectStats() {
    const stats = await this.#pc.getStats();
    const metrics = {};
    
    stats.forEach(report => {
      switch (report.type) {
        case "inbound-rtp":
          if (report.kind === "video") {
            metrics.video = {
              packetsReceived: report.packetsReceived,
              bytesReceived: report.bytesReceived,
              packetsLost: report.packetsLost,
              jitter: report.jitter,
              framesDecoded: report.framesDecoded,
              framesPerSecond: report.framesPerSecond,
              frameWidth: report.frameWidth,
              frameHeight: report.frameHeight
            };
          } else if (report.kind === "audio") {
            metrics.audio = {
              packetsReceived: report.packetsReceived,
              bytesReceived: report.bytesReceived,
              packetsLost: report.packetsLost,
              jitter: report.jitter,
              audioLevel: report.audioLevel
            };
          }
          break;
        
        case "outbound-rtp":
          if (report.kind === "video") {
            metrics.videoSent = {
              packetsSent: report.packetsSent,
              bytesSent: report.bytesSent,
              framesSent: report.framesSent,
              framesPerSecond: report.framesPerSecond,
              qualityLimitationReason: report.qualityLimitationReason
            };
          }
          break;
        
        case "candidate-pair":
          if (report.state === "succeeded") {
            metrics.connection = {
              roundTripTime: report.currentRoundTripTime * 1000, // ms
              availableBitrate: report.availableOutgoingBitrate,
              bytesSent: report.bytesSent,
              bytesReceived: report.bytesReceived
            };
          }
          break;
      }
    });
    
    // คำนวณ bandwidth
    if (this.#previousStats.video && metrics.video) {
      const timeDiff = 1; // 1 second
      const bytesDiff = metrics.video.bytesReceived - this.#previousStats.video.bytesReceived;
      metrics.videoBandwidth = (bytesDiff * 8) / timeDiff; // bits per second
    }
    
    this.#previousStats = metrics;
    this.onStats?.(metrics);
    
    // แจ้งเตือนถ้า quality ไม่ดี
    if (metrics.connection?.roundTripTime > 300) {
      this.onHighLatency?.(metrics.connection.roundTripTime);
    }
    
    if (metrics.video?.packetsLost > 100) {
      this.onPacketLoss?.(metrics.video.packetsLost);
    }
  }
  
  async getConnectionInfo() {
    const stats = await this.#pc.getStats();
    
    for (const [, report] of stats) {
      if (report.type === "local-candidate" && report.candidateType) {
        return {
          localCandidateType: report.candidateType,
          protocol: report.protocol,
          address: report.address,
          port: report.port
        };
      }
    }
  }
}

// Dashboard สำหรับแสดง stats
class StatsDashboard {
  #monitor;
  #chartData = { labels: [], rtt: [], bandwidth: [], fps: [] };
  
  constructor(monitor) {
    this.#monitor = monitor;
    
    monitor.onStats = (stats) => this.#updateDashboard(stats);
    monitor.onHighLatency = (rtt) => {
      this.#showWarning(`⚠️ Latency สูง: ${rtt.toFixed(0)}ms`);
    };
  }
  
  #updateDashboard(stats) {
    const now = new Date().toLocaleTimeString("th-TH");
    this.#chartData.labels.push(now);
    this.#chartData.rtt.push(stats.connection?.roundTripTime || 0);
    this.#chartData.bandwidth.push(stats.videoBandwidth || 0);
    this.#chartData.fps.push(stats.video?.framesPerSecond || 0);
    
    // เก็บแค่ 60 วินาที
    if (this.#chartData.labels.length > 60) {
      Object.values(this.#chartData).forEach(arr => arr.shift());
    }
    
    // Update UI
    document.getElementById("rtt").textContent = 
      `${stats.connection?.roundTripTime?.toFixed(0) || 0}ms`;
    document.getElementById("fps").textContent = 
      `${stats.video?.framesPerSecond?.toFixed(0) || 0} FPS`;
    document.getElementById("resolution").textContent = 
      `${stats.video?.frameWidth || 0}x${stats.video?.frameHeight || 0}`;
    document.getElementById("bandwidth").textContent = 
      `${((stats.videoBandwidth || 0) / 1000000).toFixed(2)} Mbps`;
  }
  
  #showWarning(message) {
    const warning = document.createElement("div");
    warning.className = "warning";
    warning.textContent = message;
    document.body.appendChild(warning);
    setTimeout(() => warning.remove(), 5000);
  }
}
```

---

## Step 1662: WebRTC in Production

```javascript
// Best practices สำหรับ WebRTC ใน production

// 1. Adaptive Bitrate
class AdaptiveBitrateController {
  #pc;
  
  constructor(peerConnection) {
    this.#pc = peerConnection;
  }
  
  async setVideoBitrate(bitsPerSecond) {
    const sender = this.#pc.getSenders()
      .find(s => s.track?.kind === "video");
    
    if (!sender) return;
    
    const params = sender.getParameters();
    
    if (!params.encodings) {
      params.encodings = [{}];
    }
    
    params.encodings[0].maxBitrate = bitsPerSecond;
    
    await sender.setParameters(params);
  }
  
  async setSimulcast(layers) {
    const sender = this.#pc.getSenders()
      .find(s => s.track?.kind === "video");
    
    if (!sender) return;
    
    const params = sender.getParameters();
    params.encodings = layers.map((layer, i) => ({
      rid: String(i),
      active: true,
      maxBitrate: layer.maxBitrate,
      scaleResolutionDownBy: layer.scaleDown
    }));
    
    await sender.setParameters(params);
  }
}

// ตัวอย่างการตั้ง simulcast
const controller = new AdaptiveBitrateController(peerConnection);
await controller.setSimulcast([
  { maxBitrate: 800000, scaleDown: 1 },   // High quality (1x)
  { maxBitrate: 400000, scaleDown: 2 },   // Medium quality (1/2x)
  { maxBitrate: 100000, scaleDown: 4 }    // Low quality (1/4x)
]);

// 2. TURN server configuration
const turnConfig = {
  iceServers: [
    {
      urls: [
        "stun:stun.example.com:3478",
        "stun:stun1.example.com:3478"
      ]
    },
    {
      urls: [
        "turn:turn.example.com:3478?transport=udp",
        "turn:turn.example.com:3478?transport=tcp",
        "turns:turn.example.com:443?transport=tcp" // TLS สำหรับ firewall bypass
      ],
      username: "dynamic_username",    // สร้างแบบ dynamic
      credential: "dynamic_password"  // มีอายุจำกัด
    }
  ],
  iceTransportPolicy: "all" // หรือ "relay" เพื่อบังคับใช้ TURN
};

// 3. Reconnection logic
class ResilientPeerConnection {
  #pc = null;
  #reconnectAttempts = 0;
  #maxReconnectAttempts = 5;
  
  async connect() {
    this.#pc = new RTCPeerConnection(turnConfig);
    this.#setupMonitoring();
  }
  
  #setupMonitoring() {
    this.#pc.onconnectionstatechange = async () => {
      const state = this.#pc.connectionState;
      
      if (state === "failed" || state === "disconnected") {
        if (this.#reconnectAttempts < this.#maxReconnectAttempts) {
          console.log(`Reconnecting... (attempt ${++this.#reconnectAttempts})`);
          await this.#reconnect();
        } else {
          this.onConnectionFailed?.();
        }
      } else if (state === "connected") {
        this.#reconnectAttempts = 0;
      }
    };
  }
  
  async #reconnect() {
    // ICE restart
    const offer = await this.#pc.createOffer({ iceRestart: true });
    await this.#pc.setLocalDescription(offer);
    this.onNeedReconnect?.(offer);
    
    // Wait with exponential backoff
    const delay = Math.min(1000 * Math.pow(2, this.#reconnectAttempts), 30000);
    await new Promise(resolve => setTimeout(resolve, delay));
  }
}

// 4. Network quality indicator
async function getNetworkQuality(pc) {
  const stats = await pc.getStats();
  let rtt = 0;
  let packetsLostRatio = 0;
  
  stats.forEach(report => {
    if (report.type === "candidate-pair" && report.state === "succeeded") {
      rtt = report.currentRoundTripTime * 1000;
    }
    if (report.type === "inbound-rtp" && report.kind === "video") {
      const total = report.packetsReceived + report.packetsLost;
      packetsLostRatio = total > 0 ? report.packetsLost / total : 0;
    }
  });
  
  if (rtt < 100 && packetsLostRatio < 0.01) return "excellent";
  if (rtt < 200 && packetsLostRatio < 0.03) return "good";
  if (rtt < 400 && packetsLostRatio < 0.08) return "fair";
  return "poor";
}
```

---

## Step 1663-1670: Group Video Call (SFU Concept)

```javascript
// Group video call ต้องใช้ SFU (Selective Forwarding Unit)
// เพราะ P2P N-to-N ใช้ bandwidth มากเกินไป

// ตัวอย่างการใช้งาน Mediasoup หรือ LiveKit

// Client-side SFU integration (ตัวอย่าง abstracted)
class SFURoom {
  #ws;
  #device;
  #transport = null;
  #producers = new Map();
  #consumers = new Map();
  
  constructor(serverUrl) {
    this.#ws = new WebSocket(serverUrl);
  }
  
  async join(roomId, userId) {
    const { routerRtpCapabilities } = await this.#request("getRouterRtpCapabilities");
    
    // Load device capabilities
    this.#device = new Device(); // mediasoup-client
    await this.#device.load({ routerRtpCapabilities });
    
    // สร้าง transports
    await this.#createSendTransport();
    await this.#createRecvTransport();
    
    // เข้าร่วม room
    const { peers } = await this.#request("join", { roomId, userId });
    
    // Subscribe to existing peers
    for (const peer of peers) {
      await this.#subscribeToProducers(peer);
    }
    
    // Listen for new peers
    this.#ws.onmessage = async (event) => {
      const message = JSON.parse(event.data);
      
      if (message.type === "newProducer") {
        await this.#consume(message.producerId, message.userId);
      }
    };
  }
  
  async publishStream(stream) {
    for (const track of stream.getTracks()) {
      const producer = await this.#transport.produce({
        track,
        encodings: track.kind === "video" ? [
          { maxBitrate: 100000, scaleResolutionDownBy: 4 },
          { maxBitrate: 300000, scaleResolutionDownBy: 2 },
          { maxBitrate: 900000 }
        ] : undefined,
        codecOptions: track.kind === "video" ? {
          videoGoogleStartBitrate: 1000
        } : undefined
      });
      
      this.#producers.set(track.kind, producer);
    }
  }
  
  async #createSendTransport() {
    const transportInfo = await this.#request("createWebRtcTransport", {
      producing: true,
      consuming: false
    });
    
    this.#transport = this.#device.createSendTransport(transportInfo);
    
    this.#transport.on("connect", async ({ dtlsParameters }, callback, errback) => {
      await this.#request("connectTransport", { dtlsParameters });
      callback();
    });
    
    this.#transport.on("produce", async ({ kind, rtpParameters }, callback, errback) => {
      const { producerId } = await this.#request("produce", {
        kind,
        rtpParameters
      });
      callback({ id: producerId });
    });
  }
  
  async #consume(producerId, peerId) {
    const { consumerParameters } = await this.#request("consume", {
      producerId,
      rtpCapabilities: this.#device.rtpCapabilities
    });
    
    const consumer = await this.#recvTransport.consume(consumerParameters);
    this.#consumers.set(peerId, consumer);
    
    // สร้าง MediaStream สำหรับ peer นี้
    const stream = new MediaStream([consumer.track]);
    this.onNewPeer?.(peerId, stream);
  }
  
  async #request(method, data = {}) {
    return new Promise((resolve, reject) => {
      const id = Math.random().toString(36).slice(2);
      
      this.#ws.send(JSON.stringify({ id, method, data }));
      
      const handler = (event) => {
        const message = JSON.parse(event.data);
        if (message.id === id) {
          this.#ws.removeEventListener("message", handler);
          if (message.error) reject(new Error(message.error));
          else resolve(message.result);
        }
      };
      
      this.#ws.addEventListener("message", handler);
    });
  }
}

// Simple P2P multi-party (สำหรับ N <= 4)
class MeshRoom {
  #peers = new Map(); // peerId -> { pc, stream }
  #localStream;
  
  constructor(signalingClient) {
    this.#signalingClient = signalingClient;
    this.#setupSignaling();
  }
  
  #setupSignaling() {
    this.#signalingClient.onUserJoined = (userId) => {
      this.#createPeerForNewUser(userId);
    };
    
    this.#signalingClient.onOffer = async (offer, from) => {
      if (!this.#peers.has(from)) {
        this.#createPeerForExistingUser(from, offer);
      }
    };
    
    this.#signalingClient.onAnswer = async (answer, from) => {
      await this.#peers.get(from)?.pc.setRemoteDescription(answer);
    };
    
    this.#signalingClient.onIceCandidate = async (candidate, from) => {
      await this.#peers.get(from)?.pc.addIceCandidate(candidate);
    };
    
    this.#signalingClient.onUserLeft = (userId) => {
      this.#removePeer(userId);
    };
  }
  
  async joinRoom(roomId, userId, localStream) {
    this.#localStream = localStream;
    this.#signalingClient.joinRoom(roomId, userId);
  }
  
  #removePeer(peerId) {
    const peer = this.#peers.get(peerId);
    if (peer) {
      peer.pc.close();
      this.#peers.delete(peerId);
      this.onPeerLeft?.(peerId);
    }
  }
  
  setLocalStream(stream) {
    this.#localStream = stream;
    this.#peers.forEach(({ pc }) => {
      stream.getTracks().forEach(track => pc.addTrack(track, stream));
    });
  }
  
  toggleAudio(enabled) {
    this.#localStream?.getAudioTracks()
      .forEach(t => (t.enabled = enabled));
  }
  
  toggleVideo(enabled) {
    this.#localStream?.getVideoTracks()
      .forEach(t => (t.enabled = enabled));
  }
}
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: Video Chat Application
สร้าง video chat แบบ 1-on-1 ที่สมบูรณ์:
- Camera/Mic permission
- Mute/Unmute buttons
- Video on/off
- Screen sharing
- Chat sidebar

### แบบฝึกหัดที่ 2: File Transfer
สร้าง P2P file sharing:
- Drag and drop files
- Progress bar
- Multiple files
- Resume interrupted transfers

### แบบฝึกหัดที่ 3: Collaborative Whiteboard
สร้าง real-time whiteboard:
- วาด shapes ผ่าน DataChannel
- Sync cursor positions
- Undo/Redo

### แบบฝึกหัดที่ 4: Network Quality Indicator
สร้าง stats dashboard:
- RTT graph
- Bandwidth graph
- Packet loss indicator
- Connection quality score

### แบบฝึกหัดที่ 5: Reconnection System
สร้าง robust reconnection:
- Detect disconnection
- ICE restart
- Exponential backoff
- State recovery

---

## สรุป (Summary)

ใน Part 84 เราได้เรียนรู้ WebRTC:

1. **WebRTC คืออะไร** - P2P communication, use cases
2. **ICE, STUN, TURN** - NAT traversal
3. **getUserMedia** - Camera/Mic access
4. **getDisplayMedia** - Screen sharing
5. **RTCPeerConnection** - P2P connection setup
6. **SDP Offer/Answer** - Media negotiation
7. **ICE Candidates** - Connection path discovery
8. **RTCDataChannel** - Binary data transfer
9. **Video Chat App** - Complete implementation
10. **File Sharing** - Chunked P2P file transfer
11. **Stats API** - Connection monitoring
12. **Production Best Practices** - Adaptive bitrate, TURN, reconnection
13. **Group Calls** - SFU concept, mesh topology

WebRTC เป็น API ที่ทรงพลังแต่ซับซ้อน
สำหรับ production ควรพิจารณาใช้ libraries เช่น PeerJS, LiveKit, Daily.co
