# Part 88: AI/ML ใน JavaScript
## ขั้นตอนที่ 1731-1750: ปัญญาประดิษฐ์และ Machine Learning

AI/ML ไม่ได้จำกัดอยู่แค่ Python อีกต่อไป JavaScript มี ecosystem ที่แข็งแกร่งสำหรับ AI/ML ทั้งใน browser และ Node.js

---

## ขั้นตอนที่ 1731: AI/ML ใน Browser

### ทำไมต้องทำ ML ใน Browser?

```
ข้อดี:
✅ Privacy: ข้อมูลไม่ออกจากเครื่องผู้ใช้
✅ Offline capability: ทำงานได้ไม่มีอินเทอร์เน็ต
✅ Low latency: ไม่ต้องส่งข้อมูลไป server
✅ ลด server cost: ไม่ต้องจ่าย GPU

ข้อเสีย:
❌ Hardware limitation: CPU/GPU ของ client อาจอ่อน
❌ Model size: ต้องดาวน์โหลด model ขนาดใหญ่
❌ Battery drain: การ inference ใช้พลังงานมาก
❌ Browser compatibility: WebGL/WebGPU ไม่รองรับทุก browser
```

### Technologies

```javascript
// 1. TensorFlow.js - Google's ML library
// 2. ONNX Runtime Web - Microsoft's ML runtime
// 3. Transformers.js - HuggingFace models
// 4. WebLLM - LLMs in browser
// 5. Brain.js - Simple neural networks
// 6. ml5.js - Friendly ML library (ใช้ TF.js)
```

---

## ขั้นตอนที่ 1732: TensorFlow.js - Setup

```html
<!-- ใช้ใน browser -->
<!-- CDN - ใช้ WebGL backend -->
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs/dist/tf.min.js"></script>

<!-- หรือ WebGPU backend (ถ้า browser รองรับ) -->
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs-backend-webgpu"></script>
```

```bash
# Node.js
npm install @tensorflow/tfjs-node

# Node.js กับ GPU
npm install @tensorflow/tfjs-node-gpu
```

```javascript
// Setup
import * as tf from '@tensorflow/tfjs';

// เลือก backend
await tf.setBackend('webgl');   // GPU acceleration ใน browser
// await tf.setBackend('cpu');  // CPU เท่านั้น
// await tf.setBackend('webgpu'); // WebGPU (เร็วกว่า WebGL)

console.log('Backend:', tf.getBackend());
console.log('TF.js version:', tf.version.tfjs);
```

---

## ขั้นตอนที่ 1733: Tensors และ Operations

```javascript
// Tensor คือ multi-dimensional array
import * as tf from '@tensorflow/tfjs';

// สร้าง Tensor
const scalar = tf.scalar(5);               // 0D tensor
const vector = tf.tensor1d([1, 2, 3, 4]); // 1D tensor
const matrix = tf.tensor2d([[1, 2], [3, 4]]); // 2D tensor
const tensor3d = tf.tensor3d([[[1, 2], [3, 4]], [[5, 6], [7, 8]]]); // 3D

console.log(matrix.shape);  // [2, 2]
console.log(matrix.dtype);  // float32

// แสดงค่า
matrix.print();
// Tensor
//     [[1, 2],
//      [3, 4]]

// Tensor operations
const a = tf.tensor2d([[1, 2], [3, 4]]);
const b = tf.tensor2d([[5, 6], [7, 8]]);

// การคำนวณพื้นฐาน
const sum = a.add(b);         // [[6, 8], [10, 12]]
const diff = a.sub(b);        // [[-4, -4], [-4, -4]]
const product = a.mul(b);     // [[5, 12], [21, 32]] element-wise
const divided = a.div(b);     // [[0.2, 0.33], [0.43, 0.5]]

// Matrix multiplication (matmul)
const matmul = tf.matMul(a, b);
// [[19, 22], [43, 50]]

// Transpose
const transposed = a.transpose();
// [[1, 3], [2, 4]]

// Statistical operations
const data = tf.tensor1d([1, 2, 3, 4, 5]);
const mean = data.mean();        // 3
const std = data.std();          // 1.41...
const min = data.min();          // 1
const max = data.max();          // 5
const sum2 = data.sum();         // 15

// Reshape
const flat = tf.tensor1d([1, 2, 3, 4, 5, 6]);
const reshaped = flat.reshape([2, 3]);
// [[1, 2, 3], [4, 5, 6]]
```

```javascript
// Memory Management - สำคัญมาก!
// TensorFlow.js ใช้ GPU memory, ต้องจัดการอย่างระวัง

// ❌ Memory leak
function badExample() {
  for (let i = 0; i < 1000; i++) {
    const t = tf.tensor2d([[1, 2], [3, 4]]); // สร้าง tensor ใหม่ทุก iteration
    // t ไม่ถูก dispose!
  }
}

// ✅ ถูกต้อง - ใช้ tf.tidy()
function goodExample() {
  tf.tidy(() => {
    // Tensors ที่สร้างใน tidy จะถูก dispose อัตโนมัติ
    const t1 = tf.tensor2d([[1, 2], [3, 4]]);
    const t2 = tf.tensor2d([[5, 6], [7, 8]]);
    const result = t1.matMul(t2);
    // t1 และ t2 จะถูก dispose เมื่อ tidy จบ
    return result; // result จะถูก keep ไว้
  });
}

// ✅ หรือ dispose manually
const t = tf.tensor2d([[1, 2], [3, 4]]);
// ใช้งาน t...
t.dispose(); // คืน memory

// ดูจำนวน tensors ใน memory
console.log(tf.memory().numTensors);
```

---

## ขั้นตอนที่ 1734: Loading Pre-trained Models

```javascript
// โหลด model จาก TensorFlow Hub
import * as tf from '@tensorflow/tfjs';
import * as mobilenet from '@tensorflow-models/mobilenet';

async function loadAndPredict() {
  console.log('กำลังโหลด model...');
  const model = await mobilenet.load();
  console.log('โหลด model สำเร็จ!');
  
  // โหลดรูปภาพ
  const imgElement = document.getElementById('myImage');
  
  // ทำนาย
  const predictions = await model.classify(imgElement);
  
  console.log('ผลการจำแนก:');
  predictions.forEach(pred => {
    console.log(`${pred.className}: ${(pred.probability * 100).toFixed(1)}%`);
  });
  
  return predictions;
}

// ตัวอย่างผลลัพธ์:
// golden retriever: 85.3%
// Labrador retriever: 7.2%
// kuvasz: 2.1%
```

```javascript
// โหลด saved model
const model = await tf.loadLayersModel('https://example.com/model/model.json');
// หรือจาก file (Node.js)
const model2 = await tf.loadLayersModel('file://./my-model/model.json');

// Inspect model
model.summary();
// Layer (type)                 Output shape          Param #
// =================================================================
// dense_Dense1 (Dense)         [null,128]            100480
// dense_Dense2 (Dense)         [null,64]             8256
// ...

// ทำ prediction
const input = tf.tensor2d([[1.5, 2.3, 0.8]]);
const prediction = model.predict(input);
prediction.print();
```

---

## ขั้นตอนที่ 1735: Making Predictions

```javascript
// Image Classification ด้วย TensorFlow.js

import * as tf from '@tensorflow/tfjs';
import * as cocoSsd from '@tensorflow-models/coco-ssd';

class ObjectDetector {
  constructor() {
    this.model = null;
  }
  
  async load() {
    console.log('กำลังโหลด Object Detection model...');
    this.model = await cocoSsd.load();
    console.log('พร้อมใช้งาน!');
  }
  
  async detect(imageElement) {
    if (!this.model) await this.load();
    
    const predictions = await this.model.detect(imageElement);
    
    return predictions.map(pred => ({
      class: pred.class,
      score: (pred.score * 100).toFixed(1) + '%',
      bbox: {
        x: Math.round(pred.bbox[0]),
        y: Math.round(pred.bbox[1]),
        width: Math.round(pred.bbox[2]),
        height: Math.round(pred.bbox[3]),
      },
    }));
  }
  
  drawPredictions(canvas, predictions) {
    const ctx = canvas.getContext('2d');
    
    predictions.forEach(pred => {
      const { x, y, width, height } = pred.bbox;
      
      // วาด bounding box
      ctx.strokeStyle = '#00ff00';
      ctx.lineWidth = 2;
      ctx.strokeRect(x, y, width, height);
      
      // วาด label
      ctx.fillStyle = '#00ff00';
      ctx.font = '16px Arial';
      ctx.fillText(`${pred.class} (${pred.score})`, x, y - 5);
    });
  }
}

// ใช้งาน
const detector = new ObjectDetector();
await detector.load();

const videoElement = document.getElementById('video');
const canvas = document.getElementById('canvas');

// Real-time detection จาก webcam
async function detectFromWebcam() {
  const predictions = await detector.detect(videoElement);
  
  canvas.getContext('2d').drawImage(videoElement, 0, 0);
  detector.drawPredictions(canvas, predictions);
  
  requestAnimationFrame(detectFromWebcam);
}

// เริ่ม webcam
const stream = await navigator.mediaDevices.getUserMedia({ video: true });
videoElement.srcObject = stream;
videoElement.addEventListener('loadedmetadata', detectFromWebcam);
```

---

## ขั้นตอนที่ 1736: Training ใน Browser

```javascript
// สร้างและ train neural network ใน browser
import * as tf from '@tensorflow/tfjs';

// สร้าง dataset (XOR problem)
const xs = tf.tensor2d([
  [0, 0],
  [0, 1],
  [1, 0],
  [1, 1],
]);

const ys = tf.tensor2d([
  [0],
  [1],
  [1],
  [0],
]);

// สร้าง model
function createModel() {
  const model = tf.sequential();
  
  model.add(tf.layers.dense({
    inputShape: [2],
    units: 4,
    activation: 'relu',
    kernelInitializer: 'glorotUniform',
  }));
  
  model.add(tf.layers.dense({
    units: 1,
    activation: 'sigmoid',
  }));
  
  return model;
}

async function trainModel() {
  const model = createModel();
  
  model.compile({
    optimizer: tf.train.adam(0.01),
    loss: 'binaryCrossentropy',
    metrics: ['accuracy'],
  });
  
  // Train
  const history = await model.fit(xs, ys, {
    epochs: 100,
    validationSplit: 0.2,
    callbacks: {
      onEpochEnd: (epoch, logs) => {
        if (epoch % 10 === 0) {
          console.log(`Epoch ${epoch}: loss=${logs.loss.toFixed(4)}, accuracy=${logs.acc.toFixed(4)}`);
        }
      },
    },
  });
  
  console.log('Training เสร็จสิ้น!');
  
  // ทดสอบ
  const testInputs = tf.tensor2d([[0, 0], [0, 1], [1, 0], [1, 1]]);
  const predictions = model.predict(testInputs);
  predictions.print();
  
  return model;
}

const trainedModel = await trainModel();

// บันทึก model
await trainedModel.save('localstorage://xor-model');
// หรือ
await trainedModel.save('downloads://xor-model'); // ดาวน์โหลด
```

---

## ขั้นตอนที่ 1737: Transfer Learning

```javascript
// Transfer Learning กับ MobileNet
import * as tf from '@tensorflow/tfjs';
import * as mobilenet from '@tensorflow-models/mobilenet';

class ImageClassifierTransfer {
  constructor() {
    this.baseModel = null;
    this.classifier = null;
    this.trainingData = { features: [], labels: [] };
    this.numClasses = 0;
    this.classNames = [];
  }
  
  async loadBaseModel() {
    this.baseModel = await mobilenet.load();
    console.log('โหลด MobileNet สำเร็จ');
  }
  
  // เพิ่ม training example
  async addExample(imageElement, classIndex) {
    const features = await this.extractFeatures(imageElement);
    
    this.trainingData.features.push(features);
    this.trainingData.labels.push(classIndex);
    
    if (classIndex >= this.numClasses) {
      this.numClasses = classIndex + 1;
    }
  }
  
  async extractFeatures(imageElement) {
    // ใช้ MobileNet เป็น feature extractor
    const activation = this.baseModel.infer(imageElement, true);
    return activation;
  }
  
  async train() {
    const featureTensors = tf.stack(this.trainingData.features);
    const labelsTensor = tf.oneHot(
      tf.tensor1d(this.trainingData.labels, 'int32'),
      this.numClasses
    );
    
    // สร้าง classifier layer ง่ายๆ
    this.classifier = tf.sequential();
    this.classifier.add(tf.layers.dense({
      inputShape: [featureTensors.shape[1]],
      units: 100,
      activation: 'relu',
    }));
    this.classifier.add(tf.layers.dense({
      units: this.numClasses,
      activation: 'softmax',
    }));
    
    this.classifier.compile({
      optimizer: tf.train.adam(),
      loss: 'categoricalCrossentropy',
      metrics: ['accuracy'],
    });
    
    await this.classifier.fit(featureTensors, labelsTensor, {
      epochs: 20,
      callbacks: {
        onEpochEnd: (epoch, logs) => {
          console.log(`Epoch ${epoch + 1}: loss=${logs.loss.toFixed(3)}`);
        },
      },
    });
    
    // Cleanup
    featureTensors.dispose();
    labelsTensor.dispose();
    this.trainingData.features.forEach(f => f.dispose());
    
    console.log('Train สำเร็จ!');
  }
  
  async predict(imageElement) {
    const features = await this.extractFeatures(imageElement);
    const prediction = this.classifier.predict(
      tf.expandDims(features, 0)
    );
    
    const data = await prediction.data();
    const maxIndex = data.indexOf(Math.max(...data));
    
    features.dispose();
    prediction.dispose();
    
    return {
      className: this.classNames[maxIndex],
      confidence: (data[maxIndex] * 100).toFixed(1) + '%',
      probabilities: this.classNames.map((name, i) => ({
        name,
        probability: (data[i] * 100).toFixed(1) + '%',
      })),
    };
  }
}

// ใช้งาน: Teach Machine to recognize cats vs dogs
const classifier = new ImageClassifierTransfer();
await classifier.loadBaseModel();
classifier.classNames = ['แมว', 'หมา'];

// เพิ่ม training examples
for (const catImage of catImages) {
  await classifier.addExample(catImage, 0); // 0 = แมว
}

for (const dogImage of dogImages) {
  await classifier.addExample(dogImage, 1); // 1 = หมา
}

// Train
await classifier.train();

// Predict
const result = await classifier.predict(testImage);
console.log(`ผลการทำนาย: ${result.className} (${result.confidence})`);
```

---

## ขั้นตอนที่ 1738: ONNX Runtime สำหรับ JavaScript

```javascript
// ONNX Runtime Web - รัน .onnx models ใน browser
import * as ort from 'onnxruntime-web';

// Configure WebAssembly paths
ort.env.wasm.wasmPaths = 'https://cdn.jsdelivr.net/npm/onnxruntime-web/dist/';

async function runONNXModel() {
  // โหลด ONNX model
  const session = await ort.InferenceSession.create('./model.onnx', {
    executionProviders: ['webgl', 'wasm'], // ลองใช้ GPU ก่อน
  });
  
  console.log('Input names:', session.inputNames);
  console.log('Output names:', session.outputNames);
  
  // สร้าง input tensor
  const inputData = new Float32Array([1.0, 2.0, 3.0, 4.0]);
  const tensor = new ort.Tensor('float32', inputData, [1, 4]);
  
  // Run inference
  const results = await session.run({
    [session.inputNames[0]]: tensor,
  });
  
  const output = results[session.outputNames[0]];
  console.log('Output:', output.data);
  
  return output;
}
```

```javascript
// ONNX กับ Computer Vision
import * as ort from 'onnxruntime-web';

class ONNXVisionModel {
  constructor(modelPath) {
    this.modelPath = modelPath;
    this.session = null;
  }
  
  async load() {
    this.session = await ort.InferenceSession.create(this.modelPath);
  }
  
  preprocessImage(imageElement, targetWidth = 224, targetHeight = 224) {
    const canvas = document.createElement('canvas');
    canvas.width = targetWidth;
    canvas.height = targetHeight;
    
    const ctx = canvas.getContext('2d');
    ctx.drawImage(imageElement, 0, 0, targetWidth, targetHeight);
    
    const imageData = ctx.getImageData(0, 0, targetWidth, targetHeight);
    const { data } = imageData;
    
    // Normalize: [0, 255] -> [0, 1] และแยก channels (R, G, B)
    // ImageNet normalization
    const mean = [0.485, 0.456, 0.406];
    const std = [0.229, 0.224, 0.225];
    
    const float32Data = new Float32Array(3 * targetWidth * targetHeight);
    
    for (let i = 0; i < targetWidth * targetHeight; i++) {
      const r = data[i * 4] / 255;
      const g = data[i * 4 + 1] / 255;
      const b = data[i * 4 + 2] / 255;
      
      float32Data[i] = (r - mean[0]) / std[0];
      float32Data[targetWidth * targetHeight + i] = (g - mean[1]) / std[1];
      float32Data[2 * targetWidth * targetHeight + i] = (b - mean[2]) / std[2];
    }
    
    return new ort.Tensor('float32', float32Data, [1, 3, targetHeight, targetWidth]);
  }
  
  async predict(imageElement) {
    const inputTensor = this.preprocessImage(imageElement);
    
    const outputs = await this.session.run({
      [this.session.inputNames[0]]: inputTensor,
    });
    
    const outputData = outputs[this.session.outputNames[0]].data;
    
    // Softmax
    const expData = Array.from(outputData).map(Math.exp);
    const sumExp = expData.reduce((a, b) => a + b, 0);
    const probabilities = expData.map(e => e / sumExp);
    
    return probabilities;
  }
}
```

---

## ขั้นตอนที่ 1739: Transformers.js

```javascript
// Transformers.js - HuggingFace models ใน browser/Node.js
import { pipeline, env } from '@xenova/transformers';

// กำหนดว่า model เก็บที่ไหน
env.cacheDir = './.cache';

// Text Classification (Sentiment Analysis)
async function analyzeSentiment() {
  const classifier = await pipeline(
    'text-classification',
    'Xenova/bert-base-multilingual-uncased-sentiment'
  );
  
  const results = await classifier([
    'ฉันรักวันนี้มาก! มันสวยงามจริงๆ',
    'วันนี้แย่มาก ทุกอย่างผิดพลาด',
    'ก็โอเคนะ ไม่ดีไม่แย่',
  ]);
  
  results.forEach((result, i) => {
    console.log(`ข้อความ ${i+1}: ${result[0].label} (${(result[0].score * 100).toFixed(1)}%)`);
  });
}
```

```javascript
// Text Generation ด้วย Transformers.js
const generator = await pipeline(
  'text-generation',
  'Xenova/gpt2'
);

const result = await generator('The future of AI is', {
  max_new_tokens: 50,
  temperature: 0.9,
  do_sample: true,
});

console.log(result[0].generated_text);
```

```javascript
// Feature Extraction (Embeddings)
const extractor = await pipeline(
  'feature-extraction',
  'Xenova/all-MiniLM-L6-v2'
);

const sentences = [
  'สุนัขชอบวิ่ง',
  'หมากระโดดเล่น',
  'แมวชอบนอน',
  'ฝนตกวันนี้',
];

const embeddings = await extractor(sentences, {
  pooling: 'mean',
  normalize: true,
});

console.log('Embedding shape:', embeddings.dims);
// [4, 384]

// คำนวณ cosine similarity
function cosineSimilarity(a, b) {
  const dotProduct = a.reduce((sum, ai, i) => sum + ai * b[i], 0);
  const magnitudeA = Math.sqrt(a.reduce((sum, ai) => sum + ai * ai, 0));
  const magnitudeB = Math.sqrt(b.reduce((sum, bi) => sum + bi * bi, 0));
  return dotProduct / (magnitudeA * magnitudeB);
}

const emb = embeddings.tolist();
console.log('ความคล้ายคลึงระหว่าง "สุนัข" และ "หมา":', 
  cosineSimilarity(emb[0], emb[1]).toFixed(3)
); // ประมาณ 0.89

console.log('ความคล้ายคลึงระหว่าง "สุนัข" และ "ฝน":', 
  cosineSimilarity(emb[0], emb[3]).toFixed(3)
); // ประมาณ 0.12
```

```javascript
// Zero-Shot Classification
const classifier = await pipeline(
  'zero-shot-classification',
  'Xenova/mobilebert-uncased-mnli'
);

const text = 'ฉันต้องการซื้อ iPhone ใหม่';
const labels = ['เทคโนโลยี', 'อาหาร', 'กีฬา', 'การเงิน'];

const result = await classifier(text, labels);
console.log(result);
// {
//   labels: ['เทคโนโลยี', 'การเงิน', 'กีฬา', 'อาหาร'],
//   scores: [0.89, 0.07, 0.02, 0.02]
// }
```

---

## ขั้นตอนที่ 1740: WebLLM - LLMs ใน Browser

```javascript
// WebLLM - รัน LLMs ใน browser ด้วย WebGPU
import * as webllm from "@mlc-ai/web-llm";

async function runLLMInBrowser() {
  // สร้าง engine
  const engine = new webllm.MLCEngine();
  
  // โหลด model (ดาวน์โหลด ~3-4GB ครั้งแรก)
  await engine.reload("Llama-3.1-8B-Instruct-q4f32_1-MLC", {
    initProgressCallback: (progress) => {
      console.log(`Loading: ${(progress.progress * 100).toFixed(1)}%`);
    },
  });
  
  console.log('Model โหลดสำเร็จ!');
  
  // Chat completion
  const reply = await engine.chat.completions.create({
    messages: [
      { role: "system", content: "คุณเป็น AI assistant ภาษาไทย" },
      { role: "user", content: "อธิบาย JavaScript ให้ฉันฟังหน่อย" },
    ],
    temperature: 0.7,
    max_tokens: 500,
  });
  
  console.log('AI:', reply.choices[0].message.content);
  
  // Streaming response
  const stream = await engine.chat.completions.create({
    messages: [
      { role: "user", content: "เล่าเรื่องสั้นๆ ให้ฟัง" },
    ],
    stream: true,
  });
  
  let fullText = '';
  for await (const chunk of stream) {
    const delta = chunk.choices[0]?.delta?.content;
    if (delta) {
      process.stdout.write(delta);
      fullText += delta;
    }
  }
  
  console.log('\n\nFull text:', fullText);
}
```

---

## ขั้นตอนที่ 1741: Claude API Integration

```javascript
// ใช้ Claude API จาก JavaScript (Node.js / Browser ผ่าน proxy)
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

// Basic message
async function askClaude(question) {
  const message = await client.messages.create({
    model: 'claude-opus-4-5',
    max_tokens: 1024,
    messages: [
      {
        role: 'user',
        content: question,
      },
    ],
  });
  
  return message.content[0].text;
}

const answer = await askClaude('อธิบาย JavaScript closure ให้เข้าใจง่าย');
console.log(answer);
```

```javascript
// Claude API กับ System Prompt และ Multi-turn
async function multiTurnChat() {
  const messages = [];
  
  async function chat(userMessage) {
    messages.push({ role: 'user', content: userMessage });
    
    const response = await client.messages.create({
      model: 'claude-opus-4-5',
      max_tokens: 1024,
      system: 'คุณเป็น JavaScript tutor ผู้เชี่ยวชาญ ตอบเป็นภาษาไทยเสมอ',
      messages,
    });
    
    const assistantMessage = response.content[0].text;
    messages.push({ role: 'assistant', content: assistantMessage });
    
    return assistantMessage;
  }
  
  console.log(await chat('Promise คืออะไร?'));
  console.log(await chat('แล้ว async/await ต่างกันยังไง?'));
  console.log(await chat('ช่วยยกตัวอย่างโค้ดที่ใช้ทั้งสองอย่างได้ไหม?'));
}
```

```javascript
// Claude API กับ Streaming
async function streamResponse(question) {
  const stream = await client.messages.stream({
    model: 'claude-opus-4-5',
    max_tokens: 1024,
    messages: [{ role: 'user', content: question }],
  });
  
  // Stream ทีละ token
  for await (const chunk of stream) {
    if (chunk.type === 'content_block_delta') {
      process.stdout.write(chunk.delta.text);
    }
  }
  
  const finalMessage = await stream.getFinalMessage();
  console.log('\n\nUsage:', finalMessage.usage);
}
```

```javascript
// Claude API กับ Tool Use
async function claudeWithTools() {
  const tools = [
    {
      name: 'get_weather',
      description: 'Get current weather for a city',
      input_schema: {
        type: 'object',
        properties: {
          city: {
            type: 'string',
            description: 'The city name',
          },
        },
        required: ['city'],
      },
    },
    {
      name: 'calculate',
      description: 'Perform mathematical calculations',
      input_schema: {
        type: 'object',
        properties: {
          expression: {
            type: 'string',
            description: 'Math expression to evaluate',
          },
        },
        required: ['expression'],
      },
    },
  ];
  
  const messages = [
    {
      role: 'user',
      content: 'อากาศที่กรุงเทพเป็นยังไง? และ 125 * 37 เท่ากับเท่าไหร่?',
    },
  ];
  
  // Loop จนกว่า Claude จะตอบสำเร็จ
  while (true) {
    const response = await client.messages.create({
      model: 'claude-opus-4-5',
      max_tokens: 1024,
      tools,
      messages,
    });
    
    if (response.stop_reason === 'end_turn') {
      console.log('Claude ตอบ:', response.content[0].text);
      break;
    }
    
    if (response.stop_reason === 'tool_use') {
      // Process tool calls
      const toolResults = [];
      
      for (const block of response.content) {
        if (block.type === 'tool_use') {
          let result;
          
          if (block.name === 'get_weather') {
            // จำลองการดึงข้อมูลอากาศ
            result = `อากาศที่${block.input.city}: 32°C ร้อน มีเมฆบางส่วน`;
          } else if (block.name === 'calculate') {
            try {
              result = String(eval(block.input.expression));
            } catch {
              result = 'คำนวณไม่ได้';
            }
          }
          
          toolResults.push({
            type: 'tool_result',
            tool_use_id: block.id,
            content: result,
          });
        }
      }
      
      // เพิ่ม assistant message และ tool results
      messages.push({ role: 'assistant', content: response.content });
      messages.push({ role: 'user', content: toolResults });
    }
  }
}
```

---

## ขั้นตอนที่ 1742: OpenAI API Integration

```javascript
// OpenAI API จาก JavaScript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

// Chat completion
async function chatWithGPT(messages) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo',
    messages,
    temperature: 0.7,
    max_tokens: 1000,
  });
  
  return completion.choices[0].message;
}

// Function calling
const response = await openai.chat.completions.create({
  model: 'gpt-4-turbo',
  messages: [
    { role: 'user', content: 'สภาพอากาศที่เชียงใหม่เป็นยังไง?' },
  ],
  tools: [
    {
      type: 'function',
      function: {
        name: 'get_weather',
        description: 'Get weather information for a city',
        parameters: {
          type: 'object',
          properties: {
            city: { type: 'string', description: 'City name' },
            country: { type: 'string', description: 'Country code' },
          },
          required: ['city'],
        },
      },
    },
  ],
  tool_choice: 'auto',
});
```

---

## ขั้นตอนที่ 1743: Building AI Chatbot

```javascript
// Express.js Chatbot API
import express from 'express';
import Anthropic from '@anthropic-ai/sdk';
import { createClient } from 'redis';

const app = express();
const anthropic = new Anthropic();
const redis = createClient({ url: process.env.REDIS_URL });

await redis.connect();

app.use(express.json());

// เก็บ conversation history ใน Redis
async function getConversation(sessionId) {
  const data = await redis.get(`chat:${sessionId}`);
  return data ? JSON.parse(data) : [];
}

async function saveConversation(sessionId, messages) {
  await redis.setEx(
    `chat:${sessionId}`,
    3600, // TTL 1 ชั่วโมง
    JSON.stringify(messages)
  );
}

// Chat endpoint
app.post('/api/chat', async (req, res) => {
  const { sessionId, message } = req.body;
  
  if (!sessionId || !message) {
    return res.status(400).json({ error: 'sessionId และ message จำเป็น' });
  }
  
  // ดึง history
  const history = await getConversation(sessionId);
  
  // เพิ่ม user message
  history.push({ role: 'user', content: message });
  
  try {
    // ส่งไป Claude
    const response = await anthropic.messages.create({
      model: 'claude-opus-4-5',
      max_tokens: 1024,
      system: `คุณเป็น AI assistant ที่เป็นมิตร ตอบเป็นภาษาไทย 
               ถ้าไม่รู้คำตอบให้บอกตรงๆ ไม่ต้องแต่งเรื่อง`,
      messages: history.slice(-20), // ส่ง 20 messages ล่าสุด
    });
    
    const assistantMessage = response.content[0].text;
    
    // เพิ่ม response ไปยัง history
    history.push({ role: 'assistant', content: assistantMessage });
    
    // บันทึก history
    await saveConversation(sessionId, history);
    
    return res.json({
      message: assistantMessage,
      sessionId,
      usage: response.usage,
    });
    
  } catch (error) {
    console.error('AI Error:', error);
    return res.status(500).json({ error: 'ไม่สามารถตอบได้ขณะนี้' });
  }
});

// Streaming chat endpoint
app.post('/api/chat/stream', async (req, res) => {
  const { sessionId, message } = req.body;
  
  const history = await getConversation(sessionId);
  history.push({ role: 'user', content: message });
  
  // Server-Sent Events
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  
  let fullResponse = '';
  
  const stream = await anthropic.messages.stream({
    model: 'claude-opus-4-5',
    max_tokens: 1024,
    messages: history.slice(-20),
  });
  
  for await (const chunk of stream) {
    if (chunk.type === 'content_block_delta') {
      const text = chunk.delta.text;
      fullResponse += text;
      
      res.write(`data: ${JSON.stringify({ text })}\n\n`);
    }
  }
  
  // บันทึก history
  history.push({ role: 'assistant', content: fullResponse });
  await saveConversation(sessionId, history);
  
  res.write('data: [DONE]\n\n');
  res.end();
});

app.listen(3000, () => console.log('Chatbot API running on port 3000'));
```

---

## ขั้นตอนที่ 1744: Image Classification ใน Browser

```javascript
// Complete Image Classification Demo
import * as tf from '@tensorflow/tfjs';
import * as mobilenet from '@tensorflow-models/mobilenet';

const IMAGENET_CLASSES = {
  // มีประมาณ 1000 classes ใน ImageNet
};

class ImageClassificationDemo {
  constructor() {
    this.model = null;
    this.isLoading = false;
  }
  
  async init() {
    this.isLoading = true;
    this.updateUI('กำลังโหลด Model...');
    
    // โหลด MobileNet
    this.model = await mobilenet.load({
      version: 2,
      alpha: 1.0, // 0.25, 0.50, 0.75, 1.0 - ขนาด model
    });
    
    this.isLoading = false;
    this.updateUI('พร้อมใช้งาน! อัปโหลดรูปภาพ');
    
    console.log('TF.js backend:', tf.getBackend());
  }
  
  async classifyImage(imageElement) {
    if (!this.model) await this.init();
    
    // วัดเวลา
    const startTime = performance.now();
    
    const predictions = await this.model.classify(imageElement, 5);
    
    const elapsed = (performance.now() - startTime).toFixed(1);
    
    return {
      predictions: predictions.map(p => ({
        class: p.className,
        probability: (p.probability * 100).toFixed(2) + '%',
        bar: Math.round(p.probability * 100),
      })),
      inferenceTime: elapsed + 'ms',
    };
  }
  
  updateUI(message) {
    const statusEl = document.getElementById('status');
    if (statusEl) statusEl.textContent = message;
  }
  
  displayResults(results, container) {
    container.innerHTML = `
      <div class="results">
        <p>เวลาในการวิเคราะห์: ${results.inferenceTime}</p>
        ${results.predictions.map(p => `
          <div class="prediction">
            <span class="label">${p.class}</span>
            <div class="bar-container">
              <div class="bar" style="width: ${p.bar}%"></div>
            </div>
            <span class="probability">${p.probability}</span>
          </div>
        `).join('')}
      </div>
    `;
  }
}

// Initialize
const demo = new ImageClassificationDemo();
await demo.init();

// Event listeners
document.getElementById('imageUpload').addEventListener('change', async (e) => {
  const file = e.target.files[0];
  if (!file) return;
  
  const reader = new FileReader();
  reader.onload = async (event) => {
    const img = document.getElementById('preview');
    img.src = event.target.result;
    
    img.onload = async () => {
      const results = await demo.classifyImage(img);
      demo.displayResults(results, document.getElementById('results'));
    };
  };
  
  reader.readAsDataURL(file);
});
```

---

## ขั้นตอนที่ 1745: Object Detection

```javascript
// Real-time Object Detection
import * as tf from '@tensorflow/tfjs';
import * as cocoSsd from '@tensorflow-models/coco-ssd';

class RealTimeDetector {
  constructor(videoEl, canvasEl) {
    this.video = videoEl;
    this.canvas = canvasEl;
    this.ctx = canvasEl.getContext('2d');
    this.model = null;
    this.running = false;
    this.detectionCount = {};
  }
  
  async init() {
    this.model = await cocoSsd.load({ base: 'mobilenet_v2' });
    console.log('COCO-SSD model โหลดสำเร็จ');
  }
  
  async startCamera() {
    const stream = await navigator.mediaDevices.getUserMedia({
      video: { width: 640, height: 480 },
    });
    
    this.video.srcObject = stream;
    await new Promise(resolve => {
      this.video.onloadedmetadata = resolve;
    });
    
    this.canvas.width = this.video.videoWidth;
    this.canvas.height = this.video.videoHeight;
  }
  
  start() {
    this.running = true;
    this.detectLoop();
  }
  
  stop() {
    this.running = false;
  }
  
  async detectLoop() {
    if (!this.running) return;
    
    this.ctx.drawImage(this.video, 0, 0);
    
    const predictions = await this.model.detect(this.video);
    
    this.drawPredictions(predictions);
    this.updateStats(predictions);
    
    requestAnimationFrame(() => this.detectLoop());
  }
  
  drawPredictions(predictions) {
    predictions.forEach(pred => {
      const [x, y, width, height] = pred.bbox;
      const score = (pred.score * 100).toFixed(1);
      
      // Bounding box
      this.ctx.strokeStyle = this.getColorForClass(pred.class);
      this.ctx.lineWidth = 2;
      this.ctx.strokeRect(x, y, width, height);
      
      // Background for label
      this.ctx.fillStyle = this.getColorForClass(pred.class);
      const labelWidth = this.ctx.measureText(`${pred.class} ${score}%`).width + 10;
      this.ctx.fillRect(x, y - 25, labelWidth, 20);
      
      // Label text
      this.ctx.fillStyle = 'white';
      this.ctx.font = '14px Arial';
      this.ctx.fillText(`${pred.class} ${score}%`, x + 5, y - 10);
    });
  }
  
  getColorForClass(className) {
    const colors = {
      person: '#FF0000',
      car: '#00FF00',
      cat: '#FF00FF',
      dog: '#FFFF00',
      chair: '#00FFFF',
    };
    return colors[className] || '#FFFFFF';
  }
  
  updateStats(predictions) {
    this.detectionCount = {};
    predictions.forEach(p => {
      this.detectionCount[p.class] = (this.detectionCount[p.class] || 0) + 1;
    });
    
    const statsEl = document.getElementById('stats');
    if (statsEl) {
      statsEl.innerHTML = Object.entries(this.detectionCount)
        .map(([cls, count]) => `<span>${cls}: ${count}</span>`)
        .join(', ');
    }
  }
}
```

---

## ขั้นตอนที่ 1746: Natural Language Processing

```javascript
// NLP Tasks ด้วย Transformers.js
import { pipeline } from '@xenova/transformers';

// Named Entity Recognition
async function extractEntities(text) {
  const ner = await pipeline('ner', 'Xenova/bert-base-NER');
  
  const entities = await ner(text, { aggregation_strategy: 'simple' });
  
  return entities.map(entity => ({
    text: entity.word,
    type: entity.entity_group,
    score: (entity.score * 100).toFixed(1) + '%',
    start: entity.start,
    end: entity.end,
  }));
}

const text = 'Apple Inc. was founded by Steve Jobs in Cupertino, California in 1976.';
const entities = await extractEntities(text);
console.log(entities);
// [
//   { text: 'Apple Inc.', type: 'ORG', score: '99.2%' },
//   { text: 'Steve Jobs', type: 'PER', score: '98.9%' },
//   { text: 'Cupertino', type: 'LOC', score: '97.5%' },
//   { text: 'California', type: 'LOC', score: '98.1%' },
// ]
```

```javascript
// Question Answering
const qa = await pipeline('question-answering', 'Xenova/distilbert-base-uncased-distilled-squad');

const context = `JavaScript เป็นภาษาโปรแกรมที่ถูกสร้างโดย Brendan Eich ในปี 1995 
                 และเผยแพร่ครั้งแรกใน Netscape Navigator 2.0`;

const questions = [
  'ใครสร้าง JavaScript?',
  'JavaScript ถูกสร้างขึ้นปีใด?',
  'JavaScript เผยแพร่ใน browser อะไร?',
];

for (const question of questions) {
  const result = await qa({ question, context });
  console.log(`Q: ${question}`);
  console.log(`A: ${result.answer} (confidence: ${(result.score * 100).toFixed(1)}%)`);
}
```

```javascript
// Translation
const translator = await pipeline(
  'translation',
  'Helsinki-NLP/opus-mt-en-th'
);

const texts = [
  'Hello, how are you?',
  'JavaScript is a programming language',
  'I love learning new things',
];

const translations = await translator(texts);
translations.forEach((t, i) => {
  console.log(`EN: ${texts[i]}`);
  console.log(`TH: ${t.translation_text}`);
  console.log();
});
```

---

## ขั้นตอนที่ 1747: Sentiment Analysis

```javascript
// Sentiment Analysis Application
import { pipeline } from '@xenova/transformers';

class SentimentAnalyzer {
  constructor() {
    this.model = null;
  }
  
  async init() {
    this.model = await pipeline(
      'sentiment-analysis',
      'Xenova/distilbert-base-uncased-finetuned-sst-2-english'
    );
    console.log('Sentiment model โหลดสำเร็จ');
  }
  
  async analyze(text) {
    if (!this.model) await this.init();
    
    const result = await this.model(text);
    const { label, score } = result[0];
    
    return {
      text,
      sentiment: label === 'POSITIVE' ? 'เชิงบวก' : 'เชิงลบ',
      originalLabel: label,
      confidence: (score * 100).toFixed(1),
      emoji: label === 'POSITIVE' ? '😊' : '😞',
    };
  }
  
  async analyzeMultiple(texts) {
    const results = await Promise.all(texts.map(t => this.analyze(t)));
    
    const summary = {
      total: texts.length,
      positive: results.filter(r => r.originalLabel === 'POSITIVE').length,
      negative: results.filter(r => r.originalLabel === 'NEGATIVE').length,
    };
    
    summary.positiveRate = ((summary.positive / summary.total) * 100).toFixed(1) + '%';
    
    return { results, summary };
  }
}

// Product Review Analyzer
const analyzer = new SentimentAnalyzer();
await analyzer.init();

const reviews = [
  'สินค้าดีมาก ส่งเร็ว บรรจุภัณฑ์สวย แนะนำมากๆ',
  'ของมาช้ามาก คุณภาพไม่ดีเลย ผิดหวัง',
  'โอเคนะ ราคาเหมาะสม ไม่ได้ดีเป็นพิเศษ',
  'Perfect! Exactly as described. Will buy again.',
  'Very disappointed. Not as shown in pictures.',
];

const { results, summary } = await analyzer.analyzeMultiple(reviews);

console.log('สรุปความคิดเห็น:');
console.log(`Total: ${summary.total} reviews`);
console.log(`เชิงบวก: ${summary.positive} (${summary.positiveRate})`);
console.log(`เชิงลบ: ${summary.negative}`);
console.log('\nรายละเอียด:');
results.forEach(r => {
  console.log(`${r.emoji} [${r.sentiment} ${r.confidence}%] "${r.text.substring(0, 50)}..."`);
});
```

---

## ขั้นตอนที่ 1748: Text Generation

```javascript
// Text Generation กับ Claude API (Node.js)
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

class ContentGenerator {
  constructor() {
    this.client = client;
  }
  
  async generateBlogPost(topic, keywords, wordCount = 500) {
    const message = await this.client.messages.create({
      model: 'claude-opus-4-5',
      max_tokens: 2000,
      messages: [{
        role: 'user',
        content: `เขียนบทความบล็อกเกี่ยวกับ "${topic}"
                  ใช้ keywords เหล่านี้: ${keywords.join(', ')}
                  ความยาวประมาณ ${wordCount} คำ
                  ภาษาไทย ให้น่าอ่านและมีประโยชน์
                  มี heading หลักและ subheadings`,
      }],
    });
    
    return message.content[0].text;
  }
  
  async generateProductDescription(product) {
    const message = await this.client.messages.create({
      model: 'claude-opus-4-5',
      max_tokens: 500,
      messages: [{
        role: 'user',
        content: `เขียน product description สำหรับ:
                  ชื่อสินค้า: ${product.name}
                  ราคา: ${product.price} บาท
                  คุณสมบัติ: ${product.features.join(', ')}
                  กลุ่มเป้าหมาย: ${product.targetAudience}
                  
                  ให้น่าสนใจ ดึงดูดให้ซื้อ ไม่เกิน 150 คำ`,
      }],
    });
    
    return message.content[0].text;
  }
  
  async summarizeText(text, maxLength = 200) {
    const message = await this.client.messages.create({
      model: 'claude-haiku-4-5',
      max_tokens: maxLength * 2,
      messages: [{
        role: 'user',
        content: `สรุปข้อความต่อไปนี้ให้กระชับ ไม่เกิน ${maxLength} คำ ภาษาไทย:

${text}`,
      }],
    });
    
    return message.content[0].text;
  }
}

// ใช้งาน
const generator = new ContentGenerator();

const blogPost = await generator.generateBlogPost(
  'JavaScript Performance Optimization',
  ['lazy loading', 'code splitting', 'memoization', 'Web Workers'],
  800
);

console.log('Blog Post:\n', blogPost);
```

---

## ขั้นตอนที่ 1749: Embeddings และ Vector Search

```javascript
// Semantic Search ด้วย Embeddings
import { pipeline } from '@xenova/transformers';

class SemanticSearch {
  constructor() {
    this.embedder = null;
    this.documents = [];
    this.embeddings = [];
  }
  
  async init() {
    this.embedder = await pipeline(
      'feature-extraction',
      'Xenova/all-MiniLM-L6-v2'
    );
  }
  
  async embed(text) {
    const output = await this.embedder(text, {
      pooling: 'mean',
      normalize: true,
    });
    return Array.from(output.data);
  }
  
  async addDocuments(documents) {
    for (const doc of documents) {
      const embedding = await this.embed(doc.text);
      this.documents.push(doc);
      this.embeddings.push(embedding);
    }
    
    console.log(`เพิ่ม ${documents.length} documents แล้ว`);
  }
  
  cosineSimilarity(a, b) {
    const dotProduct = a.reduce((sum, ai, i) => sum + ai * b[i], 0);
    return dotProduct; // normalized vectors, dot product = cosine similarity
  }
  
  async search(query, topK = 3) {
    const queryEmbedding = await this.embed(query);
    
    // คำนวณ similarity กับทุก documents
    const similarities = this.embeddings.map((docEmb, i) => ({
      index: i,
      score: this.cosineSimilarity(queryEmbedding, docEmb),
      document: this.documents[i],
    }));
    
    // เรียงลำดับตาม score
    similarities.sort((a, b) => b.score - a.score);
    
    return similarities.slice(0, topK).map(item => ({
      ...item.document,
      score: (item.score * 100).toFixed(1) + '%',
    }));
  }
}

// ตัวอย่าง: ค้นหาคำถามจาก FAQ
const search = new SemanticSearch();
await search.init();

const faqItems = [
  {
    id: 1,
    question: 'วิธีตั้งค่า 2FA',
    text: 'คุณสามารถเปิดใช้งาน Two-Factor Authentication ได้จากหน้า Account Settings > Security',
  },
  {
    id: 2,
    question: 'วิธีเปลี่ยนรหัสผ่าน',
    text: 'ไปที่ Profile > Change Password แล้วกรอกรหัสผ่านเก่าและใหม่',
  },
  {
    id: 3,
    question: 'วิธียกเลิกสมาชิก',
    text: 'ไปที่ Account > Subscription > Cancel Membership',
  },
  {
    id: 4,
    question: 'วิธีดูใบเสร็จ',
    text: 'ดูประวัติการชำระเงินได้ที่ Billing > Invoice History',
  },
];

await search.addDocuments(faqItems);

// ค้นหา
const results = await search.search('ฉันอยากเพิ่มความปลอดภัยบัญชี');
console.log('ผลการค้นหา:');
results.forEach(r => {
  console.log(`[${r.score}] Q: ${r.question}`);
  console.log(`A: ${r.text}\n`);
});
// [95.3%] Q: วิธีตั้งค่า 2FA
// A: คุณสามารถเปิดใช้งาน Two-Factor Authentication...
```

---

## ขั้นตอนที่ 1750: Image Generation API

```javascript
// ใช้ DALL-E API จาก OpenAI
import OpenAI from 'openai';
import fs from 'fs';

const openai = new OpenAI();

async function generateImage(prompt, options = {}) {
  const {
    model = 'dall-e-3',
    size = '1024x1024',
    quality = 'standard',
    n = 1,
    style = 'natural',
  } = options;
  
  const response = await openai.images.generate({
    model,
    prompt,
    size,
    quality,
    n,
    style,
  });
  
  return response.data.map(img => ({
    url: img.url,
    revisedPrompt: img.revised_prompt,
  }));
}

// ใช้งาน
const images = await generateImage(
  'ภาพวิวภูเขาในยามเช้า มีหมอกลอยอยู่ ท้องฟ้าสีส้ม สวยงามสไตล์ photography',
  { quality: 'hd', style: 'natural' }
);

console.log('Generated Image URL:', images[0].url);
console.log('Revised Prompt:', images[0].revisedPrompt);

// ดาวน์โหลดรูปภาพ
async function downloadImage(url, filename) {
  const response = await fetch(url);
  const buffer = await response.arrayBuffer();
  fs.writeFileSync(filename, Buffer.from(buffer));
  console.log(`บันทึกรูปภาพที่: ${filename}`);
}

await downloadImage(images[0].url, 'mountain.png');
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Emotion Detector
สร้าง web app ที่:
- ใช้ webcam เพื่อถ่ายรูปใบหน้า
- วิเคราะห์อารมณ์ด้วย TensorFlow.js
- แสดงผล real-time บน canvas
- บันทึก emotion history

### แบบฝึกหัดที่ 2: Thai Document Summarizer
สร้าง app ที่:
- รับข้อความภาษาไทย
- ส่งไป Claude API เพื่อสรุป
- แสดง summary พร้อม key points
- มี streaming output

### แบบฝึกหัดที่ 3: Product Review Analyzer
สร้าง dashboard ที่:
- รับ reviews หลายๆ อัน
- วิเคราะห์ sentiment แต่ละ review
- แสดง statistics (เชิงบวก/เชิงลบ/เฉยๆ)
- หา common themes

### แบบฝึกหัดที่ 4: Semantic FAQ Search
สร้าง FAQ system ที่:
- เก็บ FAQ ใน vector database (ในหน่วยความจำ)
- รับคำถามจากผู้ใช้
- ค้นหาคำตอบที่ใกล้เคียงด้วย embeddings
- ถ้าไม่พบ ส่งไป Claude API

### แบบฝึกหัดที่ 5: AI Image Classifier
สร้าง app ที่:
- รองรับ drag-and-drop รูปภาพ
- จำแนกประเภทรูปด้วย MobileNet
- แสดง top-5 predictions พร้อม confidence bars
- ทำ Transfer Learning กับ custom categories

---

## สรุป

| Library | Use Case | Size | Speed |
|---------|----------|------|-------|
| TensorFlow.js | General ML, Training | 4MB+ | เร็ว (WebGL) |
| ONNX Runtime Web | Pre-trained models | ~7MB | เร็ว (WebAssembly) |
| Transformers.js | NLP tasks | ขึ้นกับ model | ปานกลาง |
| WebLLM | LLMs in browser | 3-8GB model | ช้า (hardware dependent) |
| Claude/OpenAI API | Complex tasks | N/A (server) | เร็ว (network) |

ในส่วนถัดไปเราจะเรียนเรื่อง Real-time Applications!
