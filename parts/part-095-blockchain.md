# Part 95: Blockchain กับ JavaScript (Steps 1871-1890)

## Blockchain ด้วย JavaScript - จาก Fundamentals สู่ DApp

ในส่วนนี้เราจะเรียนรู้เกี่ยวกับ blockchain technology ตั้งแต่ fundamentals การสร้าง blockchain อย่างง่าย smart contracts บน Ethereum ไปจนถึงการสร้าง DApp ด้วย ethers.js

---

## Step 1871: Blockchain Fundamentals

Blockchain คือ distributed ledger (สมุดบัญชีกระจายศูนย์) ที่ข้อมูลถูกเก็บในรูปแบบ blocks ที่เชื่อมต่อกัน

```javascript
// Blockchain Core Concepts

// 1. Block - หน่วยข้อมูลพื้นฐาน
const block = {
  index: 0,              // ลำดับที่ของ block
  timestamp: 1234567890, // เวลาที่สร้าง
  data: 'Genesis Block', // ข้อมูลใน block
  previousHash: '0',     // hash ของ block ก่อนหน้า
  hash: 'abc123...',     // hash ของ block นี้
  nonce: 0,              // ตัวเลขสำหรับ Proof of Work
};

// 2. Chain - chain ของ blocks
// Block 0 (Genesis) <- Block 1 <- Block 2 <- ...
// แต่ละ block อ้างอิงถึง hash ของ block ก่อนหน้า

// 3. คุณสมบัติของ Blockchain
const blockchainProperties = {
  immutability: 'เปลี่ยนข้อมูลเก่าไม่ได้ - ต้องเปลี่ยน hash ทุก block หลังจากนั้น',
  decentralized: 'ไม่มีจุดศูนย์กลาง - ทุก node มี copy ของ blockchain',
  transparent: 'ทุกคนดูได้ แต่ pseudonymous',
  consensus: 'ทุก node ต้องเห็นด้วยกับ state ของ chain',
};

// 4. Types of blockchain
const blockchainTypes = {
  public: ['Bitcoin', 'Ethereum'],          // ทุกคนเข้าร่วมได้
  private: ['Hyperledger Fabric'],          // ต้องได้รับอนุญาต
  consortium: ['R3 Corda'],                 // กลุ่มองค์กร
  hybrid: ['Dragonchain'],                  // ผสมผสาน
};

// 5. Consensus mechanisms
const consensusMechanisms = {
  ProofOfWork: 'Bitcoin - ใช้พลังงาน CPU',
  ProofOfStake: 'Ethereum 2.0 - ใช้ stake tokens',
  DelegatedPoS: 'EOS - เลือกตัวแทน',
  ProofOfAuthority: 'Private chains - trusted validators',
};
```

---

## Step 1872: Building a Simple Blockchain in JavaScript

```javascript
// สร้าง Blockchain อย่างง่ายด้วย JavaScript
const crypto = require('crypto');

class Block {
  constructor(index, timestamp, data, previousHash = '') {
    this.index = index;
    this.timestamp = timestamp;
    this.data = data;
    this.previousHash = previousHash;
    this.nonce = 0;
    this.hash = this.calculateHash();
  }

  calculateHash() {
    return crypto
      .createHash('sha256')
      .update(
        this.index +
        this.timestamp +
        JSON.stringify(this.data) +
        this.previousHash +
        this.nonce
      )
      .digest('hex');
  }

  // Proof of Work - หา hash ที่ขึ้นต้นด้วย 0 ตามจำนวน difficulty
  mineBlock(difficulty) {
    const target = '0'.repeat(difficulty);
    
    console.log(`Mining block ${this.index}...`);
    const startTime = Date.now();
    
    while (this.hash.substring(0, difficulty) !== target) {
      this.nonce++;
      this.hash = this.calculateHash();
    }
    
    const elapsed = (Date.now() - startTime) / 1000;
    console.log(`Block ${this.index} mined: ${this.hash} (${elapsed}s, nonce: ${this.nonce})`);
  }
}

class Blockchain {
  constructor(difficulty = 4) {
    this.chain = [this.createGenesisBlock()];
    this.difficulty = difficulty;
    this.pendingTransactions = [];
    this.miningReward = 100;
  }

  createGenesisBlock() {
    return new Block(0, new Date().toISOString(), 'Genesis Block', '0');
  }

  getLatestBlock() {
    return this.chain[this.chain.length - 1];
  }

  minePendingTransactions(minerAddress) {
    const block = new Block(
      this.chain.length,
      new Date().toISOString(),
      this.pendingTransactions,
      this.getLatestBlock().hash
    );

    block.mineBlock(this.difficulty);
    
    console.log('Block successfully mined!');
    this.chain.push(block);

    // Reward miner
    this.pendingTransactions = [
      {
        from: null,
        to: minerAddress,
        amount: this.miningReward,
        type: 'MINING_REWARD',
      },
    ];
  }

  addTransaction(transaction) {
    if (!transaction.from || !transaction.to) {
      throw new Error('Transaction must include from and to address');
    }
    if (!transaction.amount || transaction.amount <= 0) {
      throw new Error('Transaction amount must be positive');
    }
    if (this.getBalance(transaction.from) < transaction.amount) {
      throw new Error('Insufficient balance');
    }
    
    this.pendingTransactions.push(transaction);
  }

  getBalance(address) {
    let balance = 0;
    
    for (const block of this.chain) {
      if (!Array.isArray(block.data)) continue;
      
      for (const tx of block.data) {
        if (tx.from === address) balance -= tx.amount;
        if (tx.to === address) balance += tx.amount;
      }
    }
    
    return balance;
  }

  isChainValid() {
    for (let i = 1; i < this.chain.length; i++) {
      const currentBlock = this.chain[i];
      const previousBlock = this.chain[i - 1];
      
      // ตรวจสอบ hash ของ current block
      if (currentBlock.hash !== currentBlock.calculateHash()) {
        console.error(`Block ${i} has invalid hash`);
        return false;
      }
      
      // ตรวจสอบ previousHash
      if (currentBlock.previousHash !== previousBlock.hash) {
        console.error(`Block ${i} has invalid previousHash`);
        return false;
      }
    }
    return true;
  }

  toString() {
    return JSON.stringify(this.chain, null, 2);
  }
}

// ใช้งาน
const myChain = new Blockchain(2); // difficulty 2 (เร็วกว่า difficulty 4)

// เพิ่ม initial balance
myChain.chain[0].data = [
  { from: null, to: 'alice', amount: 1000, type: 'GENESIS' },
  { from: null, to: 'bob', amount: 500, type: 'GENESIS' },
];

// เพิ่ม transactions
myChain.addTransaction({ from: 'alice', to: 'bob', amount: 100 });
myChain.addTransaction({ from: 'alice', to: 'charlie', amount: 50 });

// Mine block
myChain.minePendingTransactions('miner-address');

console.log('\nAlice balance:', myChain.getBalance('alice'));
console.log('Bob balance:', myChain.getBalance('bob'));
console.log('Is valid:', myChain.isChainValid());

// ลองแก้ไข data (จะทำให้ chain invalid)
myChain.chain[1].data[0].amount = 9999;
console.log('After tampering - Is valid:', myChain.isChainValid()); // false
```

---

## Step 1873: Cryptographic Hashing (SHA-256)

```javascript
// SHA-256 Hashing ใน JavaScript

const crypto = require('crypto');

// Basic SHA-256
function sha256(data) {
  return crypto.createHash('sha256').update(data).digest('hex');
}

console.log(sha256('Hello, Bitcoin!'));
// ผล: hash 64 ตัวอักษร (256 bits)

// คุณสมบัติของ SHA-256
// 1. Deterministic: input เดียวกัน -> output เดียวกันเสมอ
console.log(sha256('test') === sha256('test')); // true

// 2. One-way: หา input จาก hash ไม่ได้
// sha256('?') = 'abc123...' - ต้องลองทั้งหมด

// 3. Avalanche effect: เปลี่ยนนิดเดียว -> hash เปลี่ยนมาก
console.log(sha256('Hello'));  // 185f...
console.log(sha256('hello'));  // 2cf2... ต่างกันมาก

// 4. Fixed length: ไม่ว่า input ยาวแค่ไหน output = 256 bits

// Double hashing (ใช้ใน Bitcoin)
function doubleSHA256(data) {
  const first = crypto.createHash('sha256').update(data).digest();
  return crypto.createHash('sha256').update(first).digest('hex');
}

// RIPEMD-160 (ใช้ใน Bitcoin address generation)
function ripemd160(data) {
  return crypto.createHash('ripemd160').update(data).digest('hex');
}

// Hash160 = RIPEMD160(SHA256(data))
function hash160(data) {
  const sha256Hash = crypto.createHash('sha256').update(data).digest();
  return crypto.createHash('ripemd160').update(sha256Hash).digest('hex');
}

// ทำ Bitcoin-style address จาก public key
function generateBitcoinAddress(publicKeyHex) {
  const publicKey = Buffer.from(publicKeyHex, 'hex');
  
  // Step 1: SHA256 -> RIPEMD160
  const hash = hash160(publicKey);
  
  // Step 2: Add version byte (0x00 for mainnet)
  const versionedHash = '00' + hash;
  
  // Step 3: Checksum = first 4 bytes of double SHA256
  const checksum = doubleSHA256(Buffer.from(versionedHash, 'hex')).slice(0, 8);
  
  // Step 4: Concatenate
  const final = versionedHash + checksum;
  
  // Step 5: Base58Check encoding
  return base58Encode(Buffer.from(final, 'hex'));
}

// Base58 encoding (Bitcoin uses this, no 0, O, I, l)
function base58Encode(buffer) {
  const ALPHABET = '123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz';
  
  let num = BigInt('0x' + buffer.toString('hex'));
  let encoded = '';
  
  while (num > 0n) {
    encoded = ALPHABET[Number(num % 58n)] + encoded;
    num = num / 58n;
  }
  
  // Leading zeros
  for (const byte of buffer) {
    if (byte !== 0) break;
    encoded = '1' + encoded;
  }
  
  return encoded;
}
```

---

## Step 1874: Merkle Trees

```javascript
// Merkle Tree - โครงสร้างข้อมูลสำคัญใน Blockchain

const crypto = require('crypto');

function sha256(data) {
  return crypto.createHash('sha256').update(data).digest('hex');
}

class MerkleTree {
  constructor(data) {
    this.leaves = data.map(d => sha256(JSON.stringify(d)));
    this.tree = this._buildTree(this.leaves);
  }

  _buildTree(leaves) {
    if (leaves.length === 0) return [];
    if (leaves.length === 1) return leaves;

    const tree = [leaves];
    let currentLevel = leaves;

    while (currentLevel.length > 1) {
      const nextLevel = [];
      
      for (let i = 0; i < currentLevel.length; i += 2) {
        const left = currentLevel[i];
        const right = i + 1 < currentLevel.length ? currentLevel[i + 1] : left; // duplicate last if odd
        nextLevel.push(sha256(left + right));
      }
      
      tree.push(nextLevel);
      currentLevel = nextLevel;
    }

    return tree;
  }

  get root() {
    if (this.tree.length === 0) return null;
    return this.tree[this.tree.length - 1][0];
  }

  // สร้าง proof ว่า leaf อยู่ใน tree
  getProof(leafIndex) {
    const proof = [];
    let index = leafIndex;

    for (let level = 0; level < this.tree.length - 1; level++) {
      const levelNodes = this.tree[level];
      const isRightNode = index % 2 === 1;
      const siblingIndex = isRightNode ? index - 1 : index + 1;

      if (siblingIndex < levelNodes.length) {
        proof.push({
          hash: levelNodes[siblingIndex],
          position: isRightNode ? 'left' : 'right',
        });
      }

      index = Math.floor(index / 2);
    }

    return proof;
  }

  // ตรวจสอบ proof
  verifyProof(leafData, proof) {
    let hash = sha256(JSON.stringify(leafData));

    for (const { hash: siblingHash, position } of proof) {
      if (position === 'left') {
        hash = sha256(siblingHash + hash);
      } else {
        hash = sha256(hash + siblingHash);
      }
    }

    return hash === this.root;
  }

  printTree() {
    console.log('Merkle Tree:');
    for (let level = this.tree.length - 1; level >= 0; level--) {
      const levelName = level === this.tree.length - 1 ? 'Root' : `Level ${level}`;
      console.log(`  ${levelName}: [${this.tree[level].map(h => h.slice(0, 8)).join(', ')}]`);
    }
  }
}

// ใช้งาน
const transactions = [
  { from: 'Alice', to: 'Bob', amount: 50 },
  { from: 'Bob', to: 'Charlie', amount: 30 },
  { from: 'Charlie', to: 'Alice', amount: 20 },
  { from: 'Alice', to: 'Dave', amount: 10 },
];

const tree = new MerkleTree(transactions);
tree.printTree();
console.log('\nRoot:', tree.root);

// Verify inclusion proof
const proof = tree.getProof(1); // proof ว่า transaction index 1 อยู่ใน tree
const isValid = tree.verifyProof(transactions[1], proof);
console.log('\nProof valid:', isValid); // true

// ปลอมแปลง transaction
const fakeTx = { from: 'Bob', to: 'Charlie', amount: 30000 };
const isFakeValid = tree.verifyProof(fakeTx, proof);
console.log('Fake proof valid:', isFakeValid); // false
```

---

## Step 1875: Proof of Work

```javascript
// Proof of Work - กลไก Consensus ของ Bitcoin

class ProofOfWork {
  constructor(difficulty) {
    this.difficulty = difficulty;
    this.target = '0'.repeat(difficulty);
  }

  mine(data) {
    let nonce = 0;
    let hash;
    let iterations = 0;
    const startTime = Date.now();
    
    do {
      nonce++;
      iterations++;
      hash = this.calculateHash(data, nonce);
    } while (!hash.startsWith(this.target));
    
    const elapsed = (Date.now() - startTime) / 1000;
    const hashRate = Math.floor(iterations / elapsed);
    
    return {
      nonce,
      hash,
      iterations,
      time: elapsed,
      hashRate: `${hashRate.toLocaleString()} H/s`,
    };
  }

  calculateHash(data, nonce) {
    const crypto = require('crypto');
    return crypto
      .createHash('sha256')
      .update(JSON.stringify(data) + nonce)
      .digest('hex');
  }

  verify(data, nonce, hash) {
    const calculated = this.calculateHash(data, nonce);
    return calculated === hash && hash.startsWith(this.target);
  }
}

// Difficulty Adjustment
class DifficultyAdjuster {
  constructor(targetBlockTimeMs = 10000) { // 10 seconds per block
    this.targetBlockTime = targetBlockTimeMs;
    this.adjustmentInterval = 10; // adjust every 10 blocks
    this.blockTimes = [];
  }

  recordBlockTime(timeMs) {
    this.blockTimes.push(timeMs);
    
    if (this.blockTimes.length === this.adjustmentInterval) {
      return this.adjust();
    }
    return null;
  }

  adjust() {
    const totalTime = this.blockTimes.reduce((s, t) => s + t, 0);
    const avgTime = totalTime / this.blockTimes.length;
    const expectedTime = this.targetBlockTime * this.adjustmentInterval;
    
    // Bitcoin formula: new difficulty = old difficulty * (actual time / expected time)
    const ratio = totalTime / expectedTime;
    
    this.blockTimes = [];
    
    if (ratio < 0.5) return { action: 'increase', reason: 'Blocks coming too fast' };
    if (ratio > 2) return { action: 'decrease', reason: 'Blocks coming too slow' };
    return { action: 'maintain', ratio };
  }
}

// ทดสอบ
function testProofOfWork() {
  console.log('Testing Proof of Work:\n');
  
  for (let difficulty = 1; difficulty <= 5; difficulty++) {
    const pow = new ProofOfWork(difficulty);
    const result = pow.mine({ transactions: ['tx1', 'tx2', 'tx3'] });
    
    console.log(`Difficulty ${difficulty}:`);
    console.log(`  Nonce: ${result.nonce}`);
    console.log(`  Iterations: ${result.iterations.toLocaleString()}`);
    console.log(`  Time: ${result.time.toFixed(3)}s`);
    console.log(`  Hash rate: ${result.hashRate}`);
    console.log(`  Hash: ${result.hash.slice(0, 20)}...`);
  }
}

testProofOfWork();
```

---

## Step 1876: Smart Contracts Overview

```javascript
// Smart Contracts - self-executing contracts บน blockchain

// คุณสมบัติ Smart Contracts
const smartContractProperties = {
  selfExecuting: 'ทำงานอัตโนมัติเมื่อ conditions ตรง',
  immutable: 'เปลี่ยนแปลงหลัง deploy ไม่ได้',
  transparent: 'ทุกคนดู code ได้',
  trustless: 'ไม่ต้องเชื่อใจ intermediary',
  decentralized: 'รันบน EVM ทุก node',
};

// Use cases
const smartContractUseCases = [
  'DeFi (Decentralized Finance) - lending, swaps',
  'NFTs - ownership, royalties',
  'DAOs - governance',
  'Escrow - automatic release',
  'Insurance - automatic claims',
  'Supply chain - tracking',
  'Voting systems',
];

// Ethereum Virtual Machine (EVM)
// EVM เป็น sandboxed environment ที่รัน smart contract bytecode
// ทุก operation ใช้ "gas" เป็นค่าธรรมเนียม

const evmConcepts = {
  gas: 'หน่วยที่วัด computational effort',
  gasPrice: 'ราคาต่อ gas (wei)',
  gasLimit: 'maximum gas ที่ยอมจ่าย',
  wei: 'หน่วยเล็กที่สุดของ ETH (1 ETH = 10^18 wei)',
  gwei: 'Giga-wei = 10^9 wei (ใช้บอก gas price)',
};

// Gas calculation
const gasExamples = {
  simpleTransfer: '21,000 gas',
  erc20Transfer: '~50,000 gas',
  uniswapSwap: '~150,000-300,000 gas',
  contractDeploy: '~200,000-5,000,000 gas',
};

// ETH unit converter
function weiToEther(wei) {
  return Number(BigInt(wei)) / 1e18;
}

function etherToWei(ether) {
  return BigInt(Math.round(ether * 1e18));
}

function gweiToWei(gwei) {
  return BigInt(Math.round(gwei * 1e9));
}

// Gas cost calculator
function calculateGasCost(gasUsed, gasPriceGwei, ethPriceUSD = 2000) {
  const gasCostEth = (gasUsed * gasPriceGwei) / 1e9;
  const gasCostUSD = gasCostEth * ethPriceUSD;
  return {
    eth: gasCostEth.toFixed(8),
    usd: gasCostUSD.toFixed(4),
    gwei: gasPriceGwei,
    gasUsed,
  };
}

// ตัวอย่าง
const swapCost = calculateGasCost(200000, 30, 2000);
console.log('Uniswap swap cost:');
console.log(`  ETH: ${swapCost.eth}`);  // 0.00600000 ETH
console.log(`  USD: $${swapCost.usd}`);  // $12.0000
```

---

## Step 1877: Solidity Basics

```solidity
// Solidity - Programming language สำหรับ Ethereum Smart Contracts
// ไฟล์นี้เป็น .sol (ใส่ comment อธิบาย)

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

// Simple Storage Contract
contract SimpleStorage {
    uint256 private storedValue;
    address public owner;
    
    event ValueChanged(uint256 oldValue, uint256 newValue);
    
    constructor() {
        owner = msg.sender;
        storedValue = 0;
    }
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }
    
    function store(uint256 value) public onlyOwner {
        uint256 old = storedValue;
        storedValue = value;
        emit ValueChanged(old, value);
    }
    
    function retrieve() public view returns (uint256) {
        return storedValue;
    }
}

// ERC-20 Token Contract
contract MyToken {
    string public name;
    string public symbol;
    uint8 public decimals;
    uint256 public totalSupply;
    
    mapping(address => uint256) private balances;
    mapping(address => mapping(address => uint256)) private allowances;
    
    event Transfer(address indexed from, address indexed to, uint256 amount);
    event Approval(address indexed owner, address indexed spender, uint256 amount);
    
    constructor(string memory _name, string memory _symbol) {
        name = _name;
        symbol = _symbol;
        decimals = 18;
        totalSupply = 1000000 * 10 ** decimals;
        balances[msg.sender] = totalSupply;
    }
    
    function balanceOf(address account) public view returns (uint256) {
        return balances[account];
    }
    
    function transfer(address to, uint256 amount) public returns (bool) {
        require(to != address(0), "Transfer to zero address");
        require(balances[msg.sender] >= amount, "Insufficient balance");
        
        balances[msg.sender] -= amount;
        balances[to] += amount;
        
        emit Transfer(msg.sender, to, amount);
        return true;
    }
    
    function approve(address spender, uint256 amount) public returns (bool) {
        allowances[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }
    
    function transferFrom(address from, address to, uint256 amount) public returns (bool) {
        require(allowances[from][msg.sender] >= amount, "Allowance exceeded");
        require(balances[from] >= amount, "Insufficient balance");
        
        allowances[from][msg.sender] -= amount;
        balances[from] -= amount;
        balances[to] += amount;
        
        emit Transfer(from, to, amount);
        return true;
    }
}

// NFT Contract (ERC-721)
contract MyNFT {
    string public name = "My NFT Collection";
    string public symbol = "MNFT";
    
    uint256 private _tokenIdCounter;
    
    mapping(uint256 => address) private _owners;
    mapping(address => uint256) private _balances;
    mapping(uint256 => string) private _tokenURIs;
    
    event Transfer(address indexed from, address indexed to, uint256 indexed tokenId);
    event Minted(address indexed to, uint256 indexed tokenId, string tokenURI);
    
    function mint(address to, string memory tokenURI) public returns (uint256) {
        uint256 tokenId = _tokenIdCounter++;
        _owners[tokenId] = to;
        _balances[to]++;
        _tokenURIs[tokenId] = tokenURI;
        
        emit Transfer(address(0), to, tokenId);
        emit Minted(to, tokenId, tokenURI);
        
        return tokenId;
    }
    
    function ownerOf(uint256 tokenId) public view returns (address) {
        address owner = _owners[tokenId];
        require(owner != address(0), "Token does not exist");
        return owner;
    }
    
    function tokenURI(uint256 tokenId) public view returns (string memory) {
        require(_owners[tokenId] != address(0), "Token does not exist");
        return _tokenURIs[tokenId];
    }
    
    function balanceOf(address owner) public view returns (uint256) {
        return _balances[owner];
    }
}
```

---

## Step 1878: ethers.js - Connecting to Networks

```javascript
// npm install ethers

const { ethers } = require('ethers');

// 1. Connect to networks

// Mainnet ผ่าน Infura
const mainnetProvider = new ethers.JsonRpcProvider(
  `https://mainnet.infura.io/v3/${process.env.INFURA_PROJECT_ID}`
);

// Testnet (Sepolia)
const sepoliaProvider = new ethers.JsonRpcProvider(
  `https://sepolia.infura.io/v3/${process.env.INFURA_PROJECT_ID}`
);

// Local Hardhat/Ganache
const localProvider = new ethers.JsonRpcProvider('http://localhost:8545');

// Alchemy
const alchemyProvider = new ethers.AlchemyProvider(
  'homestead',
  process.env.ALCHEMY_API_KEY
);

// 2. Get network info
async function getNetworkInfo(provider) {
  const network = await provider.getNetwork();
  const blockNumber = await provider.getBlockNumber();
  const gasPrice = await provider.getFeeData();
  
  return {
    name: network.name,
    chainId: Number(network.chainId),
    blockNumber,
    gasPrice: ethers.formatUnits(gasPrice.gasPrice, 'gwei') + ' gwei',
    maxFeePerGas: ethers.formatUnits(gasPrice.maxFeePerGas || 0, 'gwei') + ' gwei',
  };
}

// 3. Get block info
async function getBlockInfo(provider, blockNumber = 'latest') {
  const block = await provider.getBlock(blockNumber);
  
  return {
    number: block.number,
    timestamp: new Date(block.timestamp * 1000).toISOString(),
    hash: block.hash,
    transactionCount: block.transactions.length,
    miner: block.miner,
    gasUsed: block.gasUsed.toString(),
    gasLimit: block.gasLimit.toString(),
    baseFeePerGas: block.baseFeePerGas
      ? ethers.formatUnits(block.baseFeePerGas, 'gwei') + ' gwei'
      : 'N/A',
  };
}

// ใช้งาน
async function main() {
  const provider = new ethers.JsonRpcProvider(
    'https://eth-mainnet.g.alchemy.com/v2/your-api-key'
  );
  
  const network = await getNetworkInfo(provider);
  console.log('Network:', network);
  
  const latestBlock = await getBlockInfo(provider);
  console.log('Latest Block:', latestBlock);
}
```

---

## Step 1879: Wallets and Key Management

```javascript
// Wallet Management ด้วย ethers.js

const { ethers } = require('ethers');

// 1. Create new wallet
function createNewWallet() {
  const wallet = ethers.Wallet.createRandom();
  
  return {
    address: wallet.address,
    privateKey: wallet.privateKey,
    mnemonic: wallet.mnemonic.phrase,
  };
}

// 2. Restore from private key
function walletFromPrivateKey(privateKey) {
  const wallet = new ethers.Wallet(privateKey);
  return {
    address: wallet.address,
    privateKey: wallet.privateKey,
  };
}

// 3. Restore from mnemonic (BIP-39)
function walletFromMnemonic(mnemonic, path = "m/44'/60'/0'/0/0") {
  const hdNode = ethers.HDNodeWallet.fromPhrase(mnemonic, null, path);
  return {
    address: hdNode.address,
    privateKey: hdNode.privateKey,
    path,
  };
}

// 4. Derive multiple wallets from single mnemonic
function deriveWallets(mnemonic, count = 10) {
  const wallets = [];
  for (let i = 0; i < count; i++) {
    const path = `m/44'/60'/0'/0/${i}`;
    const wallet = ethers.HDNodeWallet.fromPhrase(mnemonic, null, path);
    wallets.push({
      index: i,
      path,
      address: wallet.address,
    });
  }
  return wallets;
}

// 5. Encrypt/Decrypt wallet (Keystore JSON)
async function encryptWallet(privateKey, password) {
  const wallet = new ethers.Wallet(privateKey);
  const encrypted = await wallet.encrypt(password, {
    scrypt: { N: 1024 }, // lower for dev, use default for production
  });
  return encrypted; // JSON string
}

async function decryptWallet(keystoreJson, password) {
  const wallet = await ethers.Wallet.fromEncryptedJson(keystoreJson, password);
  return {
    address: wallet.address,
    privateKey: wallet.privateKey,
  };
}

// 6. Sign messages
async function signMessage(privateKey, message) {
  const wallet = new ethers.Wallet(privateKey);
  const signature = await wallet.signMessage(message);
  return signature;
}

// 7. Verify signed message
function verifyMessage(message, signature) {
  const recoveredAddress = ethers.verifyMessage(message, signature);
  return recoveredAddress;
}

// ใช้งาน
async function walletDemo() {
  // สร้าง wallet ใหม่
  const newWallet = createNewWallet();
  console.log('New wallet created:');
  console.log('  Address:', newWallet.address);
  console.log('  Mnemonic:', newWallet.mnemonic);
  // WARNING: ไม่ควรพิมพ์ private key จริงๆ

  // Derive wallets
  const derived = deriveWallets(newWallet.mnemonic, 3);
  console.log('\nDerived wallets:');
  derived.forEach(w => console.log(`  [${w.index}] ${w.address}`));
  
  // Sign and verify
  const message = 'Hello, Ethereum!';
  const signature = await signMessage(newWallet.privateKey, message);
  const recovered = verifyMessage(message, signature);
  
  console.log('\nMessage signed:', message);
  console.log('Signature:', signature.slice(0, 20) + '...');
  console.log('Recovered address:', recovered);
  console.log('Matches:', recovered.toLowerCase() === newWallet.address.toLowerCase());
}
```

---

## Step 1880: Reading Blockchain Data

```javascript
// อ่านข้อมูลจาก Ethereum blockchain

const { ethers } = require('ethers');

// Setup provider
const provider = new ethers.JsonRpcProvider(
  `https://eth-mainnet.g.alchemy.com/v2/${process.env.ALCHEMY_KEY}`
);

// 1. Get ETH balance
async function getBalance(address) {
  const balance = await provider.getBalance(address);
  return {
    wei: balance.toString(),
    ether: ethers.formatEther(balance),
  };
}

// 2. Get transaction details
async function getTransaction(txHash) {
  const tx = await provider.getTransaction(txHash);
  const receipt = await provider.getTransactionReceipt(txHash);
  
  return {
    hash: tx.hash,
    from: tx.from,
    to: tx.to,
    value: ethers.formatEther(tx.value),
    gasPrice: ethers.formatUnits(tx.gasPrice, 'gwei'),
    gasUsed: receipt.gasUsed.toString(),
    status: receipt.status === 1 ? 'Success' : 'Failed',
    blockNumber: receipt.blockNumber,
    confirmations: await receipt.confirmations(),
  };
}

// 3. Read ERC-20 token info
const ERC20_ABI = [
  'function name() view returns (string)',
  'function symbol() view returns (string)',
  'function decimals() view returns (uint8)',
  'function totalSupply() view returns (uint256)',
  'function balanceOf(address) view returns (uint256)',
];

async function getTokenInfo(tokenAddress, userAddress) {
  const contract = new ethers.Contract(tokenAddress, ERC20_ABI, provider);
  
  const [name, symbol, decimals, totalSupply, balance] = await Promise.all([
    contract.name(),
    contract.symbol(),
    contract.decimals(),
    contract.totalSupply(),
    userAddress ? contract.balanceOf(userAddress) : 0n,
  ]);
  
  const formatAmount = (amount) =>
    parseFloat(ethers.formatUnits(amount, decimals)).toLocaleString();
  
  return {
    name,
    symbol,
    decimals: Number(decimals),
    totalSupply: formatAmount(totalSupply),
    userBalance: userAddress ? formatAmount(balance) : 'N/A',
  };
}

// 4. Listen for Events
async function listenForTransfers(tokenAddress, userAddress) {
  const contract = new ethers.Contract(
    tokenAddress,
    ['event Transfer(address indexed from, address indexed to, uint256 value)'],
    provider
  );
  
  // Listen for transfers TO user
  contract.on('Transfer', (from, to, value, event) => {
    if (to.toLowerCase() === userAddress.toLowerCase()) {
      console.log(`Received transfer!`);
      console.log(`  From: ${from}`);
      console.log(`  Amount: ${ethers.formatEther(value)} tokens`);
      console.log(`  TX: ${event.log.transactionHash}`);
    }
  });
  
  console.log(`Listening for transfers to ${userAddress}...`);
}

// 5. Query past events (Logs)
async function getPastTransfers(tokenAddress, userAddress, fromBlock = 'earliest') {
  const contract = new ethers.Contract(
    tokenAddress,
    ['event Transfer(address indexed from, address indexed to, uint256 value)'],
    provider
  );
  
  const filter = contract.filters.Transfer(null, userAddress);
  const events = await contract.queryFilter(filter, fromBlock, 'latest');
  
  return events.map(event => ({
    from: event.args.from,
    to: event.args.to,
    value: ethers.formatEther(event.args.value),
    blockNumber: event.blockNumber,
    txHash: event.transactionHash,
  }));
}

// USDC token example (Ethereum mainnet)
const USDC_ADDRESS = '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48';

async function main() {
  // Get USDC info
  const tokenInfo = await getTokenInfo(
    USDC_ADDRESS,
    '0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045' // Vitalik's address
  );
  
  console.log('USDC Token Info:', tokenInfo);
}
```

---

## Step 1881: Sending Transactions

```javascript
// ส่ง transactions ด้วย ethers.js

const { ethers } = require('ethers');

// Setup signer
const provider = new ethers.JsonRpcProvider(
  `https://sepolia.infura.io/v3/${process.env.INFURA_KEY}`
);
const wallet = new ethers.Wallet(process.env.PRIVATE_KEY, provider);

// 1. ส่ง ETH
async function sendETH(toAddress, amountEth) {
  console.log(`Sending ${amountEth} ETH to ${toAddress}...`);
  
  // Check balance first
  const balance = await wallet.provider.getBalance(wallet.address);
  const amount = ethers.parseEther(amountEth.toString());
  
  if (balance < amount) {
    throw new Error(`Insufficient balance: ${ethers.formatEther(balance)} ETH`);
  }
  
  // Estimate gas
  const gasEstimate = await wallet.estimateGas({
    to: toAddress,
    value: amount,
  });
  
  // Get current fee data
  const feeData = await provider.getFeeData();
  
  // Send transaction
  const tx = await wallet.sendTransaction({
    to: toAddress,
    value: amount,
    gasLimit: gasEstimate * 110n / 100n, // add 10% buffer
    maxFeePerGas: feeData.maxFeePerGas,
    maxPriorityFeePerGas: feeData.maxPriorityFeePerGas,
  });
  
  console.log(`Transaction sent: ${tx.hash}`);
  console.log('Waiting for confirmation...');
  
  // Wait for 1 confirmation
  const receipt = await tx.wait(1);
  
  console.log(`Confirmed in block ${receipt.blockNumber}`);
  return receipt;
}

// 2. ส่ง ERC-20 Token
const ERC20_ABI = [
  'function transfer(address to, uint256 amount) returns (bool)',
  'function balanceOf(address) view returns (uint256)',
  'function decimals() view returns (uint8)',
];

async function sendToken(tokenAddress, toAddress, amount) {
  const contract = new ethers.Contract(tokenAddress, ERC20_ABI, wallet);
  
  const decimals = await contract.decimals();
  const amountWei = ethers.parseUnits(amount.toString(), decimals);
  
  // Check balance
  const balance = await contract.balanceOf(wallet.address);
  if (balance < amountWei) {
    throw new Error('Insufficient token balance');
  }
  
  // Send token
  const tx = await contract.transfer(toAddress, amountWei);
  console.log(`Token transfer TX: ${tx.hash}`);
  
  const receipt = await tx.wait();
  return receipt;
}

// 3. Batch transactions ด้วย multicall
async function batchCalls(calls) {
  const MULTICALL_ABI = [
    'function aggregate(tuple(address target, bytes callData)[] calls) returns (uint256 blockNumber, bytes[] returnData)',
  ];
  const MULTICALL_ADDRESS = '0xcA11bde05977b3631167028862bE2a173976CA11';
  
  const multicall = new ethers.Contract(MULTICALL_ADDRESS, MULTICALL_ABI, provider);
  
  const { blockNumber, returnData } = await multicall.aggregate(calls);
  return { blockNumber, returnData };
}

// 4. Transaction with retry
async function sendWithRetry(txFunction, maxRetries = 3) {
  let lastError;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await txFunction();
    } catch (error) {
      lastError = error;
      
      if (error.code === 'REPLACEMENT_UNDERPRICED') {
        // Gas price ต่ำเกินไป ต้องเพิ่ม
        console.log(`Retry ${i + 1}: Increasing gas price...`);
        await new Promise(r => setTimeout(r, 5000)); // wait 5s
        continue;
      }
      
      if (error.code === 'NONCE_EXPIRED') {
        // Nonce ถูกใช้ไปแล้ว
        console.log(`Retry ${i + 1}: Getting new nonce...`);
        continue;
      }
      
      throw error; // ไม่ retry สำหรับ errors อื่น
    }
  }
  
  throw lastError;
}
```

---

## Step 1882: Interacting with Smart Contracts

```javascript
// Interact กับ Smart Contracts

const { ethers } = require('ethers');

// Uniswap V2 Router ABI (บางส่วน)
const UNISWAP_V2_ROUTER_ABI = [
  'function getAmountsOut(uint amountIn, address[] memory path) view returns (uint[] memory amounts)',
  'function swapExactTokensForTokens(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) returns (uint[] memory amounts)',
  'function swapExactETHForTokens(uint amountOutMin, address[] calldata path, address to, uint deadline) payable returns (uint[] memory amounts)',
  'function addLiquidity(address tokenA, address tokenB, uint amountADesired, uint amountBDesired, uint amountAMin, uint amountBMin, address to, uint deadline) returns (uint amountA, uint amountB, uint liquidity)',
];

const UNISWAP_V2_ROUTER = '0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D';
const WETH = '0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2';
const USDC = '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48';

async function getSwapQuote(amountEth) {
  const provider = new ethers.JsonRpcProvider(
    `https://eth-mainnet.g.alchemy.com/v2/${process.env.ALCHEMY_KEY}`
  );
  const router = new ethers.Contract(UNISWAP_V2_ROUTER, UNISWAP_V2_ROUTER_ABI, provider);
  
  const amountIn = ethers.parseEther(amountEth.toString());
  const path = [WETH, USDC];
  
  const amounts = await router.getAmountsOut(amountIn, path);
  
  return {
    amountIn: ethers.formatEther(amounts[0]),
    amountOut: ethers.formatUnits(amounts[1], 6), // USDC has 6 decimals
  };
}

// Deploy Smart Contract
async function deployContract(contractABI, contractBytecode, constructorArgs, signer) {
  const factory = new ethers.ContractFactory(contractABI, contractBytecode, signer);
  
  console.log('Deploying contract...');
  const contract = await factory.deploy(...constructorArgs);
  
  console.log(`Deployment TX: ${contract.deploymentTransaction().hash}`);
  console.log('Waiting for deployment...');
  
  await contract.waitForDeployment();
  
  const address = await contract.getAddress();
  console.log(`Contract deployed at: ${address}`);
  
  return contract;
}

// Complete DApp interaction example
async function dappExample() {
  const provider = new ethers.JsonRpcProvider(
    `https://sepolia.infura.io/v3/${process.env.INFURA_KEY}`
  );
  const wallet = new ethers.Wallet(process.env.PRIVATE_KEY, provider);

  // Simple storage ABI
  const STORAGE_ABI = [
    'function store(uint256 value)',
    'function retrieve() view returns (uint256)',
    'event ValueChanged(uint256 oldValue, uint256 newValue)',
  ];

  // Deploy (ต้องมี bytecode)
  // const contract = await deployContract(STORAGE_ABI, BYTECODE, [], wallet);

  // หรือใช้ existing contract
  const contractAddress = '0x...';
  const contract = new ethers.Contract(contractAddress, STORAGE_ABI, wallet);
  
  // Read
  const currentValue = await contract.retrieve();
  console.log('Current value:', currentValue.toString());
  
  // Write (ต้องเสีย gas)
  const tx = await contract.store(42);
  await tx.wait();
  console.log('Stored 42!');
  
  // Listen for events
  contract.on('ValueChanged', (oldVal, newVal) => {
    console.log(`Value changed: ${oldVal} -> ${newVal}`);
  });
  
  // Store another value
  const tx2 = await contract.store(100);
  await tx2.wait();
}
```

---

## Step 1883: MetaMask Integration

```javascript
// MetaMask Integration ใน Browser

// 1. ตรวจสอบและเชื่อมต่อ MetaMask
async function connectMetaMask() {
  if (typeof window.ethereum === 'undefined') {
    throw new Error('MetaMask ไม่ได้ติดตั้ง! ดาวน์โหลดที่ metamask.io');
  }
  
  // Request access
  const accounts = await window.ethereum.request({
    method: 'eth_requestAccounts',
  });
  
  return accounts[0];
}

// 2. Get current network
async function getCurrentNetwork() {
  const chainId = await window.ethereum.request({ method: 'eth_chainId' });
  
  const networks = {
    '0x1': 'Ethereum Mainnet',
    '0xaa36a7': 'Sepolia Testnet',
    '0x89': 'Polygon',
    '0xa': 'Optimism',
    '0xa4b1': 'Arbitrum One',
    '0x38': 'BNB Smart Chain',
  };
  
  return {
    chainId,
    name: networks[chainId] || `Unknown (${chainId})`,
  };
}

// 3. Listen for account/network changes
function setupMetaMaskListeners(onAccountChange, onNetworkChange) {
  window.ethereum.on('accountsChanged', (accounts) => {
    if (accounts.length === 0) {
      onAccountChange(null);
    } else {
      onAccountChange(accounts[0]);
    }
  });
  
  window.ethereum.on('chainChanged', (chainId) => {
    onNetworkChange(chainId);
    window.location.reload(); // แนะนำให้ reload
  });
  
  window.ethereum.on('disconnect', () => {
    onAccountChange(null);
  });
}

// 4. Switch network
async function switchToNetwork(chainId) {
  try {
    await window.ethereum.request({
      method: 'wallet_switchEthereumChain',
      params: [{ chainId }],
    });
  } catch (error) {
    if (error.code === 4902) {
      // Network ไม่มีใน MetaMask ต้อง add
      throw new Error('Network not found in MetaMask');
    }
    throw error;
  }
}

// 5. Add custom network
async function addPolygonNetwork() {
  await window.ethereum.request({
    method: 'wallet_addEthereumChain',
    params: [{
      chainId: '0x89',
      chainName: 'Polygon Mainnet',
      nativeCurrency: {
        name: 'MATIC',
        symbol: 'MATIC',
        decimals: 18,
      },
      rpcUrls: ['https://polygon-rpc.com/'],
      blockExplorerUrls: ['https://polygonscan.com/'],
    }],
  });
}

// 6. Sign transaction with MetaMask
async function sendWithMetaMask(toAddress, amountEth) {
  const provider = new ethers.BrowserProvider(window.ethereum);
  const signer = await provider.getSigner();
  
  const tx = await signer.sendTransaction({
    to: toAddress,
    value: ethers.parseEther(amountEth),
  });
  
  console.log('TX hash:', tx.hash);
  const receipt = await tx.wait();
  return receipt;
}

// 7. Complete Web3 Connection Button (React example)
function Web3Connect() {
  const [account, setAccount] = React.useState(null);
  const [network, setNetwork] = React.useState(null);
  const [loading, setLoading] = React.useState(false);
  const [error, setError] = React.useState(null);
  
  const connect = async () => {
    setLoading(true);
    setError(null);
    try {
      const addr = await connectMetaMask();
      const net = await getCurrentNetwork();
      setAccount(addr);
      setNetwork(net);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  React.useEffect(() => {
    if (account) {
      setupMetaMaskListeners(
        (newAccount) => setAccount(newAccount),
        (chainId) => setNetwork({ chainId })
      );
    }
  }, [account]);
  
  if (loading) return React.createElement('button', null, 'Connecting...');
  
  if (account) {
    return React.createElement('div', null,
      React.createElement('p', null, `Connected: ${account.slice(0, 6)}...${account.slice(-4)}`),
      React.createElement('p', null, `Network: ${network?.name}`),
    );
  }
  
  return React.createElement('button', { onClick: connect }, 'Connect MetaMask');
}
```

---

## Step 1884: Building a Simple DApp

```javascript
// Simple Token DApp ด้วย Vanilla JavaScript

// index.html
const dappHTML = `
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <title>Simple Token DApp</title>
  <script src="https://cdn.jsdelivr.net/npm/ethers@6.7.0/dist/ethers.umd.min.js"></script>
  <style>
    body { font-family: Arial; max-width: 600px; margin: 50px auto; padding: 20px; }
    button { padding: 10px 20px; cursor: pointer; margin: 5px; }
    .card { border: 1px solid #ddd; padding: 20px; margin: 10px 0; border-radius: 8px; }
    .success { color: green; }
    .error { color: red; }
  </style>
</head>
<body>
  <h1>Simple ERC-20 DApp</h1>
  
  <div class="card">
    <button id="connectBtn">เชื่อมต่อ MetaMask</button>
    <p id="accountInfo"></p>
    <p id="networkInfo"></p>
  </div>
  
  <div class="card" id="tokenCard" style="display:none">
    <h2>Token Info</h2>
    <p>Token: <span id="tokenName">-</span></p>
    <p>Your Balance: <span id="balance">-</span></p>
    
    <h3>Transfer Tokens</h3>
    <input id="toAddress" placeholder="Recipient address" style="width:300px">
    <input id="amount" type="number" placeholder="Amount" style="width:100px">
    <button id="sendBtn">Send</button>
    <p id="txStatus"></p>
  </div>
  
  <script>
    const TOKEN_ADDRESS = '0x...'; // Your token address
    const TOKEN_ABI = [
      'function name() view returns (string)',
      'function symbol() view returns (string)',
      'function decimals() view returns (uint8)',
      'function balanceOf(address) view returns (uint256)',
      'function transfer(address to, uint256 amount) returns (bool)',
      'event Transfer(address indexed from, address indexed to, uint256 value)',
    ];
    
    let provider, signer, contract, userAddress;
    
    // Connect
    document.getElementById('connectBtn').onclick = async () => {
      try {
        provider = new ethers.BrowserProvider(window.ethereum);
        await provider.send('eth_requestAccounts', []);
        signer = await provider.getSigner();
        userAddress = await signer.getAddress();
        
        document.getElementById('accountInfo').textContent = 
          'Account: ' + userAddress.slice(0, 10) + '...';
        
        const network = await provider.getNetwork();
        document.getElementById('networkInfo').textContent =
          'Network: ' + network.name + ' (Chain ID: ' + network.chainId + ')';
        
        // Load token
        contract = new ethers.Contract(TOKEN_ADDRESS, TOKEN_ABI, provider);
        const [name, symbol, decimals, balance] = await Promise.all([
          contract.name(),
          contract.symbol(),
          contract.decimals(),
          contract.balanceOf(userAddress),
        ]);
        
        document.getElementById('tokenName').textContent = name + ' (' + symbol + ')';
        document.getElementById('balance').textContent = 
          ethers.formatUnits(balance, decimals) + ' ' + symbol;
        
        document.getElementById('tokenCard').style.display = 'block';
        
        // Listen for Transfer events
        contract.on('Transfer', (from, to, value) => {
          if (to.toLowerCase() === userAddress.toLowerCase()) {
            updateBalance();
          }
        });
        
      } catch (e) {
        console.error(e);
        document.getElementById('accountInfo').textContent = 'Error: ' + e.message;
      }
    };
    
    // Send tokens
    document.getElementById('sendBtn').onclick = async () => {
      const to = document.getElementById('toAddress').value;
      const amount = document.getElementById('amount').value;
      const status = document.getElementById('txStatus');
      
      try {
        status.textContent = 'Sending...';
        status.className = '';
        
        const connectedContract = contract.connect(signer);
        const decimals = await contract.decimals();
        const amountWei = ethers.parseUnits(amount, decimals);
        
        const tx = await connectedContract.transfer(to, amountWei);
        status.textContent = 'TX submitted: ' + tx.hash.slice(0, 10) + '...';
        
        await tx.wait();
        status.textContent = 'Transaction confirmed! ✅';
        status.className = 'success';
        
        updateBalance();
      } catch (e) {
        status.textContent = 'Error: ' + e.message;
        status.className = 'error';
      }
    };
    
    async function updateBalance() {
      const balance = await contract.balanceOf(userAddress);
      const decimals = await contract.decimals();
      const symbol = await contract.symbol();
      document.getElementById('balance').textContent = 
        ethers.formatUnits(balance, decimals) + ' ' + symbol;
    }
  </script>
</body>
</html>
`;
```

---

## Step 1885: NFT Concepts

```javascript
// NFT (Non-Fungible Token) Concepts

// ERC-721 = standard สำหรับ NFT
// แต่ละ token มี unique ID และ owner

// Metadata structure สำหรับ NFT
const nftMetadata = {
  name: "Cool NFT #42",
  description: "This is a unique digital collectible",
  image: "ipfs://QmHash.../42.png",
  external_url: "https://mynft.io/token/42",
  attributes: [
    { trait_type: "Background", value: "Blue" },
    { trait_type: "Eyes", value: "Laser" },
    { trait_type: "Rarity", value: "Legendary" },
    { trait_type: "Level", display_type: "number", value: 5 },
  ],
};

// ERC-721 interactions ด้วย ethers.js
const ERC721_ABI = [
  'function name() view returns (string)',
  'function symbol() view returns (string)',
  'function ownerOf(uint256 tokenId) view returns (address)',
  'function tokenURI(uint256 tokenId) view returns (string)',
  'function balanceOf(address owner) view returns (uint256)',
  'function tokenOfOwnerByIndex(address owner, uint256 index) view returns (uint256)',
  'function transferFrom(address from, address to, uint256 tokenId)',
  'function approve(address to, uint256 tokenId)',
  'event Transfer(address indexed from, address indexed to, uint256 indexed tokenId)',
];

async function getNFTsForAddress(contractAddress, ownerAddress, provider) {
  const contract = new ethers.Contract(contractAddress, ERC721_ABI, provider);
  
  const balance = await contract.balanceOf(ownerAddress);
  const tokenIds = [];
  
  for (let i = 0; i < Number(balance); i++) {
    const tokenId = await contract.tokenOfOwnerByIndex(ownerAddress, i);
    tokenIds.push(Number(tokenId));
  }
  
  // Get metadata for each token
  const nfts = await Promise.all(
    tokenIds.map(async (tokenId) => {
      const tokenURI = await contract.tokenURI(tokenId);
      
      // Fetch metadata (IPFS or HTTP)
      const metadata = await fetchMetadata(tokenURI);
      
      return {
        tokenId,
        tokenURI,
        metadata,
      };
    })
  );
  
  return nfts;
}

async function fetchMetadata(uri) {
  // Convert IPFS URI
  if (uri.startsWith('ipfs://')) {
    const ipfsHash = uri.replace('ipfs://', '');
    uri = `https://ipfs.io/ipfs/${ipfsHash}`;
  }
  
  const response = await fetch(uri);
  return response.json();
}

// ERC-1155 Multi Token Standard (ใช้ได้ทั้ง fungible และ non-fungible)
const ERC1155_ABI = [
  'function balanceOf(address account, uint256 id) view returns (uint256)',
  'function balanceOfBatch(address[] accounts, uint256[] ids) view returns (uint256[])',
  'function uri(uint256 id) view returns (string)',
  'function safeTransferFrom(address from, address to, uint256 id, uint256 amount, bytes data)',
];

// ใช้กับ game items:
// tokenId 1 = Sword (fungible - มีหลายอัน)
// tokenId 2 = Rare Shield (semi-fungible - มีจำกัด)
// tokenId 3 = Legendary Dragon (non-fungible - อันเดียวในโลก)
```

---

## Step 1886: DeFi Concepts

```javascript
// DeFi (Decentralized Finance) Core Concepts

// 1. AMM (Automated Market Maker) - Uniswap
// ราคาถูกกำหนดโดย x * y = k (constant product formula)

function calculateAMMPrice(reserveA, reserveB, amountIn) {
  // x * y = k
  const k = reserveA * reserveB;
  const newReserveA = reserveA + amountIn;
  const newReserveB = k / newReserveA;
  const amountOut = reserveB - newReserveB;
  
  // Price impact
  const priceImpact = (amountOut / reserveB) * 100;
  
  return {
    amountOut: amountOut.toFixed(6),
    priceImpact: priceImpact.toFixed(2) + '%',
    newReserveA: newReserveA.toFixed(2),
    newReserveB: newReserveB.toFixed(2),
  };
}

// ตัวอย่าง ETH/USDC pool
const ethReserve = 1000;  // 1000 ETH
const usdcReserve = 2000000;  // 2,000,000 USDC (ราคา ETH = $2000)

console.log('Swap 10 ETH:');
console.log(calculateAMMPrice(ethReserve, usdcReserve, 10));

// 2. Liquidity Provision
// LP (Liquidity Provider) เพิ่ม tokens ทั้งสองฝั่งในอัตราส่วน

function calculateLPTokens(reserveA, reserveB, totalLP, addA, addB) {
  const lpFromA = (addA / reserveA) * totalLP;
  const lpFromB = (addB / reserveB) * totalLP;
  const lpTokens = Math.min(lpFromA, lpFromB);
  
  return {
    lpTokens,
    sharePercent: (lpTokens / (totalLP + lpTokens)) * 100,
  };
}

// 3. Yield Farming / Liquidity Mining
const yieldExample = {
  pool: 'ETH/USDC',
  tvl: 10000000, // Total Value Locked
  rewardPerBlock: 1, // tokens per block
  blocksPerDay: 6500,
  
  // APR calculation
  calculateAPR(tvl, rewardPerBlock, blocksPerDay, rewardPriceUSD) {
    const yearlyRewards = rewardPerBlock * blocksPerDay * 365 * rewardPriceUSD;
    return (yearlyRewards / tvl) * 100;
  },
};

// 4. Flash Loans
// กู้ tokens โดยไม่ต้องมี collateral แต่ต้อง return ใน transaction เดียว!
const flashLoanConcept = `
Flash Loan Process:
1. กู้ 1000 ETH จาก Aave
2. ใช้ ETH ทำ arbitrage
   - ซื้อ token X ใน exchange A (ราคาต่ำ)
   - ขาย token X ใน exchange B (ราคาสูง)
3. ได้กำไร
4. คืน 1000 ETH + fee ให้ Aave
5. เก็บกำไรไว้

ถ้าไม่คืนภายใน transaction เดียว -> transaction revert
`;

// 5. DeFi Risks
const defiRisks = {
  smartContractBugs: 'Code มีช่องโหว่ถูก exploit',
  impermanentLoss: 'LP เสีย value เมื่อ price ratio เปลี่ยน',
  rug_pull: 'Dev drain liquidity',
  flashLoanAttacks: 'ใช้ flash loan manipulate prices',
  frontRunning: 'MEV bots ดัก transactions',
};
```

---

## Step 1887: IPFS for Decentralized Storage

```javascript
// IPFS (InterPlanetary File System)
// npm install ipfs-http-client

const { create } = require('ipfs-http-client');

// เชื่อมต่อกับ Infura IPFS
const ipfs = create({
  host: 'ipfs.infura.io',
  port: 5001,
  protocol: 'https',
  headers: {
    authorization: 'Basic ' + Buffer.from(
      `${process.env.INFURA_IPFS_ID}:${process.env.INFURA_IPFS_SECRET}`
    ).toString('base64'),
  },
});

// 1. Upload file ไปยัง IPFS
async function uploadToIPFS(content) {
  const buffer = typeof content === 'string'
    ? Buffer.from(content)
    : content;
  
  const result = await ipfs.add(buffer);
  return `ipfs://${result.cid}`;
}

// 2. Upload JSON metadata
async function uploadMetadata(metadata) {
  const json = JSON.stringify(metadata);
  return uploadToIPFS(json);
}

// 3. Upload image
async function uploadImage(imagePath) {
  const fs = require('fs');
  const imageBuffer = fs.readFileSync(imagePath);
  return uploadToIPFS(imageBuffer);
}

// 4. Pin content (ป้องกัน garbage collection)
async function pinContent(cid) {
  await ipfs.pin.add(cid);
  console.log(`Pinned: ${cid}`);
}

// 5. Retrieve content
async function getFromIPFS(cid) {
  const chunks = [];
  for await (const chunk of ipfs.cat(cid)) {
    chunks.push(chunk);
  }
  return Buffer.concat(chunks).toString();
}

// NFT Upload Workflow
async function mintNFTWithIPFS(imageFile, nftData) {
  console.log('1. Uploading image to IPFS...');
  const imageURI = await uploadImage(imageFile);
  console.log(`   Image: ${imageURI}`);
  
  console.log('2. Creating metadata...');
  const metadata = {
    name: nftData.name,
    description: nftData.description,
    image: imageURI,
    attributes: nftData.attributes,
    external_url: `https://mynft.io/token/${nftData.tokenId}`,
  };
  
  console.log('3. Uploading metadata to IPFS...');
  const metadataURI = await uploadMetadata(metadata);
  console.log(`   Metadata: ${metadataURI}`);
  
  console.log('4. Minting NFT on blockchain...');
  // ส่ง transaction ด้วย metadataURI เป็น tokenURI
  
  return metadataURI;
}

// Pinata (popular IPFS pinning service)
async function uploadToPinata(content, filename) {
  const FormData = require('form-data');
  const axios = require('axios');
  
  const formData = new FormData();
  formData.append('file', Buffer.from(content), filename);
  
  const response = await axios.post(
    'https://api.pinata.cloud/pinning/pinFileToIPFS',
    formData,
    {
      headers: {
        ...formData.getHeaders(),
        Authorization: `Bearer ${process.env.PINATA_JWT}`,
      },
    }
  );
  
  return `ipfs://${response.data.IpfsHash}`;
}

// ตรวจสอบ IPFS content
async function verifyIPFSContent(uri) {
  const cid = uri.replace('ipfs://', '');
  
  // ดึงผ่าน public gateway
  const gateways = [
    `https://ipfs.io/ipfs/${cid}`,
    `https://cloudflare-ipfs.com/ipfs/${cid}`,
    `https://gateway.pinata.cloud/ipfs/${cid}`,
  ];
  
  for (const gateway of gateways) {
    try {
      const response = await fetch(gateway, { timeout: 5000 });
      if (response.ok) {
        return { accessible: true, gateway };
      }
    } catch {}
  }
  
  return { accessible: false };
}
```

---

## Step 1888: Hardhat Development Framework

```javascript
// Hardhat - Ethereum development environment
// npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox

// hardhat.config.js
const hardhatConfig = `
require("@nomicfoundation/hardhat-toolbox");

/** @type import('hardhat/config').HardhatUserConfig */
module.exports = {
  solidity: {
    version: "0.8.19",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200,
      },
    },
  },
  networks: {
    hardhat: {
      chainId: 1337,
    },
    sepolia: {
      url: process.env.SEPOLIA_URL,
      accounts: [process.env.PRIVATE_KEY],
    },
    mainnet: {
      url: process.env.MAINNET_URL,
      accounts: [process.env.PRIVATE_KEY],
    },
  },
  etherscan: {
    apiKey: process.env.ETHERSCAN_KEY,
  },
  gasReporter: {
    enabled: true,
    currency: "USD",
    coinmarketcap: process.env.CMC_KEY,
  },
};
`;

// Deploy script - scripts/deploy.js
const deployScript = `
const { ethers } = require("hardhat");

async function main() {
  const [deployer] = await ethers.getSigners();
  console.log("Deploying with:", deployer.address);
  console.log("Balance:", ethers.formatEther(await deployer.provider.getBalance(deployer.address)));
  
  // Deploy MyToken
  const MyToken = await ethers.getContractFactory("MyToken");
  const token = await MyToken.deploy("My Token", "MTK");
  await token.waitForDeployment();
  
  const address = await token.getAddress();
  console.log("MyToken deployed to:", address);
  
  // Verify on Etherscan
  if (process.env.ETHERSCAN_KEY) {
    console.log("Waiting for block confirmations...");
    await token.deploymentTransaction().wait(5);
    
    await hre.run("verify:verify", {
      address,
      constructorArguments: ["My Token", "MTK"],
    });
  }
}

main().catch(console.error);
`;

// Test file - test/MyToken.test.js
const testFile = `
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("MyToken", function() {
  let token, owner, addr1, addr2;
  
  beforeEach(async function() {
    [owner, addr1, addr2] = await ethers.getSigners();
    
    const MyToken = await ethers.getContractFactory("MyToken");
    token = await MyToken.deploy("My Token", "MTK");
  });
  
  describe("Deployment", function() {
    it("Should set the correct name and symbol", async function() {
      expect(await token.name()).to.equal("My Token");
      expect(await token.symbol()).to.equal("MTK");
    });
    
    it("Should assign total supply to owner", async function() {
      const ownerBalance = await token.balanceOf(owner.address);
      expect(await token.totalSupply()).to.equal(ownerBalance);
    });
  });
  
  describe("Transfers", function() {
    it("Should transfer tokens correctly", async function() {
      const amount = ethers.parseEther("50");
      
      await token.transfer(addr1.address, amount);
      
      expect(await token.balanceOf(addr1.address)).to.equal(amount);
    });
    
    it("Should fail if sender doesn't have enough tokens", async function() {
      const balance = await token.balanceOf(addr1.address);
      
      await expect(
        token.connect(addr1).transfer(addr2.address, balance + 1n)
      ).to.be.revertedWith("Insufficient balance");
    });
    
    it("Should emit Transfer event", async function() {
      const amount = ethers.parseEther("10");
      
      await expect(token.transfer(addr1.address, amount))
        .to.emit(token, "Transfer")
        .withArgs(owner.address, addr1.address, amount);
    });
  });
});
`;

// Hardhat commands
const hardhatCommands = `
# Compile
npx hardhat compile

# Run tests
npx hardhat test
npx hardhat test --parallel

# Deploy to local
npx hardhat run scripts/deploy.js

# Deploy to testnet
npx hardhat run scripts/deploy.js --network sepolia

# Verify on Etherscan
npx hardhat verify --network sepolia <CONTRACT_ADDRESS> "My Token" "MTK"

# Start local node
npx hardhat node

# Open console
npx hardhat console --network localhost

# Gas report
REPORT_GAS=true npx hardhat test

# Coverage
npx hardhat coverage
`;
```

---

## Step 1889: Web3.js Overview

```javascript
// Web3.js - อีก library สำหรับ Ethereum
// npm install web3

const Web3 = require('web3');

// เชื่อมต่อ
const web3 = new Web3(
  new Web3.providers.HttpProvider(
    `https://mainnet.infura.io/v3/${process.env.INFURA_KEY}`
  )
);

// หรือ MetaMask
// const web3 = new Web3(window.ethereum);

// Get balance
async function getBalance(address) {
  const balanceWei = await web3.eth.getBalance(address);
  return web3.utils.fromWei(balanceWei, 'ether');
}

// Contract interaction
const contractABI = [/* ... */];
const contract = new web3.eth.Contract(contractABI, contractAddress);

// Read
const result = await contract.methods.myMethod(arg1, arg2).call();

// Write (ต้อง sign)
const tx = contract.methods.myMethod(arg1, arg2);
const gas = await tx.estimateGas({ from: userAddress });
const receipt = await tx.send({ from: userAddress, gas });

// Encode/decode
const encoded = web3.eth.abi.encodeFunctionCall(
  { name: 'transfer', inputs: [{ type: 'address' }, { type: 'uint256' }] },
  [toAddress, amountWei]
);

// Utils
web3.utils.toWei('1', 'ether');  // '1000000000000000000'
web3.utils.fromWei('1000000000000000000', 'ether');  // '1'
web3.utils.toChecksumAddress('0xabc...');  // Checksum address
web3.utils.isAddress('0xabc...');  // true/false
web3.utils.sha3('hello');  // keccak256 hash

// ethers.js vs web3.js
const comparison = {
  ethersjs: {
    size: '~80KB minified',
    ens: 'Built-in ENS support',
    provider: 'Advanced provider system',
    signer: 'Clear signer/provider separation',
    typescript: 'Excellent TypeScript support',
    maintenance: 'Actively maintained',
  },
  web3js: {
    size: '~500KB minified',
    history: 'Older, more established',
    modules: 'Modular architecture',
    typescript: 'TypeScript support',
    maintenance: 'Maintained by ChainSafe',
  },
  recommendation: 'ethers.js is generally recommended for new projects',
};
```

---

## Step 1890: Complete DApp Project Structure

```javascript
// โครงสร้างโปรเจค DApp ที่สมบูรณ์

/*
my-dapp/
├── contracts/                 <- Solidity contracts
│   ├── Token.sol
│   ├── NFT.sol
│   └── Marketplace.sol
├── scripts/                   <- Deployment scripts
│   ├── deploy.js
│   └── verify.js
├── test/                      <- Contract tests
│   ├── Token.test.js
│   └── NFT.test.js
├── frontend/                  <- React frontend
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   │   ├── ConnectWallet.jsx
│   │   │   ├── TokenBalance.jsx
│   │   │   └── NFTGallery.jsx
│   │   ├── hooks/
│   │   │   ├── useWeb3.js
│   │   │   ├── useContract.js
│   │   │   └── useNFTs.js
│   │   ├── utils/
│   │   │   ├── contractAddresses.js
│   │   │   └── abis.js
│   │   └── App.jsx
│   └── package.json
├── hardhat.config.js
└── package.json
*/

// hooks/useWeb3.js
function useWeb3() {
  const [provider, setProvider] = React.useState(null);
  const [signer, setSigner] = React.useState(null);
  const [account, setAccount] = React.useState(null);
  const [chainId, setChainId] = React.useState(null);
  const [balance, setBalance] = React.useState('0');
  const [connecting, setConnecting] = React.useState(false);
  const [error, setError] = React.useState(null);

  const connect = React.useCallback(async () => {
    if (!window.ethereum) {
      setError('Please install MetaMask');
      return;
    }
    
    setConnecting(true);
    setError(null);
    
    try {
      const web3Provider = new ethers.BrowserProvider(window.ethereum);
      await web3Provider.send('eth_requestAccounts', []);
      
      const web3Signer = await web3Provider.getSigner();
      const address = await web3Signer.getAddress();
      const network = await web3Provider.getNetwork();
      const bal = await web3Provider.getBalance(address);
      
      setProvider(web3Provider);
      setSigner(web3Signer);
      setAccount(address);
      setChainId(Number(network.chainId));
      setBalance(ethers.formatEther(bal));
      
    } catch (err) {
      setError(err.message);
    } finally {
      setConnecting(false);
    }
  }, []);

  const disconnect = React.useCallback(() => {
    setProvider(null);
    setSigner(null);
    setAccount(null);
    setChainId(null);
    setBalance('0');
  }, []);

  React.useEffect(() => {
    if (!window.ethereum) return;
    
    const handleAccountChange = (accounts) => {
      if (accounts.length === 0) disconnect();
      else connect();
    };
    
    const handleChainChange = () => {
      window.location.reload();
    };
    
    window.ethereum.on('accountsChanged', handleAccountChange);
    window.ethereum.on('chainChanged', handleChainChange);
    
    return () => {
      window.ethereum.removeListener('accountsChanged', handleAccountChange);
      window.ethereum.removeListener('chainChanged', handleChainChange);
    };
  }, [connect, disconnect]);

  return { provider, signer, account, chainId, balance, connecting, error, connect, disconnect };
}

// hooks/useContract.js
function useContract(address, abi) {
  const { provider, signer } = useWeb3();
  
  const readContract = React.useMemo(() => {
    if (!provider || !address) return null;
    return new ethers.Contract(address, abi, provider);
  }, [provider, address, abi]);
  
  const writeContract = React.useMemo(() => {
    if (!signer || !address) return null;
    return new ethers.Contract(address, abi, signer);
  }, [signer, address, abi]);
  
  return { readContract, writeContract };
}

// blockchain security best practices
const blockchainSecurity = {
  neverExposePlrivatKey: 'ไม่ commit private key ลง git เด็ดขาด',
  useEnvVars: 'ใช้ environment variables เสมอ',
  auditContracts: 'Audit smart contracts ก่อน deploy mainnet',
  testOnTestnet: 'ทดสอบบน testnet ก่อนเสมอ',
  useMultiSig: 'ใช้ multisig สำหรับ treasury',
  timelock: 'ใช้ timelock สำหรับ upgrades',
  monitoring: 'Monitor events และ transactions',
};

console.log('Blockchain Security:', blockchainSecurity);
```

---

## แบบฝึกหัด (Exercises)

### Exercise 1: Implement Complete Blockchain
สร้าง blockchain ที่ complete ด้วย:
- Proof of Work
- Transaction validation
- Wallet system (key pairs)
- Digital signatures สำหรับ transactions
- P2P network simulation (2+ nodes)
- Consensus algorithm

### Exercise 2: Build NFT Contract
สร้าง ERC-721 contract ด้วย Hardhat:
- Mint function (with IPFS metadata)
- Burn function
- Royalty support (ERC-2981)
- Whitelist minting
- Unit tests > 90% coverage
- Deploy และ verify บน Sepolia

### Exercise 3: Simple DeFi - AMM
สร้าง Simple AMM DEX:
- Smart contract ด้วย x*y=k formula
- Liquidity provision
- Token swaps
- LP tokens
- Frontend สำหรับ interact
- Deploy บน testnet

### Exercise 4: MetaMask DApp
สร้าง DApp ที่:
- Connect MetaMask
- Show ETH balance และ token balances
- Send ETH และ ERC-20 tokens
- Show transaction history
- Handle network switching

### Exercise 5: IPFS + NFT Workflow
สร้าง workflow สำหรับ:
- Upload image ไปยัง IPFS
- สร้าง metadata JSON
- Upload metadata ไปยัง IPFS
- Mint NFT ด้วย IPFS URI
- Display NFT ใน DApp

---

## ทรัพยากรเพิ่มเติม

```javascript
const additionalResources = {
  documentation: {
    ethereum: 'https://ethereum.org/developers',
    ethersjs: 'https://docs.ethers.org/v6/',
    hardhat: 'https://hardhat.org/docs',
    openzeppelin: 'https://docs.openzeppelin.com',
    solidity: 'https://docs.soliditylang.org',
  },
  
  testing: {
    sepolia: 'https://sepoliafaucet.com (testnet ETH)',
    alchemy: 'https://www.alchemy.com (free tier)',
    infura: 'https://infura.io (free tier)',
  },
  
  security: {
    slither: 'Static analysis tool สำหรับ Solidity',
    mythril: 'Security analysis tool',
    consensys_diligence: 'Professional audit tools',
    owasp: 'OWASP Smart Contract Top 10',
  },
  
  learning: {
    cryptozombies: 'https://cryptozombies.io - เรียน Solidity แบบ interactive',
    speedrunethereum: 'https://speedrunethereum.com - challenges',
    buildspace: 'https://buildspace.so - project-based learning',
    alchemy_university: 'https://university.alchemy.com - free courses',
  },
};
```

---

*จบ Part 95: Blockchain กับ JavaScript*
*จบ Steps 1871-1890*

## สรุป Parts 91-95

| Part | หัวข้อ | Steps |
|------|--------|-------|
| 91 | Code Architecture Patterns | 1791-1810 |
| 92 | Open Source Contribution | 1811-1830 |
| 93 | Performance at Scale | 1831-1850 |
| 94 | Advanced Security | 1851-1870 |
| 95 | Blockchain กับ JavaScript | 1871-1890 |

ยินดีด้วย! คุณได้เรียนรู้ JavaScript ในระดับ Advanced/Expert แล้ว! 🎉
