# Part 95: Blockchain กับ JavaScript (Steps 1871-1890)

## บทนำ

Blockchain เป็นเทคโนโลยีที่เปลี่ยนแปลงโลกการเงินและระบบข้อมูลอย่างมาก ในบทนี้เราจะเรียนรู้ตั้งแต่พื้นฐาน Blockchain ไปจนถึงการสร้าง DApp (Decentralized Application) ด้วย JavaScript โดยใช้ภาษาไทยตลอดทั้งบท

---

## Step 1871: Blockchain คืออะไร?

Blockchain คือฐานข้อมูลแบบกระจายศูนย์ (Distributed Ledger) ที่เก็บข้อมูลในรูปแบบ "บล็อก" ที่เชื่อมต่อกันเป็น "ห่วงโซ่" แต่ละบล็อกมีการเข้ารหัสและอ้างอิงถึงบล็อกก่อนหน้า ทำให้ไม่สามารถแก้ไขข้อมูลเก่าได้โดยไม่ถูกตรวจจับ

### คุณสมบัติหลักของ Blockchain

1. **Immutability (ไม่เปลี่ยนแปลงได้)** - เมื่อบันทึกข้อมูลแล้วจะเปลี่ยนไม่ได้
2. **Decentralization (กระจายศูนย์)** - ไม่มีผู้ควบคุมกลาง
3. **Transparency (โปร่งใส)** - ทุกคนสามารถตรวจสอบได้
4. **Security (ปลอดภัย)** - ใช้การเข้ารหัสป้องกันการปลอมแปลง

### โครงสร้างของ Block

```javascript
// โครงสร้างพื้นฐานของ Block
const block = {
  index: 0,              // ลำดับที่ของบล็อก
  timestamp: Date.now(), // เวลาที่สร้างบล็อก
  data: {},              // ข้อมูลที่เก็บในบล็อก
  previousHash: "0",     // hash ของบล็อกก่อนหน้า
  hash: "abc123...",     // hash ของบล็อกนี้
  nonce: 0               // ตัวเลขสำหรับ Proof of Work
};
```

### โครงสร้างของ Blockchain

```javascript
// Blockchain คือ array ของ blocks ที่เชื่อมกัน
const blockchain = [
  { index: 0, previousHash: "0",      hash: "genesis..." }, // Genesis Block
  { index: 1, previousHash: "genesis...", hash: "block1..." },
  { index: 2, previousHash: "block1...", hash: "block2..." },
];
```

---

## Step 1872: Decentralization และ Consensus Mechanisms

### Decentralization (การกระจายศูนย์)

ใน Blockchain แบบดั้งเดิม (เช่น Bitcoin, Ethereum) ไม่มีเซิร์ฟเวอร์กลาง แต่ละ Node (คอมพิวเตอร์ในเครือข่าย) เก็บสำเนาของ Blockchain ทั้งหมด

```javascript
// จำลองแนวคิด Node ใน Blockchain Network
class Node {
  constructor(id, peers = []) {
    this.id = id;
    this.peers = peers;      // รายชื่อ Nodes อื่นในเครือข่าย
    this.blockchain = [];    // สำเนา blockchain ของ node นี้
  }

  // รับ block ใหม่และกระจายไปยัง peers
  receiveBlock(block) {
    if (this.validateBlock(block)) {
      this.blockchain.push(block);
      this.broadcastToPeers(block);
    }
  }

  broadcastToPeers(block) {
    this.peers.forEach(peer => peer.receiveBlock(block));
  }

  validateBlock(block) {
    // ตรวจสอบว่า block ถูกต้องหรือไม่
    return block.previousHash === this.getLastHash();
  }

  getLastHash() {
    if (this.blockchain.length === 0) return "0";
    return this.blockchain[this.blockchain.length - 1].hash;
  }
}
```

### Consensus Mechanisms

**1. Proof of Work (PoW)** - Bitcoin ใช้วิธีนี้
```javascript
// จำลอง Proof of Work
function proofOfWork(blockData, difficulty) {
  let nonce = 0;
  let hash = "";
  const prefix = "0".repeat(difficulty); // เช่น "0000"

  // หา nonce ที่ทำให้ hash ขึ้นต้นด้วย zeros
  while (!hash.startsWith(prefix)) {
    nonce++;
    hash = calculateHash({ ...blockData, nonce });
  }

  console.log(`พบ nonce: ${nonce}, hash: ${hash}`);
  return { nonce, hash };
}
```

**2. Proof of Stake (PoS)** - Ethereum 2.0 ใช้วิธีนี้
```javascript
// จำลองแนวคิด Proof of Stake
class Validator {
  constructor(address, stake) {
    this.address = address;
    this.stake = stake; // จำนวน ETH ที่ stake ไว้
  }
}

// เลือก validator ตาม stake (โอกาสสูงกว่าถ้า stake มากกว่า)
function selectValidator(validators) {
  const totalStake = validators.reduce((sum, v) => sum + v.stake, 0);
  let random = Math.random() * totalStake;

  for (const validator of validators) {
    random -= validator.stake;
    if (random <= 0) return validator;
  }
}

const validators = [
  new Validator("0xAlice", 32),
  new Validator("0xBob", 64),
  new Validator("0xCarol", 16),
];

const chosen = selectValidator(validators);
console.log(`Validator ที่เลือก: ${chosen.address}`);
```

---

## Step 1873: สร้าง Block Class ด้วย JavaScript

```javascript
const crypto = require("crypto");

class Block {
  constructor(index, timestamp, data, previousHash = "") {
    this.index = index;
    this.timestamp = timestamp;
    this.data = data;
    this.previousHash = previousHash;
    this.nonce = 0;
    this.hash = this.calculateHash();
  }

  // คำนวณ SHA-256 hash
  calculateHash() {
    const content =
      this.index +
      this.timestamp +
      this.previousHash +
      JSON.stringify(this.data) +
      this.nonce;

    return crypto.createHash("sha256").update(content).digest("hex");
  }

  // ทำ Mining ด้วย Proof of Work
  mineBlock(difficulty) {
    const prefix = "0".repeat(difficulty);

    while (!this.hash.startsWith(prefix)) {
      this.nonce++;
      this.hash = this.calculateHash();
    }

    console.log(`Block ${this.index} mined: ${this.hash}`);
    console.log(`Nonce ที่ใช้: ${this.nonce}`);
  }

  // แสดงข้อมูล block
  toString() {
    return JSON.stringify(
      {
        index: this.index,
        timestamp: new Date(this.timestamp).toISOString(),
        data: this.data,
        previousHash: this.previousHash.substring(0, 20) + "...",
        hash: this.hash.substring(0, 20) + "...",
        nonce: this.nonce,
      },
      null,
      2
    );
  }
}

// ทดสอบสร้าง block
const block = new Block(1, Date.now(), { message: "Hello Blockchain!" }, "0");
console.log("Hash ก่อน mine:", block.hash);
block.mineBlock(3); // difficulty = 3 (hash ต้องขึ้นต้นด้วย "000")
console.log("Hash หลัง mine:", block.hash);
```

---

## Step 1874: สร้าง Blockchain Class

```javascript
const crypto = require("crypto");

class Block {
  constructor(index, timestamp, data, previousHash = "") {
    this.index = index;
    this.timestamp = timestamp;
    this.data = data;
    this.previousHash = previousHash;
    this.nonce = 0;
    this.hash = this.calculateHash();
  }

  calculateHash() {
    return crypto
      .createHash("sha256")
      .update(
        this.index +
          this.timestamp +
          this.previousHash +
          JSON.stringify(this.data) +
          this.nonce
      )
      .digest("hex");
  }

  mineBlock(difficulty) {
    const prefix = "0".repeat(difficulty);
    while (!this.hash.startsWith(prefix)) {
      this.nonce++;
      this.hash = this.calculateHash();
    }
    console.log(`✓ Block ${this.index} mined: ${this.hash.substring(0, 30)}...`);
  }
}

class Blockchain {
  constructor() {
    this.chain = [this.createGenesisBlock()];
    this.difficulty = 3;
    this.pendingTransactions = [];
    this.miningReward = 100; // รางวัลสำหรับ miner
  }

  // สร้าง Genesis Block (บล็อกแรก)
  createGenesisBlock() {
    return new Block(0, Date.now(), "Genesis Block", "0");
  }

  // ดึง block ล่าสุด
  getLatestBlock() {
    return this.chain[this.chain.length - 1];
  }

  // เพิ่ม block ใหม่
  addBlock(newBlock) {
    newBlock.previousHash = this.getLatestBlock().hash;
    newBlock.mineBlock(this.difficulty);
    this.chain.push(newBlock);
  }

  // ตรวจสอบความถูกต้องของ chain
  isChainValid() {
    for (let i = 1; i < this.chain.length; i++) {
      const currentBlock = this.chain[i];
      const previousBlock = this.chain[i - 1];

      // ตรวจสอบ hash ของ block ปัจจุบัน
      if (currentBlock.hash !== currentBlock.calculateHash()) {
        console.error(`Block ${i}: hash ไม่ถูกต้อง`);
        return false;
      }

      // ตรวจสอบการเชื่อมต่อกับ block ก่อนหน้า
      if (currentBlock.previousHash !== previousBlock.hash) {
        console.error(`Block ${i}: previousHash ไม่ตรงกัน`);
        return false;
      }
    }
    return true;
  }

  // แสดงสรุป chain
  printChain() {
    this.chain.forEach((block, index) => {
      console.log(`\n=== Block ${index} ===`);
      console.log(`Hash: ${block.hash.substring(0, 40)}...`);
      console.log(`PrevHash: ${block.previousHash.substring(0, 40)}...`);
      console.log(`Data:`, block.data);
      console.log(`Nonce: ${block.nonce}`);
    });
  }
}

// ทดสอบ Blockchain
const myChain = new Blockchain();

console.log("กำลัง mining block 1...");
myChain.addBlock(new Block(1, Date.now(), { from: "Alice", to: "Bob", amount: 50 }));

console.log("\nกำลัง mining block 2...");
myChain.addBlock(new Block(2, Date.now(), { from: "Bob", to: "Carol", amount: 25 }));

console.log("\nกำลัง mining block 3...");
myChain.addBlock(
  new Block(3, Date.now(), { from: "Carol", to: "Dave", amount: 10 })
);

myChain.printChain();
console.log("\nBlockchain ถูกต้องหรือไม่?", myChain.isChainValid());

// ทดลองแก้ไขข้อมูล (จะทำให้ chain ไม่ถูกต้อง)
console.log("\n--- ทดลองแก้ไขข้อมูล ---");
myChain.chain[1].data = { from: "Alice", to: "Bob", amount: 9999 };
console.log("หลังแก้ไข, blockchain ถูกต้องหรือไม่?", myChain.isChainValid());
```

---

## Step 1875: Merkle Tree Implementation

Merkle Tree ใช้ใน Bitcoin และ Ethereum เพื่อสรุป transactions ทั้งหมดใน block อย่างมีประสิทธิภาพ

```javascript
const crypto = require("crypto");

function sha256(data) {
  return crypto.createHash("sha256").update(data).digest("hex");
}

class MerkleTree {
  constructor(transactions) {
    this.transactions = transactions;
    this.root = this.buildTree(transactions);
  }

  // สร้าง Merkle Root จาก transactions
  buildTree(data) {
    if (data.length === 0) return "";
    if (data.length === 1) return sha256(data[0]);

    // Hash แต่ละ transaction
    let layer = data.map(tx => sha256(JSON.stringify(tx)));

    // รวม pairs จนเหลือ 1 hash
    while (layer.length > 1) {
      const nextLayer = [];

      for (let i = 0; i < layer.length; i += 2) {
        if (i + 1 < layer.length) {
          // รวม 2 hash เข้าด้วยกัน
          nextLayer.push(sha256(layer[i] + layer[i + 1]));
        } else {
          // ถ้าเหลือ hash เดียว (จำนวนคี่) ให้ duplicate
          nextLayer.push(sha256(layer[i] + layer[i]));
        }
      }

      layer = nextLayer;
    }

    return layer[0]; // Merkle Root
  }

  // ตรวจสอบว่า transaction อยู่ใน tree หรือไม่ (Merkle Proof)
  getProof(transaction) {
    const txHash = sha256(JSON.stringify(transaction));
    let layer = this.transactions.map(tx => sha256(JSON.stringify(tx)));
    const proof = [];

    let index = layer.indexOf(txHash);
    if (index === -1) return null;

    while (layer.length > 1) {
      const nextLayer = [];
      const nextProof = [];

      for (let i = 0; i < layer.length; i += 2) {
        const left = layer[i];
        const right = i + 1 < layer.length ? layer[i + 1] : layer[i];

        nextLayer.push(sha256(left + right));

        if (i === index || i + 1 === index) {
          // บันทึก sibling hash สำหรับ proof
          const siblingIndex = index % 2 === 0 ? index + 1 : index - 1;
          if (siblingIndex < layer.length) {
            proof.push({
              hash: layer[siblingIndex],
              position: index % 2 === 0 ? "right" : "left",
            });
          }
        }
      }

      index = Math.floor(index / 2);
      layer = nextLayer;
    }

    return { root: layer[0], proof };
  }
}

// ทดสอบ Merkle Tree
const transactions = [
  { from: "Alice", to: "Bob", amount: 10 },
  { from: "Bob", to: "Carol", amount: 5 },
  { from: "Carol", to: "Dave", amount: 3 },
  { from: "Dave", to: "Eve", amount: 8 },
];

const tree = new MerkleTree(transactions);
console.log("Merkle Root:", tree.root);

const proof = tree.getProof(transactions[0]);
console.log("\nProof สำหรับ transaction แรก:");
console.log(JSON.stringify(proof, null, 2));
```

---

## Step 1876: Smart Contracts คืออะไร?

Smart Contract คือโปรแกรมที่ทำงานบน Blockchain โดยอัตโนมัติเมื่อเงื่อนไขถูกตรงตาม ไม่ต้องการตัวกลาง

```javascript
// จำลองแนวคิด Smart Contract ด้วย JavaScript
class SimpleEscrowContract {
  constructor(buyer, seller, amount) {
    this.buyer = buyer;
    this.seller = seller;
    this.amount = amount;
    this.state = "CREATED"; // CREATED, FUNDED, COMPLETED, REFUNDED
    this.balance = 0;
    this.createdAt = Date.now();
    this.deadline = Date.now() + 7 * 24 * 60 * 60 * 1000; // 7 วัน
  }

  // Buyer ฝากเงิน
  fund(from, amount) {
    if (from !== this.buyer) throw new Error("เฉพาะ buyer เท่านั้น");
    if (this.state !== "CREATED") throw new Error("ต้องอยู่ใน state CREATED");
    if (amount !== this.amount) throw new Error("จำนวนเงินไม่ถูกต้อง");

    this.balance = amount;
    this.state = "FUNDED";
    console.log(`✓ Buyer ฝากเงิน ${amount} ETH เข้า escrow`);
  }

  // Buyer ยืนยันรับของแล้ว → โอนเงินให้ seller
  confirmDelivery(from) {
    if (from !== this.buyer) throw new Error("เฉพาะ buyer เท่านั้น");
    if (this.state !== "FUNDED") throw new Error("ต้องอยู่ใน state FUNDED");

    this.state = "COMPLETED";
    console.log(`✓ โอน ${this.balance} ETH ให้ ${this.seller}`);
    this.balance = 0;
  }

  // คืนเงินถ้าหมดเวลา
  refund(from) {
    if (from !== this.buyer && from !== this.seller) {
      throw new Error("เฉพาะ buyer หรือ seller เท่านั้น");
    }
    if (Date.now() < this.deadline) throw new Error("ยังไม่หมดเวลา");
    if (this.state !== "FUNDED") throw new Error("ไม่มีเงินใน escrow");

    this.state = "REFUNDED";
    console.log(`✓ คืนเงิน ${this.balance} ETH ให้ ${this.buyer}`);
    this.balance = 0;
  }

  getStatus() {
    return {
      state: this.state,
      balance: this.balance,
      buyer: this.buyer,
      seller: this.seller,
    };
  }
}

// ทดสอบ
const escrow = new SimpleEscrowContract("Alice", "Bob", 1.5);
escrow.fund("Alice", 1.5);
console.log("Status:", escrow.getStatus());
escrow.confirmDelivery("Alice");
console.log("Status หลังยืนยัน:", escrow.getStatus());
```

---

## Step 1877: Ethereum และ EVM Basics

### Ethereum คืออะไร?

Ethereum เป็น Blockchain platform ที่รองรับ Smart Contracts ใช้ภาษา Solidity ในการเขียน Smart Contracts ซึ่งทำงานบน Ethereum Virtual Machine (EVM)

```javascript
// ตัวอย่าง Smart Contract ภาษา Solidity (เพื่อทำความเข้าใจ)
/*
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract SimpleStorage {
    uint256 private value;
    address public owner;
    
    event ValueChanged(uint256 oldValue, uint256 newValue);
    
    constructor(uint256 _initialValue) {
        value = _initialValue;
        owner = msg.sender;
    }
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner can call this");
        _;
    }
    
    function setValue(uint256 _value) public onlyOwner {
        uint256 oldValue = value;
        value = _value;
        emit ValueChanged(oldValue, _value);
    }
    
    function getValue() public view returns (uint256) {
        return value;
    }
}
*/

// จำลอง contract state ด้วย JavaScript เพื่อทำความเข้าใจ
class SimpleStorage {
  constructor(initialValue, deployer) {
    this.value = initialValue;
    this.owner = deployer;
    this.events = [];
  }

  setValue(newValue, caller) {
    if (caller !== this.owner) {
      throw new Error("Only owner can call this");
    }
    const oldValue = this.value;
    this.value = newValue;
    this.events.push({
      event: "ValueChanged",
      oldValue,
      newValue,
      timestamp: Date.now(),
    });
    console.log(`ValueChanged: ${oldValue} → ${newValue}`);
  }

  getValue() {
    return this.value;
  }
}

// Deploy contract (จำลอง)
const contract = new SimpleStorage(42, "0xOwnerAddress");
console.log("Current value:", contract.getValue());
contract.setValue(100, "0xOwnerAddress");
console.log("New value:", contract.getValue());
console.log("Events:", contract.events);
```

---

## Step 1878: ethers.js - ติดตั้งและตั้งค่า

```bash
# ติดตั้ง ethers.js
npm install ethers

# หรือใช้ yarn
yarn add ethers
```

```javascript
// import แบบ ES Module
import { ethers } from "ethers";

// หรือ CommonJS
const { ethers } = require("ethers");

// ตรวจสอบ version
console.log("ethers version:", ethers.version);

// ประเภท Provider ที่ใช้บ่อย
// 1. JsonRpcProvider - เชื่อมต่อกับ RPC endpoint
// 2. WebSocketProvider - เชื่อมต่อแบบ WebSocket สำหรับ events
// 3. BrowserProvider - ใช้กับ MetaMask ใน browser
// 4. InfuraProvider - เชื่อมต่อผ่าน Infura service
// 5. AlchemyProvider - เชื่อมต่อผ่าน Alchemy service
```

---

## Step 1879: ethers.js - Provider และ Signer

```javascript
const { ethers } = require("ethers");

// ===== Provider =====
// Provider ใช้สำหรับอ่านข้อมูลจาก blockchain (read-only)

// 1. เชื่อมต่อกับ Mainnet ผ่าน default provider
const mainnetProvider = ethers.getDefaultProvider("mainnet");

// 2. เชื่อมต่อผ่าน Infura
const infuraProvider = new ethers.JsonRpcProvider(
  "https://mainnet.infura.io/v3/YOUR_INFURA_KEY"
);

// 3. เชื่อมต่อกับ local Hardhat node
const localProvider = new ethers.JsonRpcProvider("http://localhost:8545");

// 4. เชื่อมต่อกับ Sepolia testnet
const sepoliaProvider = new ethers.JsonRpcProvider(
  "https://sepolia.infura.io/v3/YOUR_INFURA_KEY"
);

// ===== Signer =====
// Signer ใช้สำหรับ sign transactions (สามารถส่ง transaction ได้)

// 1. สร้าง Wallet จาก private key
const privateKey = "0x" + "a".repeat(64); // ตัวอย่างเท่านั้น! ห้ามใช้จริง
const wallet = new ethers.Wallet(privateKey, localProvider);
console.log("Wallet address:", wallet.address);

// 2. สร้าง Wallet จาก mnemonic
const mnemonic = "test test test test test test test test test test test junk";
const mnemonicWallet = ethers.Wallet.fromPhrase(mnemonic);
console.log("Mnemonic wallet:", mnemonicWallet.address);

// 3. สร้าง random wallet
const randomWallet = ethers.Wallet.createRandom();
console.log("Random wallet:", randomWallet.address);
console.log("Private key:", randomWallet.privateKey);
console.log("Mnemonic:", randomWallet.mnemonic.phrase);

// 4. เชื่อม wallet กับ provider
const connectedWallet = randomWallet.connect(localProvider);
```

---

## Step 1880: ethers.js - อ่านข้อมูลจาก Blockchain

```javascript
const { ethers } = require("ethers");

async function readBlockchainData() {
  // เชื่อมต่อกับ Sepolia testnet (ใช้ public RPC)
  const provider = new ethers.JsonRpcProvider(
    "https://rpc.sepolia.org"
  );

  try {
    // 1. ดู block ล่าสุด
    const blockNumber = await provider.getBlockNumber();
    console.log("Block ล่าสุด:", blockNumber);

    // 2. ดูข้อมูล block
    const block = await provider.getBlock(blockNumber);
    console.log("\nข้อมูล Block:");
    console.log("  Hash:", block.hash);
    console.log("  Timestamp:", new Date(block.timestamp * 1000).toLocaleString());
    console.log("  Transactions:", block.transactions.length);
    console.log("  Gas Used:", block.gasUsed.toString());
    console.log("  Gas Limit:", block.gasLimit.toString());
    console.log("  Miner:", block.miner);
    console.log("  Base Fee:", ethers.formatUnits(block.baseFeePerGas, "gwei"), "gwei");

    // 3. ดู balance
    const address = "0x742d35Cc6634C0532925a3b844Bc454e4438f44e";
    const balance = await provider.getBalance(address);
    console.log("\nBalance ของ", address + ":");
    console.log("  Wei:", balance.toString());
    console.log("  ETH:", ethers.formatEther(balance));

    // 4. ดู transaction
    if (block.transactions.length > 0) {
      const txHash = block.transactions[0];
      const tx = await provider.getTransaction(txHash);
      if (tx) {
        console.log("\nTransaction:");
        console.log("  Hash:", tx.hash);
        console.log("  From:", tx.from);
        console.log("  To:", tx.to);
        console.log("  Value:", ethers.formatEther(tx.value), "ETH");
        console.log("  Gas Price:", ethers.formatUnits(tx.gasPrice, "gwei"), "gwei");
        console.log("  Nonce:", tx.nonce);
      }
    }

    // 5. ดู network info
    const network = await provider.getNetwork();
    console.log("\nNetwork:");
    console.log("  Name:", network.name);
    console.log("  Chain ID:", network.chainId.toString());

    // 6. ดู gas price
    const feeData = await provider.getFeeData();
    console.log("\nFee Data:");
    console.log("  Gas Price:", ethers.formatUnits(feeData.gasPrice, "gwei"), "gwei");
    console.log("  Max Fee:", ethers.formatUnits(feeData.maxFeePerGas, "gwei"), "gwei");

  } catch (error) {
    console.error("Error:", error.message);
  }
}

readBlockchainData();
```

---

## Step 1881: ethers.js - ส่ง ETH Transaction

```javascript
const { ethers } = require("ethers");

async function sendTransaction() {
  // ใช้ local Hardhat node สำหรับทดสอบ
  const provider = new ethers.JsonRpcProvider("http://localhost:8545");

  // Hardhat default accounts (มีเงินอยู่แล้ว)
  const privateKey =
    "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80";
  const wallet = new ethers.Wallet(privateKey, provider);

  console.log("Sender:", wallet.address);
  console.log(
    "Balance:",
    ethers.formatEther(await provider.getBalance(wallet.address)),
    "ETH"
  );

  const recipient = "0x70997970C51812dc3A010C7d01b50e0d17dc79C8";

  // ส่ง ETH
  try {
    const tx = await wallet.sendTransaction({
      to: recipient,
      value: ethers.parseEther("0.1"), // 0.1 ETH
      gasLimit: 21000,
    });

    console.log("\nTransaction ส่งแล้ว!");
    console.log("Hash:", tx.hash);

    // รอให้ transaction ยืนยัน
    console.log("รอการยืนยัน...");
    const receipt = await tx.wait();

    console.log("\nTransaction ยืนยันแล้ว!");
    console.log("Block number:", receipt.blockNumber);
    console.log("Gas used:", receipt.gasUsed.toString());
    console.log("Status:", receipt.status === 1 ? "สำเร็จ" : "ล้มเหลว");

    const newBalance = await provider.getBalance(wallet.address);
    console.log("\nBalance หลังส่ง:", ethers.formatEther(newBalance), "ETH");
  } catch (error) {
    console.error("Error:", error.message);
  }
}

sendTransaction();
```

---

## Step 1882: ethers.js - Interacting with Smart Contracts

```javascript
const { ethers } = require("ethers");

// ABI ของ ERC-20 Token (ส่วนที่ใช้บ่อย)
const ERC20_ABI = [
  "function name() view returns (string)",
  "function symbol() view returns (string)",
  "function decimals() view returns (uint8)",
  "function totalSupply() view returns (uint256)",
  "function balanceOf(address owner) view returns (uint256)",
  "function transfer(address to, uint256 amount) returns (bool)",
  "function approve(address spender, uint256 amount) returns (bool)",
  "function allowance(address owner, address spender) view returns (uint256)",
  "event Transfer(address indexed from, address indexed to, uint256 value)",
  "event Approval(address indexed owner, address indexed spender, uint256 value)",
];

async function interactWithERC20() {
  const provider = new ethers.JsonRpcProvider(
    "https://mainnet.infura.io/v3/YOUR_INFURA_KEY"
  );

  // USDC contract address บน Mainnet
  const USDC_ADDRESS = "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48";

  // สร้าง contract instance (read-only)
  const usdcContract = new ethers.Contract(USDC_ADDRESS, ERC20_ABI, provider);

  // อ่านข้อมูล token
  const name = await usdcContract.name();
  const symbol = await usdcContract.symbol();
  const decimals = await usdcContract.decimals();
  const totalSupply = await usdcContract.totalSupply();

  console.log("Token Info:");
  console.log("  Name:", name);
  console.log("  Symbol:", symbol);
  console.log("  Decimals:", decimals);
  console.log(
    "  Total Supply:",
    ethers.formatUnits(totalSupply, decimals),
    symbol
  );

  // ดู balance ของ address
  const address = "0x47ac0Fb4F2D84898e4D9E7b4DaB3C24507a6D503";
  const balance = await usdcContract.balanceOf(address);
  console.log(`\nBalance ของ ${address}:`);
  console.log(`  ${ethers.formatUnits(balance, decimals)} ${symbol}`);
}

// ตัวอย่างการ send transaction ไปยัง contract
async function transferToken(contractAddress, abi, toAddress, amount) {
  const provider = new ethers.JsonRpcProvider("http://localhost:8545");
  const privateKey =
    "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80";
  const signer = new ethers.Wallet(privateKey, provider);

  // สร้าง contract instance พร้อม signer (สามารถส่ง tx ได้)
  const contract = new ethers.Contract(contractAddress, abi, signer);

  // เรียก transfer function
  const tx = await contract.transfer(
    toAddress,
    ethers.parseUnits(amount.toString(), 18)
  );
  console.log("Transaction hash:", tx.hash);

  const receipt = await tx.wait();
  console.log("Block:", receipt.blockNumber);
  console.log("Status:", receipt.status === 1 ? "สำเร็จ" : "ล้มเหลว");
}

interactWithERC20().catch(console.error);
```

---

## Step 1883: ethers.js - Listening to Events

```javascript
const { ethers } = require("ethers");

const ERC20_ABI = [
  "event Transfer(address indexed from, address indexed to, uint256 value)",
  "event Approval(address indexed owner, address indexed spender, uint256 value)",
  "function decimals() view returns (uint8)",
  "function symbol() view returns (string)",
];

async function listenToEvents() {
  const provider = new ethers.WebSocketProvider(
    "wss://mainnet.infura.io/ws/v3/YOUR_INFURA_KEY"
  );

  const USDC_ADDRESS = "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48";
  const contract = new ethers.Contract(USDC_ADDRESS, ERC20_ABI, provider);

  const decimals = await contract.decimals();
  const symbol = await contract.symbol();

  console.log(`กำลังฟัง Transfer events ของ ${symbol}...`);

  // ฟัง event แบบ real-time
  contract.on("Transfer", (from, to, value, event) => {
    const amount = ethers.formatUnits(value, decimals);
    console.log(`\n💸 Transfer event:`);
    console.log(`  From: ${from}`);
    console.log(`  To: ${to}`);
    console.log(`  Amount: ${amount} ${symbol}`);
    console.log(`  Tx Hash: ${event.log.transactionHash}`);
  });

  // ฟัง event เฉพาะ address ที่กำหนด (filter)
  const targetAddress = "0x742d35Cc6634C0532925a3b844Bc454e4438f44e";
  const filterFrom = contract.filters.Transfer(targetAddress);
  const filterTo = contract.filters.Transfer(null, targetAddress);

  contract.on(filterFrom, (from, to, value) => {
    console.log(`\n📤 ${targetAddress} ส่ง ${ethers.formatUnits(value, decimals)} ${symbol}`);
  });

  contract.on(filterTo, (from, to, value) => {
    console.log(`\n📥 ${targetAddress} รับ ${ethers.formatUnits(value, decimals)} ${symbol}`);
  });
}

// ดึง events ในอดีต
async function getPastEvents() {
  const provider = new ethers.JsonRpcProvider(
    "https://mainnet.infura.io/v3/YOUR_INFURA_KEY"
  );
  const USDC_ADDRESS = "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48";
  const contract = new ethers.Contract(USDC_ADDRESS, ERC20_ABI, provider);
  const decimals = await contract.decimals();

  const currentBlock = await provider.getBlockNumber();
  const fromBlock = currentBlock - 100; // 100 blocks ที่ผ่านมา

  console.log(`ดึง Transfer events ตั้งแต่ block ${fromBlock} ถึง ${currentBlock}`);

  const events = await contract.queryFilter(
    contract.filters.Transfer(),
    fromBlock,
    currentBlock
  );

  console.log(`พบ ${events.length} events`);
  events.slice(0, 5).forEach(event => {
    console.log(`  ${event.args.from} → ${event.args.to}: ${ethers.formatUnits(event.args.value, decimals)} USDC`);
  });
}

// listenToEvents();
getPastEvents().catch(console.error);
```

---

## Step 1884: ethers.js - ENS Resolution

```javascript
const { ethers } = require("ethers");

async function ensExamples() {
  const provider = ethers.getDefaultProvider("mainnet");

  // 1. แปลง ENS name เป็น address
  const address = await provider.resolveName("vitalik.eth");
  console.log("vitalik.eth →", address);

  // 2. แปลง address เป็น ENS name (Reverse lookup)
  if (address) {
    const name = await provider.lookupAddress(address);
    console.log(address, "→", name);
  }

  // 3. ดู ENS avatar
  const resolver = await provider.getResolver("vitalik.eth");
  if (resolver) {
    const avatar = await resolver.getAvatar();
    console.log("Avatar:", avatar);

    const twitter = await resolver.getText("com.twitter");
    console.log("Twitter:", twitter);

    const github = await resolver.getText("com.github");
    console.log("GitHub:", github);
  }

  // 4. ตรวจสอบว่า name ถูก resolve หรือเปล่า
  async function safeResolve(name) {
    try {
      const addr = await provider.resolveName(name);
      if (!addr) {
        console.log(`${name} ไม่ได้ register หรือไม่มี address`);
        return null;
      }
      return addr;
    } catch (error) {
      console.error(`Error resolving ${name}:`, error.message);
      return null;
    }
  }

  await safeResolve("nick.eth");
  await safeResolve("nonexistent12345.eth");
}

ensExamples().catch(console.error);
```

---

## Step 1885: Web3.js Overview และเปรียบเทียบกับ ethers.js

```javascript
// ===== เปรียบเทียบ Web3.js vs ethers.js =====

// Web3.js (เวอร์ชัน 1.x)
const Web3 = require("web3");
const web3 = new Web3("https://mainnet.infura.io/v3/YOUR_KEY");

// ดู balance (Web3.js)
async function web3Example() {
  const balance = await web3.eth.getBalance("0xAddress...");
  console.log("Balance (Wei):", balance);
  console.log("Balance (ETH):", web3.utils.fromWei(balance, "ether"));

  const block = await web3.eth.getBlock("latest");
  console.log("Block number:", block.number);

  // สร้าง account
  const account = web3.eth.accounts.create();
  console.log("Address:", account.address);
  console.log("Private Key:", account.privateKey);
}

// ethers.js (เวอร์ชัน 6.x)
const { ethers } = require("ethers");

async function ethersExample() {
  const provider = new ethers.JsonRpcProvider(
    "https://mainnet.infura.io/v3/YOUR_KEY"
  );
  const balance = await provider.getBalance("0xAddress...");
  console.log("Balance (ETH):", ethers.formatEther(balance));

  const block = await provider.getBlock("latest");
  console.log("Block number:", block.number);

  // สร้าง wallet
  const wallet = ethers.Wallet.createRandom();
  console.log("Address:", wallet.address);
  console.log("Private Key:", wallet.privateKey);
}

/*
===== ตาราง เปรียบเทียบ =====

Feature              | Web3.js          | ethers.js
---------------------|------------------|------------------
Bundle size          | ใหญ่ (~590KB)    | เล็กกว่า (~116KB)
TypeScript support   | ปานกลาง          | ดีมาก
API style            | Callback/Promise  | Promise/async-await
Wallet management    | web3.eth.accounts | Wallet class
ENS support          | จำกัด            | Built-in สมบูรณ์
Event handling       | .events.xxx()    | contract.on()
Units                | fromWei/toWei    | formatEther/parseEther
Big numbers          | BN.js            | BigInt (native)
Maintainability      | ChainSafe        | Richard Moore (rigorously maintained)
Popularity           | เก่ากว่า         | กำลังได้รับความนิยม

แนะนำ: ใช้ ethers.js สำหรับโปรเจกต์ใหม่
*/
```

---

## Step 1886: MetaMask Integration

```javascript
// ===== MetaMask Integration =====
// ใช้ใน Browser (HTML + JavaScript)

// 1. ตรวจสอบว่ามี MetaMask หรือไม่
function checkMetaMask() {
  if (typeof window.ethereum !== "undefined") {
    console.log("MetaMask พบแล้ว!");
    return true;
  } else {
    console.log("ไม่พบ MetaMask กรุณาติดตั้ง MetaMask ก่อน");
    window.open("https://metamask.io/download/", "_blank");
    return false;
  }
}

// 2. ขอเชื่อมต่อกับ MetaMask
async function connectMetaMask() {
  if (!checkMetaMask()) return null;

  try {
    // ขอ accounts (จะแสดง MetaMask popup)
    const accounts = await window.ethereum.request({
      method: "eth_requestAccounts",
    });

    console.log("Accounts:", accounts);
    console.log("Connected account:", accounts[0]);
    return accounts[0];
  } catch (error) {
    if (error.code === 4001) {
      console.log("User ปฏิเสธการเชื่อมต่อ");
    } else {
      console.error("Error:", error);
    }
    return null;
  }
}

// 3. ดู network ปัจจุบัน
async function getNetwork() {
  const chainId = await window.ethereum.request({ method: "eth_chainId" });
  console.log("Chain ID (hex):", chainId);
  console.log("Chain ID (decimal):", parseInt(chainId, 16));

  const networks = {
    "0x1": "Ethereum Mainnet",
    "0xaa36a7": "Sepolia Testnet",
    "0x89": "Polygon Mainnet",
    "0x13881": "Polygon Mumbai Testnet",
    "0xa4b1": "Arbitrum One",
    "0xa": "Optimism",
    "0xa86a": "Avalanche",
    "0x38": "BNB Smart Chain",
  };

  return networks[chainId] || `Unknown Network (${chainId})`;
}

// 4. ส่ง transaction ผ่าน MetaMask
async function sendETH(to, amountInEth) {
  const accounts = await window.ethereum.request({
    method: "eth_accounts",
  });

  if (accounts.length === 0) {
    await connectMetaMask();
    return;
  }

  const from = accounts[0];
  const amountHex =
    "0x" +
    BigInt(Math.floor(amountInEth * 1e18)).toString(16);

  try {
    const txHash = await window.ethereum.request({
      method: "eth_sendTransaction",
      params: [
        {
          from,
          to,
          value: amountHex,
          gas: "0x5208", // 21000 in hex
        },
      ],
    });

    console.log("Transaction hash:", txHash);
    return txHash;
  } catch (error) {
    if (error.code === 4001) {
      console.log("User ยกเลิก transaction");
    } else {
      console.error("Error:", error);
    }
  }
}

// 5. ฟัง account changes
window.ethereum.on("accountsChanged", (accounts) => {
  if (accounts.length === 0) {
    console.log("MetaMask ถูก disconnect");
  } else {
    console.log("Account เปลี่ยนเป็น:", accounts[0]);
    // อัปเดต UI ใหม่
    updateUI(accounts[0]);
  }
});

// 6. ฟัง network changes
window.ethereum.on("chainChanged", (chainId) => {
  console.log("Network เปลี่ยนเป็น:", chainId);
  // รีโหลดหน้าเพื่อความปลอดภัย
  window.location.reload();
});

// 7. เพิ่ม network ใหม่ (เช่น Polygon)
async function addPolygonNetwork() {
  try {
    await window.ethereum.request({
      method: "wallet_addEthereumChain",
      params: [
        {
          chainId: "0x89",
          chainName: "Polygon Mainnet",
          nativeCurrency: {
            name: "MATIC",
            symbol: "MATIC",
            decimals: 18,
          },
          rpcUrls: ["https://polygon-rpc.com/"],
          blockExplorerUrls: ["https://polygonscan.com/"],
        },
      ],
    });
    console.log("เพิ่ม Polygon network สำเร็จ");
  } catch (error) {
    console.error("Error:", error);
  }
}

function updateUI(account) {
  // placeholder
  console.log("Updated UI for account:", account);
}
```

---

## Step 1887: สร้าง Simple Token Smart Contract

```javascript
// ===== Simple ERC-20 Token Contract (Solidity) =====
/*
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract SimpleToken {
    string public name = "Simple Token";
    string public symbol = "SIT";
    uint8 public decimals = 18;
    uint256 public totalSupply;
    
    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;
    
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
    
    constructor(uint256 _initialSupply) {
        totalSupply = _initialSupply * 10**decimals;
        balanceOf[msg.sender] = totalSupply;
        emit Transfer(address(0), msg.sender, totalSupply);
    }
    
    function transfer(address to, uint256 amount) external returns (bool) {
        require(balanceOf[msg.sender] >= amount, "Insufficient balance");
        require(to != address(0), "Cannot transfer to zero address");
        
        balanceOf[msg.sender] -= amount;
        balanceOf[to] += amount;
        emit Transfer(msg.sender, to, amount);
        return true;
    }
    
    function approve(address spender, uint256 amount) external returns (bool) {
        allowance[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }
    
    function transferFrom(address from, address to, uint256 amount) external returns (bool) {
        require(balanceOf[from] >= amount, "Insufficient balance");
        require(allowance[from][msg.sender] >= amount, "Insufficient allowance");
        require(to != address(0), "Cannot transfer to zero address");
        
        allowance[from][msg.sender] -= amount;
        balanceOf[from] -= amount;
        balanceOf[to] += amount;
        emit Transfer(from, to, amount);
        return true;
    }
}
*/

// ABI ของ SimpleToken สำหรับใช้กับ ethers.js
const SIMPLE_TOKEN_ABI = [
  "constructor(uint256 _initialSupply)",
  "function name() view returns (string)",
  "function symbol() view returns (string)",
  "function decimals() view returns (uint8)",
  "function totalSupply() view returns (uint256)",
  "function balanceOf(address) view returns (uint256)",
  "function allowance(address owner, address spender) view returns (uint256)",
  "function transfer(address to, uint256 amount) returns (bool)",
  "function approve(address spender, uint256 amount) returns (bool)",
  "function transferFrom(address from, address to, uint256 amount) returns (bool)",
  "event Transfer(address indexed from, address indexed to, uint256 value)",
  "event Approval(address indexed owner, address indexed spender, uint256 value)",
];

// Deploy contract ด้วย ethers.js
const { ethers } = require("ethers");

async function deploySimpleToken() {
  const provider = new ethers.JsonRpcProvider("http://localhost:8545");
  const signer = await provider.getSigner();

  // Bytecode ของ contract (จะได้จาก Solidity compiler)
  // const bytecode = "0x608060405234801561001057600080fd5b50...";
  // const factory = new ethers.ContractFactory(SIMPLE_TOKEN_ABI, bytecode, signer);
  // const contract = await factory.deploy(1000000); // 1,000,000 tokens
  // await contract.waitForDeployment();
  // console.log("Contract deployed at:", await contract.getAddress());

  console.log("Deploy contract เสร็จแล้ว!");
  console.log("ใช้ Hardhat หรือ Remix IDE เพื่อ compile และ deploy จริง");
}
```

---

## Step 1888: สร้าง Complete DApp Frontend

```html
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple Token DApp</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body {
            font-family: 'Segoe UI', sans-serif;
            background: #1a1a2e;
            color: #eee;
            min-height: 100vh;
            padding: 20px;
        }
        .container { max-width: 600px; margin: 0 auto; }
        h1 { text-align: center; color: #e94560; margin-bottom: 30px; font-size: 2rem; }
        .card {
            background: #16213e;
            border-radius: 12px;
            padding: 24px;
            margin-bottom: 20px;
            border: 1px solid #0f3460;
        }
        .card h2 { color: #e94560; margin-bottom: 16px; }
        .info-row { display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px solid #0f3460; }
        .label { color: #888; }
        .value { color: #fff; font-weight: 500; word-break: break-all; }
        button {
            background: #e94560;
            color: white;
            border: none;
            padding: 12px 24px;
            border-radius: 8px;
            cursor: pointer;
            font-size: 16px;
            width: 100%;
            margin-top: 10px;
            transition: opacity 0.2s;
        }
        button:hover { opacity: 0.8; }
        button:disabled { background: #555; cursor: not-allowed; }
        input {
            width: 100%;
            padding: 12px;
            background: #0f3460;
            border: 1px solid #e94560;
            border-radius: 8px;
            color: white;
            font-size: 16px;
            margin-bottom: 10px;
        }
        .status { padding: 12px; border-radius: 8px; margin-top: 10px; text-align: center; }
        .status.success { background: #1a472a; color: #4ade80; }
        .status.error { background: #7f1d1d; color: #f87171; }
        .status.pending { background: #1c1c3a; color: #60a5fa; }
        .badge {
            display: inline-block;
            padding: 4px 12px;
            border-radius: 20px;
            font-size: 12px;
            background: #0f3460;
        }
        .connected { background: #1a472a; color: #4ade80; }
        .disconnected { background: #7f1d1d; color: #f87171; }
    </style>
</head>
<body>
    <div class="container">
        <h1>🔗 Simple Token DApp</h1>

        <!-- Wallet Connection -->
        <div class="card">
            <h2>กระเป๋าสตางค์</h2>
            <div class="info-row">
                <span class="label">สถานะ</span>
                <span class="badge disconnected" id="connectionStatus">ไม่ได้เชื่อมต่อ</span>
            </div>
            <div class="info-row">
                <span class="label">Address</span>
                <span class="value" id="walletAddress">-</span>
            </div>
            <div class="info-row">
                <span class="label">Network</span>
                <span class="value" id="networkName">-</span>
            </div>
            <div class="info-row">
                <span class="label">ETH Balance</span>
                <span class="value" id="ethBalance">-</span>
            </div>
            <button id="connectBtn" onclick="connectWallet()">เชื่อมต่อ MetaMask</button>
        </div>

        <!-- Token Info -->
        <div class="card">
            <h2>ข้อมูล Token</h2>
            <div class="info-row">
                <span class="label">ชื่อ Token</span>
                <span class="value" id="tokenName">-</span>
            </div>
            <div class="info-row">
                <span class="label">Symbol</span>
                <span class="value" id="tokenSymbol">-</span>
            </div>
            <div class="info-row">
                <span class="label">Total Supply</span>
                <span class="value" id="tokenSupply">-</span>
            </div>
            <div class="info-row">
                <span class="label">Balance ของคุณ</span>
                <span class="value" id="tokenBalance">-</span>
            </div>
        </div>

        <!-- Transfer -->
        <div class="card">
            <h2>โอน Token</h2>
            <input type="text" id="toAddress" placeholder="Address ปลายทาง (0x...)" />
            <input type="number" id="transferAmount" placeholder="จำนวน Token" step="0.01" />
            <button id="transferBtn" onclick="transferToken()">โอน Token</button>
            <div id="transferStatus" style="display:none" class="status"></div>
        </div>
    </div>

    <!-- โหลด ethers.js จาก CDN -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/ethers.umd.min.js"></script>
    <script>
        // ===== Configuration =====
        const TOKEN_ADDRESS = "0x..."; // ใส่ contract address ที่ deploy แล้ว
        const TOKEN_ABI = [
            "function name() view returns (string)",
            "function symbol() view returns (string)",
            "function decimals() view returns (uint8)",
            "function totalSupply() view returns (uint256)",
            "function balanceOf(address) view returns (uint256)",
            "function transfer(address to, uint256 amount) returns (bool)",
            "event Transfer(address indexed from, address indexed to, uint256 value)",
        ];

        let provider, signer, tokenContract, userAddress;

        // ===== Connect Wallet =====
        async function connectWallet() {
            if (typeof window.ethereum === "undefined") {
                showStatus("transferStatus", "กรุณาติดตั้ง MetaMask ก่อน", "error");
                return;
            }

            try {
                provider = new ethers.BrowserProvider(window.ethereum);
                signer = await provider.getSigner();
                userAddress = await signer.getAddress();

                // อัปเดต UI
                document.getElementById("connectionStatus").textContent = "เชื่อมต่อแล้ว";
                document.getElementById("connectionStatus").className = "badge connected";
                document.getElementById("walletAddress").textContent =
                    userAddress.slice(0, 6) + "..." + userAddress.slice(-4);
                document.getElementById("connectBtn").textContent = "เชื่อมต่อแล้ว ✓";
                document.getElementById("connectBtn").disabled = true;

                // ดู network
                const network = await provider.getNetwork();
                const chainId = Number(network.chainId);
                const networks = {
                    1: "Ethereum Mainnet",
                    11155111: "Sepolia Testnet",
                    137: "Polygon",
                    31337: "Hardhat Local",
                };
                document.getElementById("networkName").textContent =
                    networks[chainId] || `Chain ${chainId}`;

                // ดู ETH balance
                const ethBalance = await provider.getBalance(userAddress);
                document.getElementById("ethBalance").textContent =
                    parseFloat(ethers.formatEther(ethBalance)).toFixed(4) + " ETH";

                // โหลด token info
                await loadTokenInfo();

            } catch (error) {
                if (error.code === 4001) {
                    showStatus("transferStatus", "คุณปฏิเสธการเชื่อมต่อ", "error");
                } else {
                    showStatus("transferStatus", "Error: " + error.message, "error");
                }
            }
        }

        // ===== Load Token Info =====
        async function loadTokenInfo() {
            try {
                tokenContract = new ethers.Contract(TOKEN_ADDRESS, TOKEN_ABI, provider);

                const [name, symbol, decimals, totalSupply, balance] = await Promise.all([
                    tokenContract.name(),
                    tokenContract.symbol(),
                    tokenContract.decimals(),
                    tokenContract.totalSupply(),
                    tokenContract.balanceOf(userAddress),
                ]);

                document.getElementById("tokenName").textContent = name;
                document.getElementById("tokenSymbol").textContent = symbol;
                document.getElementById("tokenSupply").textContent =
                    parseFloat(ethers.formatUnits(totalSupply, decimals)).toLocaleString() + " " + symbol;
                document.getElementById("tokenBalance").textContent =
                    parseFloat(ethers.formatUnits(balance, decimals)).toFixed(4) + " " + symbol;

                // ฟัง Transfer events
                tokenContract.on("Transfer", (from, to, value, event) => {
                    if (from === userAddress || to === userAddress) {
                        loadTokenInfo(); // รีโหลด balance
                    }
                });

            } catch (error) {
                console.error("Error loading token:", error);
                // ถ้า contract address ไม่ถูกต้อง
                document.getElementById("tokenName").textContent = "ไม่พบ Token";
            }
        }

        // ===== Transfer Token =====
        async function transferToken() {
            if (!signer) {
                showStatus("transferStatus", "กรุณาเชื่อมต่อ wallet ก่อน", "error");
                return;
            }

            const to = document.getElementById("toAddress").value.trim();
            const amount = document.getElementById("transferAmount").value;

            if (!ethers.isAddress(to)) {
                showStatus("transferStatus", "Address ไม่ถูกต้อง", "error");
                return;
            }

            if (!amount || parseFloat(amount) <= 0) {
                showStatus("transferStatus", "จำนวนไม่ถูกต้อง", "error");
                return;
            }

            try {
                showStatus("transferStatus", "กำลังส่ง transaction...", "pending");
                document.getElementById("transferBtn").disabled = true;

                const contractWithSigner = tokenContract.connect(signer);
                const decimals = await tokenContract.decimals();
                const parsedAmount = ethers.parseUnits(amount, decimals);

                const tx = await contractWithSigner.transfer(to, parsedAmount);
                showStatus("transferStatus", `รอการยืนยัน... TX: ${tx.hash.slice(0, 20)}...`, "pending");

                await tx.wait();
                showStatus("transferStatus", "โอนสำเร็จ! ✓", "success");

                // รีเซ็ต form
                document.getElementById("toAddress").value = "";
                document.getElementById("transferAmount").value = "";

                // รีโหลด balance
                await loadTokenInfo();

            } catch (error) {
                if (error.code === 4001) {
                    showStatus("transferStatus", "คุณยกเลิก transaction", "error");
                } else {
                    showStatus("transferStatus", "Error: " + error.message, "error");
                }
            } finally {
                document.getElementById("transferBtn").disabled = false;
            }
        }

        // ===== Helper =====
        function showStatus(elementId, message, type) {
            const el = document.getElementById(elementId);
            el.textContent = message;
            el.className = `status ${type}`;
            el.style.display = "block";
        }

        // ฟัง MetaMask changes
        if (window.ethereum) {
            window.ethereum.on("accountsChanged", (accounts) => {
                window.location.reload();
            });
            window.ethereum.on("chainChanged", () => {
                window.location.reload();
            });
        }
    </script>
</body>
</html>
```

---

## Step 1889: NFT Concepts และ ERC-721

```javascript
// ===== NFT (Non-Fungible Token) =====
// NFT เป็น token ที่ไม่สามารถแทนกันได้ แต่ละชิ้นมีความเป็นเอกลักษณ์

// Solidity ERC-721 Contract (ตัวอย่างย่อ)
/*
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract SimpleNFT is ERC721, ERC721URIStorage, Ownable {
    uint256 private _nextTokenId;
    uint256 public mintPrice = 0.01 ether;
    
    constructor() ERC721("Simple NFT", "SNFT") Ownable(msg.sender) {}
    
    function mint(address to, string memory tokenURI) external payable {
        require(msg.value >= mintPrice, "Insufficient payment");
        
        uint256 tokenId = _nextTokenId++;
        _safeMint(to, tokenId);
        _setTokenURI(tokenId, tokenURI);
    }
    
    function tokenURI(uint256 tokenId) public view override(ERC721, ERC721URIStorage)
        returns (string memory) {
        return super.tokenURI(tokenId);
    }
    
    function supportsInterface(bytes4 interfaceId) public view override(ERC721, ERC721URIStorage)
        returns (bool) {
        return super.supportsInterface(interfaceId);
    }
    
    function withdraw() external onlyOwner {
        payable(owner()).transfer(address(this).balance);
    }
}
*/

// ABI สำหรับ interact กับ NFT
const NFT_ABI = [
  "function name() view returns (string)",
  "function symbol() view returns (string)",
  "function totalSupply() view returns (uint256)",
  "function tokenURI(uint256 tokenId) view returns (string)",
  "function ownerOf(uint256 tokenId) view returns (address)",
  "function balanceOf(address owner) view returns (uint256)",
  "function mint(address to, string tokenURI) payable",
  "function transferFrom(address from, address to, uint256 tokenId)",
  "function approve(address to, uint256 tokenId)",
  "event Transfer(address indexed from, address indexed to, uint256 indexed tokenId)",
  "event Approval(address indexed owner, address indexed approved, uint256 indexed tokenId)",
];

const { ethers } = require("ethers");

// ตัวอย่าง NFT Metadata (JSON ที่เก็บบน IPFS)
const nftMetadata = {
  name: "Cool NFT #1",
  description: "NFT สวยๆ ที่สร้างด้วย JavaScript",
  image: "ipfs://QmXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/image.png",
  attributes: [
    { trait_type: "Background", value: "Blue" },
    { trait_type: "Eyes", value: "Golden" },
    { trait_type: "Mouth", value: "Smile" },
    { trait_type: "Rarity", value: "Rare" },
    { trait_type: "Level", value: 5, display_type: "number" },
  ],
  external_url: "https://mynft.example.com/1",
  animation_url: "ipfs://QmXXXXXXXXXXXXXXXX/animation.mp4",
};

console.log("NFT Metadata:", JSON.stringify(nftMetadata, null, 2));

// ===== Mint NFT =====
async function mintNFT(nftContractAddress, toAddress, metadataURI) {
  const provider = new ethers.JsonRpcProvider("http://localhost:8545");
  const signer = await provider.getSigner();

  const nftContract = new ethers.Contract(nftContractAddress, NFT_ABI, signer);

  // mint price (0.01 ETH)
  const mintPrice = ethers.parseEther("0.01");

  const tx = await nftContract.mint(toAddress, metadataURI, { value: mintPrice });
  console.log("Minting NFT...", tx.hash);

  const receipt = await tx.wait();
  console.log("NFT minted! Block:", receipt.blockNumber);

  // ดึง token ID จาก Transfer event
  const transferEvent = receipt.logs.find(
    log => nftContract.interface.parseLog(log)?.name === "Transfer"
  );
  if (transferEvent) {
    const parsed = nftContract.interface.parseLog(transferEvent);
    console.log("Token ID:", parsed.args.tokenId.toString());
  }
}

// ===== ดู NFT ของ address =====
async function getNFTsOfAddress(nftContractAddress, ownerAddress) {
  const provider = new ethers.JsonRpcProvider("http://localhost:8545");
  const nftContract = new ethers.Contract(nftContractAddress, NFT_ABI, provider);

  const balance = await nftContract.balanceOf(ownerAddress);
  console.log(`${ownerAddress} มี ${balance} NFTs`);

  // ดู Transfer events เพื่อหา tokenIds
  const filter = nftContract.filters.Transfer(null, ownerAddress);
  const events = await nftContract.queryFilter(filter, 0, "latest");

  for (const event of events) {
    const tokenId = event.args.tokenId;
    const currentOwner = await nftContract.ownerOf(tokenId);

    if (currentOwner.toLowerCase() === ownerAddress.toLowerCase()) {
      const uri = await nftContract.tokenURI(tokenId);
      console.log(`Token #${tokenId}: ${uri}`);
    }
  }
}
```

---

## Step 1890: DeFi Concepts, IPFS และ สรุป

### DeFi (Decentralized Finance) Concepts

```javascript
// ===== DEX (Decentralized Exchange) =====
// Uniswap คือ DEX ที่ใช้ Automated Market Maker (AMM)
// สูตร: x * y = k (constant product formula)

class LiquidityPool {
  constructor(tokenAAmount, tokenBAmount) {
    this.reserveA = tokenAAmount;
    this.reserveB = tokenBAmount;
    this.k = tokenAAmount * tokenBAmount; // constant
    this.totalShares = Math.sqrt(tokenAAmount * tokenBAmount);
    this.shares = {};
  }

  // คำนวณราคาปัจจุบัน
  getPrice(tokenIn) {
    if (tokenIn === "A") return this.reserveB / this.reserveA;
    return this.reserveA / this.reserveB;
  }

  // Swap Token A → Token B
  swapAForB(amountIn) {
    const amountInWithFee = amountIn * 0.997; // 0.3% fee
    const amountOut =
      (this.reserveB * amountInWithFee) / (this.reserveA + amountInWithFee);

    this.reserveA += amountIn;
    this.reserveB -= amountOut;

    console.log(`Swap ${amountIn} A → ${amountOut.toFixed(4)} B`);
    console.log(`New price: 1 A = ${this.getPrice("A").toFixed(4)} B`);

    return amountOut;
  }

  // เพิ่ม liquidity
  addLiquidity(amountA, amountB, provider) {
    const shares = Math.sqrt(amountA * amountB);
    this.reserveA += amountA;
    this.reserveB += amountB;
    this.totalShares += shares;
    this.shares[provider] = (this.shares[provider] || 0) + shares;

    console.log(`เพิ่ม Liquidity: ${amountA} A + ${amountB} B = ${shares.toFixed(4)} shares`);
    return shares;
  }

  // ถอน liquidity
  removeLiquidity(shares, provider) {
    if ((this.shares[provider] || 0) < shares) {
      throw new Error("Insufficient shares");
    }

    const amountA = (shares / this.totalShares) * this.reserveA;
    const amountB = (shares / this.totalShares) * this.reserveB;

    this.reserveA -= amountA;
    this.reserveB -= amountB;
    this.totalShares -= shares;
    this.shares[provider] -= shares;

    console.log(`ถอน Liquidity: ${amountA.toFixed(4)} A + ${amountB.toFixed(4)} B`);
    return { amountA, amountB };
  }

  getPoolInfo() {
    return {
      reserveA: this.reserveA,
      reserveB: this.reserveB,
      price_A_in_B: this.getPrice("A").toFixed(4),
      price_B_in_A: this.getPrice("B").toFixed(4),
      totalShares: this.totalShares.toFixed(4),
    };
  }
}

// ทดสอบ DEX
const pool = new LiquidityPool(1000, 2000); // 1000 ETH : 2000 USDC
console.log("Pool เริ่มต้น:", pool.getPoolInfo());

pool.addLiquidity(100, 200, "Alice");
pool.swapAForB(10);
console.log("หลัง Swap:", pool.getPoolInfo());

// ===== Yield Farming (แนวคิด) =====
class YieldFarm {
  constructor(rewardPerBlock, totalRewardBlocks) {
    this.rewardPerBlock = rewardPerBlock;
    this.totalRewardBlocks = totalRewardBlocks;
    this.startBlock = 0;
    this.totalStaked = 0;
    this.stakes = {};
    this.lastUpdateBlock = 0;
    this.accRewardPerShare = 0;
    this.pendingRewards = {};
  }

  stake(user, amount, currentBlock) {
    this.updatePool(currentBlock);

    if (this.stakes[user]) {
      const pending = (this.stakes[user] * this.accRewardPerShare) / 1e12 - (this.pendingRewards[user] || 0);
      if (pending > 0) {
        console.log(`${user} รับ reward: ${pending.toFixed(4)} tokens`);
      }
    }

    this.totalStaked += amount;
    this.stakes[user] = (this.stakes[user] || 0) + amount;
    this.pendingRewards[user] = (this.stakes[user] * this.accRewardPerShare) / 1e12;

    console.log(`${user} stake ${amount} tokens (total: ${this.stakes[user]})`);
  }

  updatePool(currentBlock) {
    if (currentBlock <= this.lastUpdateBlock) return;
    if (this.totalStaked === 0) {
      this.lastUpdateBlock = currentBlock;
      return;
    }

    const blocks = Math.min(currentBlock, this.startBlock + this.totalRewardBlocks) - this.lastUpdateBlock;
    const reward = blocks * this.rewardPerBlock;
    this.accRewardPerShare += (reward * 1e12) / this.totalStaked;
    this.lastUpdateBlock = currentBlock;
  }

  getPendingReward(user, currentBlock) {
    const tempAccReward =
      this.accRewardPerShare +
      ((currentBlock - this.lastUpdateBlock) * this.rewardPerBlock * 1e12) / (this.totalStaked || 1);

    return ((this.stakes[user] || 0) * tempAccReward) / 1e12 - (this.pendingRewards[user] || 0);
  }
}

const farm = new YieldFarm(10, 1000); // 10 tokens/block, 1000 blocks
farm.startBlock = 0;
farm.lastUpdateBlock = 0;

farm.stake("Alice", 100, 0);
farm.stake("Bob", 50, 10);

const aliceReward = farm.getPendingReward("Alice", 100);
console.log(`Alice pending reward ที่ block 100: ${aliceReward.toFixed(4)}`);
```

---

### IPFS สำหรับ Decentralized Storage

```javascript
// ===== IPFS (InterPlanetary File System) =====
// IPFS ใช้ content-addressed storage แทน location-addressed

// ===== ใช้ js-ipfs หรือ @web3-storage/w3up-client =====
// npm install ipfs-http-client

const { create } = require("ipfs-http-client");

async function ipfsExamples() {
  // เชื่อมต่อกับ IPFS node ท้องถิ่น
  const ipfs = create({ url: "http://localhost:5001" });

  // 1. เพิ่มไฟล์ text
  const { cid } = await ipfs.add("Hello from IPFS!");
  console.log("CID:", cid.toString());
  // CID ตัวอย่าง: QmXXXXXXXXXXXXXXXXXXXXXXXXXXXX

  // 2. เพิ่มไฟล์ JSON (เช่น NFT metadata)
  const metadata = {
    name: "My Cool NFT",
    description: "A unique digital artwork",
    image: "ipfs://QmImageCIDHere",
    attributes: [
      { trait_type: "Color", value: "Blue" },
      { trait_type: "Rarity", value: "Rare" },
    ],
  };

  const metadataJson = JSON.stringify(metadata);
  const { cid: metadataCid } = await ipfs.add(metadataJson);
  const metadataUri = `ipfs://${metadataCid}`;
  console.log("Metadata URI:", metadataUri);

  // 3. อ่านไฟล์จาก IPFS
  const chunks = [];
  for await (const chunk of ipfs.cat(cid)) {
    chunks.push(chunk);
  }
  const content = Buffer.concat(chunks).toString();
  console.log("เนื้อหา:", content);

  // 4. เพิ่มโฟลเดอร์พร้อมหลายไฟล์
  const files = [
    { path: "images/nft1.png", content: Buffer.from("fake png data") },
    { path: "metadata/1.json", content: Buffer.from(JSON.stringify(metadata)) },
  ];

  const results = [];
  for await (const result of ipfs.addAll(files, { wrapWithDirectory: true })) {
    results.push(result);
  }
  const folderCid = results[results.length - 1].cid;
  console.log("Folder CID:", folderCid.toString());
  console.log("Base URI:", `ipfs://${folderCid}/metadata/`);
}

// ===== ใช้ Web3.Storage =====
// npm install @web3-storage/w3up-client

async function uploadToWeb3Storage(file, authToken) {
  // ใหม่: ใช้ @web3-storage/w3up-client
  // เก่า: web3.storage client
  console.log("Web3.Storage upload:");
  console.log("1. สมัครที่ web3.storage");
  console.log("2. สร้าง API token");
  console.log("3. ใช้ client.put([file])");
  console.log("4. ได้รับ CID กลับมา");
  console.log("5. ใช้ https://w3s.link/ipfs/{CID} ดูไฟล์");
}

// ===== ดึงข้อมูลจาก IPFS Gateway =====
async function fetchFromIPFS(cid) {
  // ใช้ public gateway
  const gateways = [
    `https://ipfs.io/ipfs/${cid}`,
    `https://cloudflare-ipfs.com/ipfs/${cid}`,
    `https://gateway.pinata.cloud/ipfs/${cid}`,
    `https://w3s.link/ipfs/${cid}`,
  ];

  for (const url of gateways) {
    try {
      const response = await fetch(url);
      if (response.ok) {
        const data = await response.json();
        console.log("ดึงข้อมูลสำเร็จจาก:", url);
        return data;
      }
    } catch (error) {
      console.log(`Failed from ${url}:`, error.message);
    }
  }

  throw new Error("ไม่สามารถดึงข้อมูลจาก IPFS ได้");
}

// ตัวอย่าง: ดู NFT metadata
async function getNFTMetadata(tokenURI) {
  // แปลง ipfs:// เป็น https://
  let url = tokenURI;
  if (tokenURI.startsWith("ipfs://")) {
    const cid = tokenURI.replace("ipfs://", "");
    url = `https://ipfs.io/ipfs/${cid}`;
  }

  const response = await fetch(url);
  const metadata = await response.json();

  // แปลง image URI ด้วย
  if (metadata.image && metadata.image.startsWith("ipfs://")) {
    metadata.imageUrl = metadata.image.replace(
      "ipfs://",
      "https://ipfs.io/ipfs/"
    );
  }

  return metadata;
}
```

---

### สรุปภาพรวม Blockchain Stack

```javascript
// ===== Blockchain Development Stack =====

const blockchainStack = {
  // Layer 1 (Blockchain)
  l1: {
    networks: ["Ethereum", "Bitcoin", "Solana", "Avalanche"],
    languages: ["Solidity", "Rust", "Move"],
  },

  // Layer 2 (Scaling Solutions)
  l2: {
    networks: ["Optimism", "Arbitrum", "zkSync", "Polygon zkEVM"],
    benefit: "เร็วขึ้น ถูกลง",
  },

  // Development Tools
  devTools: {
    frameworks: ["Hardhat", "Foundry", "Truffle"],
    testing: ["Mocha", "Chai", "Waffle"],
    deployment: ["Hardhat Deploy", "Foundry Script"],
  },

  // JavaScript Libraries
  jsLibraries: {
    primary: "ethers.js v6",
    alternative: "web3.js v4, viem",
    wallet: "wagmi (React hooks)",
  },

  // Frontend
  frontend: {
    frameworks: ["React", "Next.js", "Vue"],
    walletConnect: ["MetaMask", "WalletConnect", "Coinbase Wallet"],
    ui: ["RainbowKit", "ConnectKit", "Web3Modal"],
  },

  // Storage
  storage: {
    decentralized: ["IPFS", "Arweave", "Filecoin"],
    databases: ["The Graph (indexing)", "Ceramic", "Tableland"],
  },

  // Testing Networks
  testnets: {
    ethereum: ["Sepolia", "Goerli (deprecated)"],
    polygon: ["Mumbai"],
    local: ["Hardhat Network", "Anvil (Foundry)"],
  },
};

console.log("Blockchain Development Stack:");
console.log(JSON.stringify(blockchainStack, null, 2));
```

---

## สรุปสิ่งที่เรียนในบทนี้

```javascript
const summary = {
  step1871: "Blockchain fundamentals - blocks, chain, immutability",
  step1872: "Decentralization และ Consensus Mechanisms (PoW, PoS)",
  step1873: "สร้าง Block class ด้วย JavaScript + SHA-256",
  step1874: "สร้าง Blockchain class พร้อม validation",
  step1875: "Merkle Tree implementation",
  step1876: "Smart Contracts concept",
  step1877: "Ethereum, EVM, Solidity basics",
  step1878: "ethers.js - Installation & Setup",
  step1879: "ethers.js - Provider และ Signer",
  step1880: "ethers.js - อ่านข้อมูล blockchain",
  step1881: "ethers.js - ส่ง ETH transactions",
  step1882: "ethers.js - Contract interaction",
  step1883: "ethers.js - Event listening",
  step1884: "ethers.js - ENS resolution",
  step1885: "Web3.js vs ethers.js comparison",
  step1886: "MetaMask integration",
  step1887: "Simple ERC-20 Token contract",
  step1888: "Complete DApp frontend",
  step1889: "NFT concepts, ERC-721",
  step1890: "DeFi, DEX, Yield Farming, IPFS",
};

Object.entries(summary).forEach(([step, desc]) => {
  console.log(`${step}: ${desc}`);
});
```

---

## แบบฝึกหัด (Exercises)

### Exercise 1: เพิ่ม Transaction ใน Blockchain

สร้าง Blockchain ที่รองรับ transactions โดยแต่ละ block เก็บ array ของ transactions พร้อม:
- ตรวจสอบว่า sender มีเงินเพียงพอ
- เก็บ balances ของทุก address
- ตรวจสอบ double-spending

```javascript
// โครงสร้างที่ต้องสร้าง:
class Transaction {
  constructor(fromAddress, toAddress, amount) {
    this.fromAddress = fromAddress;
    this.toAddress = toAddress;
    this.amount = amount;
    this.timestamp = Date.now();
  }
}

class TransactionBlockchain extends Blockchain {
  constructor() {
    super();
    // TODO: เพิ่ม balances tracking
    // TODO: เพิ่ม pendingTransactions
    // TODO: override addBlock ให้ตรวจสอบ balance
  }

  createTransaction(transaction) {
    // TODO: validate transaction
    // TODO: เพิ่มเข้า pendingTransactions
  }

  minePendingTransactions(minerAddress) {
    // TODO: สร้าง block จาก pendingTransactions
    // TODO: ให้ miner reward
  }

  getBalanceOfAddress(address) {
    // TODO: คำนวณ balance จาก chain
  }
}
```

---

### Exercise 2: สร้าง Digital Wallet

สร้าง Wallet class ที่จัดการ keypairs:
- สร้าง key pair (public/private)
- Sign messages ด้วย private key
- Verify signatures ด้วย public key
- เก็บ wallet ลง localStorage (encrypt ด้วย password)

```javascript
const crypto = require("crypto");

class Wallet {
  constructor() {
    // TODO: สร้าง keypair
    const { privateKey, publicKey } = crypto.generateKeyPairSync("ec", {
      namedCurve: "secp256k1",
      // ...
    });
    this.privateKey = privateKey;
    this.publicKey = publicKey;
  }

  sign(data) {
    // TODO: sign data ด้วย private key
    // return signature
  }

  static verify(data, signature, publicKey) {
    // TODO: verify signature
    // return true/false
  }

  getAddress() {
    // TODO: คำนวณ address จาก public key (hash)
  }
}

// ทดสอบ:
// const wallet = new Wallet();
// const sig = wallet.sign("Hello!");
// console.log(Wallet.verify("Hello!", sig, wallet.publicKey));
```

---

### Exercise 3: Token Balance Checker

สร้างโปรแกรม Node.js ที่:
1. รับ address จาก command line argument
2. เชื่อมต่อกับ Ethereum Sepolia testnet
3. แสดง ETH balance
4. แสดง ERC-20 token balances ของ tokens ที่กำหนด
5. แสดงประวัติ transactions 10 รายการล่าสุด

```javascript
const { ethers } = require("ethers");

const TOKENS = [
  {
    name: "USDC",
    address: "0x...", // Sepolia USDC
    decimals: 6,
  },
  {
    name: "LINK",
    address: "0x...", // Sepolia LINK
    decimals: 18,
  },
];

async function checkBalance(address) {
  const provider = new ethers.JsonRpcProvider(
    "https://rpc.sepolia.org"
  );

  // TODO: ตรวจสอบ ETH balance
  // TODO: ตรวจสอบ token balances
  // TODO: ดูประวัติ transactions
}

const address = process.argv[2];
if (!address) {
  console.log("Usage: node balance.js <address>");
  process.exit(1);
}

checkBalance(address);
```

---

### Exercise 4: NFT Gallery DApp

สร้าง Web App ที่:
1. เชื่อมต่อ MetaMask
2. ดึง NFTs ที่ user ถืออยู่ (ใช้ OpenSea API หรือ Alchemy NFT API)
3. แสดงรูปภาพและ metadata ของแต่ละ NFT
4. ให้ transfer NFT ไปยัง address อื่นได้

```javascript
// ใช้ Alchemy SDK
// npm install alchemy-sdk

const { Alchemy, Network } = require("alchemy-sdk");

const alchemy = new Alchemy({
  apiKey: "YOUR_ALCHEMY_API_KEY",
  network: Network.ETH_MAINNET,
});

async function getNFTsForOwner(ownerAddress) {
  // TODO: ดึง NFTs ทั้งหมดของ owner
  const nfts = await alchemy.nft.getNftsForOwner(ownerAddress);

  // TODO: แสดงข้อมูลแต่ละ NFT
  for (const nft of nfts.ownedNfts) {
    console.log(`${nft.title} - ${nft.contract.address} #${nft.tokenId}`);
    // TODO: แสดงรูปและ attributes
  }
}
```

---

### Exercise 5: Simple DEX Interface

สร้าง DApp ที่ interact กับ Uniswap V2:
1. แสดงราคา ETH/USDC จาก Uniswap
2. คำนวณ slippage
3. ส่ง swap transaction (optional: ใช้ testnet)
4. แสดง price chart แบบง่าย

```javascript
const { ethers } = require("ethers");

// Uniswap V2 Router ABI (ส่วนที่ใช้)
const UNISWAP_V2_ROUTER_ABI = [
  "function getAmountsOut(uint amountIn, address[] memory path) public view returns (uint[] memory amounts)",
  "function swapExactTokensForTokens(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts)",
  "function swapExactETHForTokens(uint amountOutMin, address[] calldata path, address to, uint deadline) external payable returns (uint[] memory amounts)",
];

const UNISWAP_V2_ROUTER = "0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D";
const WETH = "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2";
const USDC = "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48";

async function getETHPrice() {
  const provider = new ethers.JsonRpcProvider(
    "https://mainnet.infura.io/v3/YOUR_KEY"
  );
  const router = new ethers.Contract(
    UNISWAP_V2_ROUTER,
    UNISWAP_V2_ROUTER_ABI,
    provider
  );

  // TODO: คำนวณราคา 1 ETH = ? USDC
  const amountsOut = await router.getAmountsOut(
    ethers.parseEther("1"),
    [WETH, USDC]
  );

  const usdcOut = ethers.formatUnits(amountsOut[1], 6);
  console.log(`ราคา ETH: $${parseFloat(usdcOut).toFixed(2)} USDC`);

  // TODO: คำนวณ slippage
  // TODO: แสดง price impact
}

getETHPrice().catch(console.error);
```

---

## เนื้อหาเพิ่มเติม: Hardhat Development Environment

```javascript
// ===== Hardhat Setup =====
// npm install --save-dev hardhat

// hardhat.config.js
module.exports = {
  solidity: "0.8.19",
  networks: {
    hardhat: {
      chainId: 31337,
    },
    sepolia: {
      url: process.env.SEPOLIA_RPC_URL,
      accounts: [process.env.PRIVATE_KEY],
    },
    mainnet: {
      url: process.env.MAINNET_RPC_URL,
      accounts: [process.env.PRIVATE_KEY],
    },
  },
  etherscan: {
    apiKey: process.env.ETHERSCAN_API_KEY,
  },
};

// ===== Deploy Script =====
// scripts/deploy.js
async function main() {
  const [deployer] = await ethers.getSigners();
  console.log("Deploying with:", deployer.address);

  const balance = await ethers.provider.getBalance(deployer.address);
  console.log("Balance:", ethers.formatEther(balance));

  const SimpleToken = await ethers.getContractFactory("SimpleToken");
  const token = await SimpleToken.deploy(1000000); // 1M tokens

  await token.waitForDeployment();
  const address = await token.getAddress();
  console.log("SimpleToken deployed to:", address);
}

main().catch(console.error);

// ===== Test Script =====
// test/SimpleToken.test.js
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("SimpleToken", function () {
  let token;
  let owner, alice, bob;

  beforeEach(async function () {
    [owner, alice, bob] = await ethers.getSigners();

    const SimpleToken = await ethers.getContractFactory("SimpleToken");
    token = await SimpleToken.deploy(1000000);
    await token.waitForDeployment();
  });

  it("ควรมี total supply ถูกต้อง", async function () {
    const supply = await token.totalSupply();
    expect(supply).to.equal(ethers.parseEther("1000000"));
  });

  it("owner ควรมี balance เท่ากับ total supply", async function () {
    const balance = await token.balanceOf(owner.address);
    const supply = await token.totalSupply();
    expect(balance).to.equal(supply);
  });

  it("ควร transfer token ได้", async function () {
    const amount = ethers.parseEther("100");
    await token.transfer(alice.address, amount);

    expect(await token.balanceOf(alice.address)).to.equal(amount);
  });

  it("ควรล้มเหลวถ้า balance ไม่พอ", async function () {
    const amount = ethers.parseEther("100");
    await expect(
      token.connect(alice).transfer(bob.address, amount)
    ).to.be.revertedWith("Insufficient balance");
  });

  it("ควร emit Transfer event", async function () {
    const amount = ethers.parseEther("50");
    await expect(token.transfer(alice.address, amount))
      .to.emit(token, "Transfer")
      .withArgs(owner.address, alice.address, amount);
  });
});
```

---

## ตัวอย่างโค้ดเพิ่มเติม: Crypto Utilities

```javascript
const crypto = require("crypto");
const { ethers } = require("ethers");

// ===== Hash Functions =====
function sha256(data) {
  return crypto.createHash("sha256").update(data).digest("hex");
}

function keccak256(data) {
  return ethers.keccak256(ethers.toUtf8Bytes(data));
}

// ===== Address Utilities =====
function isValidAddress(address) {
  return ethers.isAddress(address);
}

function checksumAddress(address) {
  return ethers.getAddress(address); // EIP-55 checksum
}

function addressFromPrivateKey(privateKey) {
  const wallet = new ethers.Wallet(privateKey);
  return wallet.address;
}

// ===== Units Conversion =====
function ethToWei(eth) {
  return ethers.parseEther(eth.toString());
}

function weiToEth(wei) {
  return ethers.formatEther(wei);
}

function parseUnits(amount, decimals) {
  return ethers.parseUnits(amount.toString(), decimals);
}

function formatUnits(amount, decimals) {
  return ethers.formatUnits(amount, decimals);
}

// ===== Signing =====
async function signMessage(message, privateKey) {
  const wallet = new ethers.Wallet(privateKey);
  const signature = await wallet.signMessage(message);
  return signature;
}

function verifyMessage(message, signature) {
  const recoveredAddress = ethers.verifyMessage(message, signature);
  return recoveredAddress;
}

// ทดสอบ
console.log("SHA-256:", sha256("Hello Blockchain"));
console.log("Keccak256:", keccak256("Hello Blockchain"));

const testAddress = "0x742d35Cc6634C0532925a3b844Bc454e4438f44e";
console.log("Valid address:", isValidAddress(testAddress));
console.log("Checksum:", checksumAddress(testAddress.toLowerCase()));

console.log("1 ETH in Wei:", ethToWei(1).toString());
console.log("1 Gwei in ETH:", weiToEth(1000000000n));

// Sign และ verify message
async function testSigning() {
  const wallet = ethers.Wallet.createRandom();
  const message = "สวัสดี Blockchain!";

  const sig = await signMessage(message, wallet.privateKey);
  const recovered = verifyMessage(message, sig);

  console.log("\nOriginal address:", wallet.address);
  console.log("Recovered address:", recovered);
  console.log("Match:", wallet.address === recovered);
}

testSigning();
```

---

## ตัวอย่างโค้ดเพิ่มเติม: Multi-signature Wallet Concept

```javascript
// ===== Multi-Signature Wallet =====
// ต้องมี k จาก n signatures จึงจะ execute transaction ได้

class MultiSigWallet {
  constructor(owners, requiredSignatures) {
    if (owners.length < requiredSignatures) {
      throw new Error("Required signatures > number of owners");
    }

    this.owners = new Set(owners);
    this.required = requiredSignatures;
    this.transactions = [];
    this.confirmations = new Map();
    this.balance = 0;
  }

  // ฝากเงิน
  deposit(from, amount) {
    this.balance += amount;
    console.log(`${from} ฝาก ${amount} ETH (total: ${this.balance})`);
  }

  // เสนอ transaction
  submitTransaction(from, to, amount, data = "") {
    if (!this.owners.has(from)) throw new Error("Not an owner");
    if (amount > this.balance) throw new Error("Insufficient balance");

    const txId = this.transactions.length;
    this.transactions.push({
      id: txId,
      to,
      amount,
      data,
      executed: false,
      confirmationCount: 0,
    });
    this.confirmations.set(txId, new Set());

    console.log(`Transaction #${txId} เสนอโดย ${from}: ส่ง ${amount} ETH ไป ${to}`);

    // confirm ทันทีจาก submitter
    this.confirmTransaction(from, txId);

    return txId;
  }

  // ยืนยัน transaction
  confirmTransaction(from, txId) {
    if (!this.owners.has(from)) throw new Error("Not an owner");

    const tx = this.transactions[txId];
    if (!tx) throw new Error("Transaction not found");
    if (tx.executed) throw new Error("Already executed");

    const confirms = this.confirmations.get(txId);
    if (confirms.has(from)) throw new Error("Already confirmed");

    confirms.add(from);
    tx.confirmationCount = confirms.size;
    console.log(`${from} ยืนยัน TX #${txId} (${tx.confirmationCount}/${this.required})`);

    // execute ถ้ามี confirmations พอ
    if (tx.confirmationCount >= this.required) {
      this.executeTransaction(txId);
    }
  }

  // ยกเลิกการยืนยัน
  revokeConfirmation(from, txId) {
    if (!this.owners.has(from)) throw new Error("Not an owner");

    const tx = this.transactions[txId];
    if (tx.executed) throw new Error("Already executed");

    const confirms = this.confirmations.get(txId);
    confirms.delete(from);
    tx.confirmationCount = confirms.size;
    console.log(`${from} ยกเลิกการยืนยัน TX #${txId}`);
  }

  // execute transaction
  executeTransaction(txId) {
    const tx = this.transactions[txId];
    if (tx.executed) throw new Error("Already executed");
    if (tx.confirmationCount < this.required) {
      throw new Error("Insufficient confirmations");
    }

    this.balance -= tx.amount;
    tx.executed = true;
    console.log(`✓ TX #${txId} executed: ส่ง ${tx.amount} ETH ไป ${tx.to}`);
  }

  getStatus(txId) {
    const tx = this.transactions[txId];
    const confirms = this.confirmations.get(txId);
    return {
      ...tx,
      confirmers: [...confirms],
    };
  }
}

// ทดสอบ Multi-Sig Wallet (2-of-3)
const wallet = new MultiSigWallet(["Alice", "Bob", "Carol"], 2);
wallet.deposit("Dave", 10);

const txId = wallet.submitTransaction("Alice", "Recipient", 5);
wallet.confirmTransaction("Bob", txId);
// tx จะ execute หลังจาก Bob confirm (2/2 met)

console.log("\nFinal balance:", wallet.balance);
console.log("TX Status:", wallet.getStatus(txId));
```

---

## ตัวอย่างโค้ดเพิ่มเติม: Gas Estimation

```javascript
const { ethers } = require("ethers");

async function estimateGas() {
  const provider = new ethers.JsonRpcProvider("http://localhost:8545");
  const signer = await provider.getSigner();

  // 1. Estimate gas สำหรับ ETH transfer
  const gasEstimate = await provider.estimateGas({
    from: signer.address,
    to: "0x70997970C51812dc3A010C7d01b50e0d17dc79C8",
    value: ethers.parseEther("0.1"),
  });
  console.log("Gas estimate (ETH transfer):", gasEstimate.toString());

  // 2. ดู fee data
  const feeData = await provider.getFeeData();
  const gasPrice = feeData.gasPrice;
  const maxFeePerGas = feeData.maxFeePerGas;
  const maxPriorityFeePerGas = feeData.maxPriorityFeePerGas;

  console.log("\nFee Data:");
  console.log("Gas Price:", ethers.formatUnits(gasPrice, "gwei"), "Gwei");
  console.log("Max Fee:", ethers.formatUnits(maxFeePerGas, "gwei"), "Gwei");
  console.log("Priority Fee:", ethers.formatUnits(maxPriorityFeePerGas, "gwei"), "Gwei");

  // 3. คำนวณค่าใช้จ่ายทั้งหมด
  const totalCostWei = gasEstimate * gasPrice;
  const totalCostEth = ethers.formatEther(totalCostWei);
  console.log("\nTotal gas cost:", totalCostEth, "ETH");

  // 4. EIP-1559 transaction
  const tx = {
    to: "0x70997970C51812dc3A010C7d01b50e0d17dc79C8",
    value: ethers.parseEther("0.01"),
    maxFeePerGas: maxFeePerGas,
    maxPriorityFeePerGas: maxPriorityFeePerGas,
    gasLimit: gasEstimate,
  };

  console.log("\nEIP-1559 Transaction:");
  console.log("Max Fee:", ethers.formatUnits(tx.maxFeePerGas, "gwei"), "Gwei");
  console.log("Priority Fee:", ethers.formatUnits(tx.maxPriorityFeePerGas, "gwei"), "Gwei");
}

estimateGas().catch(console.error);
```

---

## ตัวอย่างโค้ดเพิ่มเติม: Contract Events Decoder

```javascript
const { ethers } = require("ethers");

// Decode transaction logs manually
async function decodeLogs(txHash, abi) {
  const provider = new ethers.JsonRpcProvider(
    "https://rpc.sepolia.org"
  );

  const receipt = await provider.getTransactionReceipt(txHash);
  if (!receipt) {
    console.log("Transaction not found");
    return;
  }

  const iface = new ethers.Interface(abi);

  console.log(`Transaction ${txHash}:`);
  console.log(`Block: ${receipt.blockNumber}`);
  console.log(`Status: ${receipt.status === 1 ? "สำเร็จ" : "ล้มเหลว"}`);
  console.log(`Gas used: ${receipt.gasUsed.toString()}`);
  console.log(`\nLogs (${receipt.logs.length} total):`);

  for (const log of receipt.logs) {
    try {
      const parsed = iface.parseLog(log);
      if (parsed) {
        console.log(`\n  Event: ${parsed.name}`);
        parsed.fragment.inputs.forEach((input, i) => {
          let value = parsed.args[i];
          if (typeof value === "bigint") {
            value = value.toString();
          }
          console.log(`    ${input.name}: ${value}`);
        });
      }
    } catch (e) {
      console.log(`  Unknown event (topic: ${log.topics[0].slice(0, 10)}...)`);
    }
  }
}

// ตัวอย่างการใช้
const ERC20_ABI = [
  "event Transfer(address indexed from, address indexed to, uint256 value)",
  "event Approval(address indexed owner, address indexed spender, uint256 value)",
];

// decodeLogs("0xtxhash...", ERC20_ABI);

// ===== Watch Pending Transactions =====
async function watchPendingTransactions() {
  const provider = new ethers.WebSocketProvider(
    "wss://sepolia.infura.io/ws/v3/YOUR_KEY"
  );

  console.log("กำลังฟัง pending transactions...");

  provider.on("pending", async (txHash) => {
    try {
      const tx = await provider.getTransaction(txHash);
      if (tx && tx.value > ethers.parseEther("1")) {
        console.log(`\nLarge TX detected: ${txHash}`);
        console.log(`From: ${tx.from}`);
        console.log(`To: ${tx.to}`);
        console.log(`Value: ${ethers.formatEther(tx.value)} ETH`);
      }
    } catch (e) {
      // ignore
    }
  });
}
```

---

## เคล็ดลับและข้อควรระวัง

```javascript
// ===== Security Best Practices =====

// 1. ห้ามเก็บ Private Key ใน code
// ❌ Bad
const privateKey = "0xabcd1234...";

// ✓ Good - ใช้ environment variables
const privateKey2 = process.env.PRIVATE_KEY;

// 2. ใช้ .env file
// npm install dotenv
require("dotenv").config();
// .env file: PRIVATE_KEY=0xabcd...

// 3. ตรวจสอบ address checksum
function safeGetAddress(address) {
  try {
    return ethers.getAddress(address); // throw ถ้า invalid
  } catch {
    throw new Error(`Invalid address: ${address}`);
  }
}

// 4. Handle BigInt อย่างระวัง
// ❌ Bad
const amount = 0.1 * 1e18; // floating point error!

// ✓ Good
const amount2 = ethers.parseEther("0.1");

// 5. จัดการ errors อย่างครบถ้วน
async function safeSendTransaction(wallet, to, amount) {
  try {
    const tx = await wallet.sendTransaction({ to, value: amount });
    const receipt = await tx.wait();

    if (receipt.status === 0) {
      throw new Error("Transaction reverted");
    }

    return receipt;
  } catch (error) {
    if (error.code === "INSUFFICIENT_FUNDS") {
      throw new Error("ยอดเงินไม่เพียงพอ");
    } else if (error.code === "NETWORK_ERROR") {
      throw new Error("ปัญหาการเชื่อมต่อ network");
    } else if (error.code === 4001) {
      throw new Error("User ยกเลิก transaction");
    }
    throw error;
  }
}

// 6. ใช้ nonce management สำหรับ high-frequency transactions
class NonceManager {
  constructor(wallet) {
    this.wallet = wallet;
    this.nonce = null;
  }

  async getNextNonce() {
    if (this.nonce === null) {
      this.nonce = await this.wallet.getNonce();
    }
    return this.nonce++;
  }

  reset() {
    this.nonce = null;
  }
}

// 7. Retry mechanism
async function withRetry(fn, maxRetries = 3, delay = 1000) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxRetries) throw error;
      console.log(`Attempt ${attempt} failed, retrying in ${delay}ms...`);
      await new Promise(resolve => setTimeout(resolve, delay * attempt));
    }
  }
}

// 8. ตรวจสอบ network ก่อนทำ transaction
async function ensureNetwork(provider, expectedChainId) {
  const network = await provider.getNetwork();
  if (Number(network.chainId) !== expectedChainId) {
    throw new Error(
      `Wrong network! Expected chain ${expectedChainId}, got ${network.chainId}`
    );
  }
}
```

---

## จบ Part 95: Blockchain กับ JavaScript

ในบทนี้เราได้เรียนรู้:

1. **Blockchain Fundamentals** - โครงสร้าง block, chain, และ consensus mechanisms
2. **สร้าง Blockchain ตั้งแต่ต้น** - Block class, Blockchain class, Mining, Merkle Tree
3. **Smart Contracts** - แนวคิด Solidity และ EVM
4. **ethers.js** - Library หลักสำหรับ Ethereum development
5. **MetaMask Integration** - การเชื่อมต่อ wallet กับ DApp
6. **DApp Development** - Token contract และ Frontend สมบูรณ์
7. **NFTs** - ERC-721 standard และ metadata
8. **DeFi** - DEX, Liquidity Pools, Yield Farming
9. **IPFS** - Decentralized storage สำหรับ NFT metadata
10. **Security** - Best practices สำหรับ Blockchain development

### ขั้นตอนถัดไป

- เรียน Solidity อย่างลึกซึ้งที่ [cryptozombies.io](https://cryptozombies.io)
- ฝึกใช้ Hardhat framework
- สร้าง DApp บน Sepolia testnet
- ศึกษา OpenZeppelin contracts library
- เรียน The Graph สำหรับ blockchain data indexing
- ลองสร้าง NFT collection บน testnet

---

*Part 95 เสร็จสมบูรณ์ | Steps 1871-1890 | JavaScript Blockchain Development*
