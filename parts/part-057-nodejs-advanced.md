# Part 57: Node.js ขั้นสูง (Steps 1111-1130)

## บทนำ

ในบทนี้เราจะเรียนรู้คุณสมบัติขั้นสูงของ Node.js ที่ช่วยให้สามารถสร้างแอปพลิเคชันที่มีประสิทธิภาพสูง รองรับข้อมูลขนาดใหญ่ และทำงานแบบ concurrent ได้อย่างมีประสิทธิภาพ

---

## Step 1111: Streams - ทำความรู้จัก

Streams คือ abstract interface สำหรับทำงานกับ streaming data ใน Node.js ช่วยให้สามารถประมวลผลข้อมูลขนาดใหญ่ได้โดยไม่ต้องโหลดทั้งหมดเข้า memory พร้อมกัน

### ประเภทของ Streams

```javascript
// 4 ประเภทหลัก:
// 1. Readable - อ่านข้อมูล (เช่น fs.createReadStream)
// 2. Writable - เขียนข้อมูล (เช่น fs.createWriteStream)
// 3. Transform - แปลงข้อมูลระหว่างอ่าน/เขียน
// 4. Duplex - ทั้ง readable และ writable

const fs = require('fs');

// ===== ทำไม Streams ดีกว่า readFile? =====

// วิธีเดิม: โหลดไฟล์ทั้งหมดเข้า memory
fs.readFile('./bigFile.txt', (err, data) => {
  if (err) throw err;
  console.log(data.toString());
  // ปัญหา: ถ้าไฟล์ 1GB จะใช้ RAM 1GB+
});

// วิธี Stream: อ่านทีละ chunk (default 64KB)
const readStream = fs.createReadStream('./bigFile.txt');
readStream.on('data', (chunk) => {
  // chunk คือ Buffer หรือ string
  process.stdout.write(chunk);
  // ใช้ RAM น้อยมาก!
});

readStream.on('end', () => {
  console.log('\nFinished reading');
});

readStream.on('error', (err) => {
  console.error('Error:', err.message);
});
```

---

## Step 1112: Readable Streams

```javascript
const fs = require('fs');
const { Readable } = require('stream');

// ===== fs.createReadStream =====
const fileStream = fs.createReadStream('./data.txt', {
  encoding: 'utf8',       // string mode
  highWaterMark: 16384,   // chunk size (bytes), default 64KB
  start: 0,               // start byte
  end: 1000               // end byte
});

// Events
fileStream.on('data', (chunk) => {
  console.log(`Chunk size: ${chunk.length} bytes`);
});

fileStream.on('end', () => {
  console.log('Stream ended');
});

fileStream.on('error', (err) => {
  console.error('Stream error:', err);
});

fileStream.on('close', () => {
  console.log('Stream closed');
});

// ===== Pause/Resume =====
fileStream.pause();
fileStream.resume();

// ===== Readable.from() - สร้างจาก array หรือ iterable =====
const arrayStream = Readable.from([1, 2, 3, 4, 5]);

arrayStream.on('data', (item) => {
  console.log('Item:', item);
});

// สร้างจาก async generator
async function* generateNumbers() {
  for (let i = 1; i <= 5; i++) {
    await new Promise(resolve => setTimeout(resolve, 100));
    yield i;
  }
}

const generatorStream = Readable.from(generateNumbers());

generatorStream.on('data', (num) => {
  console.log('Number:', num);
});

// ===== Custom Readable Stream =====
class CounterStream extends Readable {
  constructor(max, options = {}) {
    super(options);
    this.current = 1;
    this.max = max;
  }

  _read() {
    if (this.current <= this.max) {
      this.push(Buffer.from(this.current.toString() + '\n'));
      this.current++;
    } else {
      this.push(null); // สัญญาณ end of stream
    }
  }
}

const counter = new CounterStream(5);
counter.on('data', (chunk) => {
  process.stdout.write(chunk.toString());
});
counter.on('end', () => console.log('Count done!'));

// ===== Consuming streams with for...of =====
async function readLargeFile() {
  const stream = fs.createReadStream('./large.txt', { encoding: 'utf8' });
  
  let totalChars = 0;
  for await (const chunk of stream) {
    totalChars += chunk.length;
    // process chunk
  }
  
  console.log('Total characters:', totalChars);
}

readLargeFile();
```

---

## Step 1113: Writable Streams

```javascript
const fs = require('fs');
const { Writable } = require('stream');

// ===== fs.createWriteStream =====
const writeStream = fs.createWriteStream('./output.txt', {
  encoding: 'utf8',
  flags: 'w'   // 'w' = overwrite, 'a' = append
});

// เขียนข้อมูล
writeStream.write('Line 1\n');
writeStream.write('Line 2\n');
writeStream.write(Buffer.from('Line 3\n'));

// ปิด stream
writeStream.end('Final line\n', () => {
  console.log('File write complete');
});

// Events
writeStream.on('drain', () => {
  console.log('Buffer drained, safe to write more');
});

writeStream.on('finish', () => {
  console.log('Stream finished');
});

writeStream.on('error', (err) => {
  console.error('Write error:', err);
});

// ===== backpressure handling =====
const output = fs.createWriteStream('./big-output.txt');

async function writeLargeData() {
  for (let i = 0; i < 1000000; i++) {
    const data = `Line ${i}: ${'x'.repeat(100)}\n`;
    
    // write() returns false เมื่อ buffer เต็ม
    const canContinue = output.write(data);
    
    if (!canContinue) {
      // รอให้ buffer drain ก่อน
      await new Promise(resolve => output.once('drain', resolve));
    }
  }
  
  output.end();
}

writeLargeData().then(() => console.log('Done!'));

// ===== Custom Writable Stream =====
class DatabaseWriter extends Writable {
  constructor(options = {}) {
    super({ objectMode: true, ...options });
    this.records = [];
    this.writeCount = 0;
  }

  _write(chunk, encoding, callback) {
    // chunk คือ object (objectMode: true)
    this.records.push(chunk);
    this.writeCount++;
    
    // จำลองการ insert into database
    setTimeout(() => {
      console.log(`Inserted record ${this.writeCount}:`, chunk);
      callback(); // บอก stream ว่าเขียนเสร็จแล้ว
    }, 10);
  }

  _final(callback) {
    // รันก่อน 'finish' event
    console.log(`Total records: ${this.records.length}`);
    callback();
  }
}

const db = new DatabaseWriter();

db.write({ id: 1, name: 'Alice' });
db.write({ id: 2, name: 'Bob' });
db.write({ id: 3, name: 'Charlie' });
db.end();

db.on('finish', () => console.log('All records saved'));
```

---

## Step 1114: Transform Streams

```javascript
const { Transform, pipeline } = require('stream');
const fs = require('fs');
const zlib = require('zlib');

// ===== Custom Transform Stream =====
class UpperCaseTransform extends Transform {
  _transform(chunk, encoding, callback) {
    // แปลงข้อมูล
    const upperCased = chunk.toString().toUpperCase();
    this.push(upperCased);
    callback();
  }
}

const upper = new UpperCaseTransform();

process.stdin.pipe(upper).pipe(process.stdout);
// พิมพ์อะไรก็จะแปลงเป็น uppercase

// ===== LineByLine Transform =====
class LineByLine extends Transform {
  constructor(options = {}) {
    super(options);
    this.buffer = '';
  }

  _transform(chunk, encoding, callback) {
    this.buffer += chunk.toString();
    const lines = this.buffer.split('\n');
    this.buffer = lines.pop(); // เก็บ partial line
    
    for (const line of lines) {
      this.push(line + '\n');
    }
    
    callback();
  }

  _flush(callback) {
    if (this.buffer) {
      this.push(this.buffer);
    }
    callback();
  }
}

// ===== JSON Lines Transform =====
class JSONLinesParser extends Transform {
  constructor() {
    super({ objectMode: true });
    this.buffer = '';
  }

  _transform(chunk, encoding, callback) {
    this.buffer += chunk.toString();
    const lines = this.buffer.split('\n');
    this.buffer = lines.pop();
    
    for (const line of lines) {
      if (line.trim()) {
        try {
          this.push(JSON.parse(line));
        } catch (err) {
          this.emit('error', new Error(`Invalid JSON: ${line}`));
        }
      }
    }
    
    callback();
  }
}

// ===== Compress/Decompress Files =====
async function compressFile(input, output) {
  const { promisify } = require('util');
  const pipelineAsync = promisify(pipeline);
  
  await pipelineAsync(
    fs.createReadStream(input),
    zlib.createGzip(),
    fs.createWriteStream(output)
  );
  
  console.log(`Compressed ${input} -> ${output}`);
}

async function decompressFile(input, output) {
  const { promisify } = require('util');
  const pipelineAsync = promisify(pipeline);
  
  await pipelineAsync(
    fs.createReadStream(input),
    zlib.createGunzip(),
    fs.createWriteStream(output)
  );
  
  console.log(`Decompressed ${input} -> ${output}`);
}

compressFile('./data.txt', './data.txt.gz');
```

---

## Step 1115: Piping Streams

```javascript
const fs = require('fs');
const zlib = require('zlib');
const crypto = require('crypto');
const { pipeline, Transform } = require('stream');
const { promisify } = require('util');

const pipelineAsync = promisify(pipeline);

// ===== pipe() =====
// อ่านไฟล์ -> compress -> บันทึก
fs.createReadStream('./source.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('./source.txt.gz'));

// ===== pipeline() - แนะนำ! (จัดการ error ได้ดีกว่า) =====
async function processFile() {
  const progressTracker = new Transform({
    transform(chunk, enc, cb) {
      process.stdout.write('.');
      cb(null, chunk);
    }
  });
  
  await pipelineAsync(
    fs.createReadStream('./large.txt'),
    progressTracker,
    zlib.createGzip(),
    fs.createWriteStream('./large.txt.gz')
  );
  
  console.log('\nCompression complete!');
}

// ===== ตัวอย่าง ETL Pipeline =====
// Extract -> Transform -> Load
class CSVParser extends Transform {
  constructor() {
    super({ objectMode: true });
    this.headers = null;
    this.buffer = '';
  }

  _transform(chunk, enc, cb) {
    this.buffer += chunk.toString();
    const lines = this.buffer.split('\n');
    this.buffer = lines.pop();
    
    for (const line of lines) {
      if (!line.trim()) continue;
      
      if (!this.headers) {
        this.headers = line.split(',').map(h => h.trim());
      } else {
        const values = line.split(',');
        const record = {};
        this.headers.forEach((h, i) => {
          record[h] = (values[i] || '').trim();
        });
        this.push(record);
      }
    }
    
    cb();
  }
}

class DataValidator extends Transform {
  constructor() {
    super({ objectMode: true });
    this.validCount = 0;
    this.invalidCount = 0;
  }

  _transform(record, enc, cb) {
    if (record.name && record.email && record.email.includes('@')) {
      this.validCount++;
      this.push(record);
    } else {
      this.invalidCount++;
    }
    cb();
  }
}

class JSONWriter extends Writable {
  constructor(filePath) {
    super({ objectMode: true });
    this.stream = fs.createWriteStream(filePath);
    this.first = true;
    this.stream.write('[\n');
  }

  _write(record, enc, cb) {
    const prefix = this.first ? '' : ',\n';
    this.stream.write(prefix + JSON.stringify(record, null, 2));
    this.first = false;
    cb();
  }

  _final(cb) {
    this.stream.write('\n]\n', cb);
  }
}

async function runETLPipeline() {
  const validator = new DataValidator();
  
  await pipelineAsync(
    fs.createReadStream('./users.csv'),
    new CSVParser(),
    validator,
    new JSONWriter('./users.json')
  );
  
  console.log(`Valid: ${validator.validCount}, Invalid: ${validator.invalidCount}`);
}
```

---

## Step 1116: Buffer Object

```javascript
// Buffer เป็น fixed-length byte sequence
// ใช้สำหรับ binary data, files, network packets

// ===== สร้าง Buffer =====
// จาก string
const buf1 = Buffer.from('Hello, World!');
const buf2 = Buffer.from('Hello', 'utf8');      // default encoding
const buf3 = Buffer.from('48656c6c6f', 'hex');  // from hex
const buf4 = Buffer.from('SGVsbG8=', 'base64'); // from base64

// จาก array of bytes
const buf5 = Buffer.from([72, 101, 108, 108, 111]);

// สร้าง empty buffer
const buf6 = Buffer.alloc(10);        // 10 bytes, filled with 0
const buf7 = Buffer.alloc(10, 0xff); // filled with 0xFF
const buf8 = Buffer.allocUnsafe(10);  // faster but uninitialized (อันตราย!)

// ===== อ่าน Buffer =====
console.log(buf1.toString());           // 'Hello, World!'
console.log(buf1.toString('hex'));      // hex string
console.log(buf1.toString('base64'));   // base64 string
console.log(buf1.length);              // 13 bytes

// เข้าถึง byte แต่ละตัว
console.log(buf1[0]);                  // 72 (ASCII code of 'H')

// ===== เขียน Buffer =====
const buf = Buffer.alloc(5);
buf.write('Hello');
console.log(buf.toString()); // 'Hello'

buf[0] = 0x48; // 'H'
buf.writeUInt8(101, 1); // 'e' at index 1

// ===== Buffer operations =====
const a = Buffer.from('Hello');
const b = Buffer.from(', World!');

// Concatenate
const combined = Buffer.concat([a, b]);
console.log(combined.toString()); // 'Hello, World!'

// Copy
const source = Buffer.from('Hello, World!');
const target = Buffer.alloc(5);
source.copy(target, 0, 0, 5);
console.log(target.toString()); // 'Hello'

// Slice
const slice = source.slice(7, 12);
console.log(slice.toString()); // 'World'

// Compare
const x = Buffer.from('abc');
const y = Buffer.from('abc');
const z = Buffer.from('xyz');

console.log(Buffer.compare(x, y));  // 0 (equal)
console.log(Buffer.compare(x, z));  // -1 (x < z)
console.log(x.equals(y));           // true

// ===== Reading binary file =====
const imgBuffer = require('fs').readFileSync('./image.jpg');
console.log('File type magic bytes:',
  imgBuffer.slice(0, 4).toString('hex'));
// JPEG: ffd8ff
// PNG: 89504e47
// PDF: 25504446

// ===== Converting between formats =====
function hexToBase64(hex) {
  return Buffer.from(hex, 'hex').toString('base64');
}

function base64ToHex(b64) {
  return Buffer.from(b64, 'base64').toString('hex');
}

const hex = '48656c6c6f';
const b64 = hexToBase64(hex);
console.log('Hex:', hex);
console.log('Base64:', b64);
console.log('Back to hex:', base64ToHex(b64));
```

---

## Step 1117: Child Processes

```javascript
const { spawn, exec, execFile, fork } = require('child_process');
const { promisify } = require('util');

// ===== exec() - รัน shell command =====
// ง่าย แต่ buffer output ทั้งหมด (จำกัด 1MB)

exec('ls -la', (error, stdout, stderr) => {
  if (error) {
    console.error('Error:', error.message);
    return;
  }
  if (stderr) {
    console.error('Stderr:', stderr);
  }
  console.log('Output:', stdout);
});

// exec as Promise
const execAsync = promisify(exec);

async function runCommand(cmd) {
  try {
    const { stdout, stderr } = await execAsync(cmd);
    return stdout.trim();
  } catch (err) {
    throw new Error(`Command failed: ${err.message}`);
  }
}

async function getNodeVersion() {
  const version = await runCommand('node --version');
  console.log('Node version:', version);
}

// ===== spawn() - รัน process แบบ stream =====
// เหมาะกับ output ขนาดใหญ่

const ls = spawn('ls', ['-la', '/tmp']);

ls.stdout.on('data', (data) => {
  process.stdout.write(data);
});

ls.stderr.on('data', (data) => {
  process.stderr.write(data);
});

ls.on('close', (code) => {
  console.log(`Child process exited with code ${code}`);
});

// รัน Python script
const python = spawn('python3', ['script.py', 'arg1']);

python.stdout.on('data', data => {
  console.log('Python output:', data.toString());
});

python.stdin.write('input data\n');
python.stdin.end();

// ===== execFile() - รัน executable โดยตรง =====
execFile('/usr/bin/node', ['--version'], (err, stdout) => {
  if (err) throw err;
  console.log('Version:', stdout.trim());
});

// ===== fork() - สร้าง Node.js child process =====
// มี IPC channel สำหรับ parent-child communication

// worker.js
if (require.main === module) {
  process.on('message', (msg) => {
    console.log('Worker received:', msg);
    
    if (msg.type === 'calculate') {
      const result = heavyCalculation(msg.data);
      process.send({ type: 'result', result });
    }
  });
  
  process.send({ type: 'ready' });
}

function heavyCalculation(data) {
  // CPU-intensive work
  let sum = 0;
  for (let i = 0; i < data; i++) {
    sum += Math.sqrt(i);
  }
  return sum;
}

// main.js
const worker = fork('./worker.js');

worker.on('message', (msg) => {
  if (msg.type === 'ready') {
    console.log('Worker ready!');
    worker.send({ type: 'calculate', data: 1000000 });
  } else if (msg.type === 'result') {
    console.log('Result:', msg.result);
    worker.kill();
  }
});

worker.on('exit', (code) => {
  console.log('Worker exited with code:', code);
});

// ===== spawn แบบ detached =====
const subprocess = spawn('node', ['long-running.js'], {
  detached: true,
  stdio: 'ignore'
});

subprocess.unref(); // parent สามารถ exit ได้โดยไม่รอ child
```

---

## Step 1118: Cluster Module

```javascript
const cluster = require('cluster');
const http = require('http');
const os = require('os');

const numCPUs = os.cpus().length;

if (cluster.isPrimary) {
  // ===== Primary process =====
  console.log(`Primary ${process.pid} is running`);
  console.log(`Forking ${numCPUs} workers...`);

  // สร้าง worker สำหรับแต่ละ CPU
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  // ดู worker events
  cluster.on('online', (worker) => {
    console.log(`Worker ${worker.process.pid} is online`);
  });

  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died (${signal || code})`);
    console.log('Starting a new worker...');
    cluster.fork(); // restart worker
  });

  // ส่ง message ถึง workers
  for (const id in cluster.workers) {
    cluster.workers[id].send({ type: 'config', value: 'production' });
  }

} else {
  // ===== Worker process =====
  const server = http.createServer((req, res) => {
    res.writeHead(200);
    res.end(`Hello from Worker ${process.pid}\n`);
  });

  server.listen(3000, () => {
    console.log(`Worker ${process.pid} started`);
  });

  // รับ message จาก primary
  process.on('message', (msg) => {
    if (msg.type === 'config') {
      console.log(`Worker ${process.pid} received config:`, msg.value);
    }
  });

  // graceful shutdown
  process.on('SIGTERM', () => {
    server.close(() => {
      process.exit(0);
    });
  });
}

// ===== Zero-downtime restart =====
// เมื่อต้อง deploy ใหม่ โดยไม่หยุด server

if (cluster.isPrimary) {
  let workers = [];
  
  function restartWorkers() {
    const oldWorkers = Object.values(cluster.workers);
    
    // สร้าง worker ใหม่ก่อน
    for (let i = 0; i < numCPUs; i++) {
      workers.push(cluster.fork());
    }
    
    // รอให้ worker ใหม่ ready แล้วค่อยหยุด worker เก่า
    setTimeout(() => {
      oldWorkers.forEach(worker => {
        worker.send({ type: 'shutdown' });
        setTimeout(() => worker.kill(), 5000);
      });
    }, 1000);
  }

  process.on('SIGUSR2', restartWorkers);
}
```

---

## Step 1119: Worker Threads

```javascript
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');
const os = require('os');

// Worker Threads ใช้ shared memory (SharedArrayBuffer)
// เหมาะกับ CPU-intensive tasks

// ===== Simple Worker Thread =====

// worker.js (ถ้าเป็น separate file)
if (!isMainThread) {
  const { start, end } = workerData;
  
  let sum = 0;
  for (let i = start; i <= end; i++) {
    sum += i;
  }
  
  parentPort.postMessage(sum);
}

// main.js
if (isMainThread) {
  function runWorker(start, end) {
    return new Promise((resolve, reject) => {
      const worker = new Worker(__filename, {
        workerData: { start, end }
      });
      
      worker.on('message', resolve);
      worker.on('error', reject);
      worker.on('exit', (code) => {
        if (code !== 0) {
          reject(new Error(`Worker stopped with exit code ${code}`));
        }
      });
    });
  }

  async function parallelSum() {
    const total = 1000000;
    const numWorkers = os.cpus().length;
    const chunkSize = Math.ceil(total / numWorkers);
    
    const promises = [];
    for (let i = 0; i < numWorkers; i++) {
      const start = i * chunkSize + 1;
      const end = Math.min((i + 1) * chunkSize, total);
      promises.push(runWorker(start, end));
    }
    
    const results = await Promise.all(promises);
    const sum = results.reduce((a, b) => a + b, 0);
    
    console.log(`Sum 1 to ${total}:`, sum);
  }
  
  parallelSum();
}

// ===== Inline Worker (ไม่ต้องสร้างไฟล์แยก) =====
const { Worker: W2 } = require('worker_threads');

function createInlineWorker(fn, data) {
  const code = `
    const { parentPort, workerData } = require('worker_threads');
    const fn = ${fn.toString()};
    const result = fn(workerData);
    parentPort.postMessage(result);
  `;
  
  return new Promise((resolve, reject) => {
    const worker = new W2(code, { eval: true, workerData: data });
    worker.on('message', resolve);
    worker.on('error', reject);
  });
}

async function main() {
  const result = await createInlineWorker(
    (data) => {
      let sum = 0;
      for (let i = 0; i <= data.n; i++) sum += i;
      return sum;
    },
    { n: 1000000 }
  );
  
  console.log('Result:', result);
}

main();

// ===== SharedArrayBuffer =====
const { Worker: W3, isMainThread: isMT, workerData: wd } = require('worker_threads');

if (isMT) {
  const shared = new SharedArrayBuffer(4);
  const arr = new Int32Array(shared);
  arr[0] = 0;
  
  const workers = Array.from({ length: 4 }, () =>
    new W3(__filename, { workerData: { shared } })
  );
  
  Promise.all(workers.map(w => new Promise(res => w.on('exit', res))))
    .then(() => {
      console.log('Final count:', arr[0]);
    });
} else {
  const arr = new Int32Array(wd.shared);
  
  for (let i = 0; i < 1000; i++) {
    Atomics.add(arr, 0, 1); // thread-safe increment
  }
}
```

---

## Step 1120: Event-driven Architecture

```javascript
const EventEmitter = require('events');

// ===== Event-driven patterns =====

// Pattern 1: Domain Events
class OrderService extends EventEmitter {
  constructor() {
    super();
    this.orders = new Map();
  }

  placeOrder(order) {
    const id = Date.now().toString();
    this.orders.set(id, { ...order, id, status: 'pending' });
    this.emit('order:placed', { id, ...order });
    return id;
  }

  confirmOrder(id) {
    const order = this.orders.get(id);
    if (!order) throw new Error('Order not found');
    
    order.status = 'confirmed';
    this.emit('order:confirmed', order);
  }

  shipOrder(id, trackingNumber) {
    const order = this.orders.get(id);
    if (!order) throw new Error('Order not found');
    
    order.status = 'shipped';
    order.trackingNumber = trackingNumber;
    this.emit('order:shipped', order);
  }
}

const orderService = new OrderService();

// Subscribers
orderService.on('order:placed', (order) => {
  console.log(`[Email] New order placed: ${order.id}`);
});

orderService.on('order:placed', (order) => {
  console.log(`[Inventory] Reserve stock for order: ${order.id}`);
});

orderService.on('order:confirmed', (order) => {
  console.log(`[Email] Order ${order.id} confirmed`);
});

orderService.on('order:shipped', (order) => {
  console.log(`[SMS] Order ${order.id} shipped. Tracking: ${order.trackingNumber}`);
});

const orderId = orderService.placeOrder({
  product: 'Laptop',
  quantity: 1,
  price: 25000
});

orderService.confirmOrder(orderId);
orderService.shipOrder(orderId, 'TH123456789');

// Pattern 2: Event Bus (pub/sub)
class EventBus {
  #listeners = new Map();

  subscribe(event, listener) {
    if (!this.#listeners.has(event)) {
      this.#listeners.set(event, new Set());
    }
    this.#listeners.get(event).add(listener);
    
    // Return unsubscribe function
    return () => this.#listeners.get(event)?.delete(listener);
  }

  publish(event, data) {
    const listeners = this.#listeners.get(event);
    if (!listeners) return;
    
    for (const listener of listeners) {
      try {
        listener(data);
      } catch (err) {
        console.error(`Event listener error for ${event}:`, err);
      }
    }
  }

  once(event, listener) {
    const unsubscribe = this.subscribe(event, (data) => {
      listener(data);
      unsubscribe();
    });
    return unsubscribe;
  }
}

const bus = new EventBus();

const unsub = bus.subscribe('user:login', ({ userId, timestamp }) => {
  console.log(`User ${userId} logged in at ${timestamp}`);
});

bus.once('user:login', ({ userId }) => {
  console.log(`First login detected for user ${userId}`);
});

bus.publish('user:login', { userId: 'u123', timestamp: new Date() });
bus.publish('user:login', { userId: 'u456', timestamp: new Date() });

unsub(); // unsubscribe first listener
bus.publish('user:login', { userId: 'u789', timestamp: new Date() });
```

---

## Step 1121: Error Handling ใน Node.js

```javascript
// ===== Types of errors =====

// 1. Synchronous errors - try/catch
function divide(a, b) {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
}

try {
  console.log(divide(10, 2));   // 5
  console.log(divide(10, 0));   // throws!
} catch (err) {
  console.error('Caught:', err.message);
}

// 2. Custom Error Classes
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
    Error.captureStackTrace(this, ValidationError);
  }
}

class DatabaseError extends Error {
  constructor(message, code) {
    super(message);
    this.name = 'DatabaseError';
    this.code = code;
  }
}

class NotFoundError extends Error {
  constructor(resource, id) {
    super(`${resource} with id ${id} not found`);
    this.name = 'NotFoundError';
    this.resource = resource;
    this.id = id;
    this.statusCode = 404;
  }
}

// 3. Async errors
async function fetchUser(id) {
  if (!id) throw new ValidationError('ID is required', 'id');
  
  // จำลอง database query
  if (id === 999) throw new NotFoundError('User', id);
  
  return { id, name: 'Alice' };
}

async function main() {
  try {
    const user = await fetchUser(1);
    console.log('User:', user);
    
    await fetchUser(999); // throws NotFoundError
  } catch (err) {
    if (err instanceof NotFoundError) {
      console.log(`404: ${err.message}`);
    } else if (err instanceof ValidationError) {
      console.log(`Validation failed on ${err.field}: ${err.message}`);
    } else {
      console.error('Unexpected error:', err);
      throw err; // re-throw unexpected errors
    }
  }
}

// 4. Callback-style error handling (Node.js convention)
function readConfig(path, callback) {
  const fs = require('fs');
  
  fs.readFile(path, 'utf8', (err, data) => {
    if (err) {
      callback(new Error(`Failed to read config: ${err.message}`));
      return;
    }
    
    try {
      const config = JSON.parse(data);
      callback(null, config); // success: null error, then result
    } catch (parseErr) {
      callback(new Error('Invalid JSON config'));
    }
  });
}

// 5. Global error handlers
process.on('uncaughtException', (err, origin) => {
  console.error('UNCAUGHT EXCEPTION:', err.message);
  console.error('Origin:', origin);
  // Log error, then exit
  process.exit(1);
});

process.on('unhandledRejection', (reason, promise) => {
  console.error('UNHANDLED REJECTION:', reason);
  // Log error
  process.exit(1);
});

// ===== Error middleware pattern =====
class AppError extends Error {
  constructor(message, statusCode = 500, code = 'INTERNAL_ERROR') {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
    this.isOperational = true; // operational vs programming errors
  }
  
  toJSON() {
    return {
      error: {
        message: this.message,
        code: this.code,
        statusCode: this.statusCode
      }
    };
  }
}

function isOperationalError(err) {
  if (err instanceof AppError) return err.isOperational;
  return false;
}

function handleError(err) {
  if (isOperationalError(err)) {
    // Operational errors: safe to handle
    console.log('Operational error:', err.message);
  } else {
    // Programming errors: crash and restart
    console.error('Programming error:', err);
    process.exit(1);
  }
}
```

---

## Step 1122: Environment Variables และ dotenv

```javascript
// ===== Environment Variables =====

// กำหนดผ่าน command line
// NODE_ENV=production PORT=4000 node app.js

// อ่านใน code
const env = process.env.NODE_ENV || 'development';
const port = parseInt(process.env.PORT, 10) || 3000;
const dbUrl = process.env.DATABASE_URL;

if (!dbUrl) {
  console.error('DATABASE_URL is required');
  process.exit(1);
}

// ===== dotenv package =====
// ติดตั้ง: npm install dotenv

// .env file:
// NODE_ENV=development
// PORT=3000
// DATABASE_URL=postgres://user:pass@localhost:5432/mydb
// JWT_SECRET=my-super-secret-key
// API_KEY=abcdef123456

// โหลด .env
require('dotenv').config();

// หรือระบุ path
require('dotenv').config({ path: '.env.local' });

// ===== .env.example =====
// ควรสร้างไฟล์ .env.example ที่มีชื่อตัวแปรแต่ไม่มีค่าจริง
// .env.example:
// NODE_ENV=development
// PORT=3000
// DATABASE_URL=postgres://user:password@localhost:5432/dbname
// JWT_SECRET=change-me-in-production
// API_KEY=your-api-key-here

// ===== Config module pattern =====
// config.js
require('dotenv').config();

const required = ['DATABASE_URL', 'JWT_SECRET'];
const missing = required.filter(key => !process.env[key]);

if (missing.length > 0) {
  throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
}

const config = {
  env: process.env.NODE_ENV || 'development',
  port: parseInt(process.env.PORT, 10) || 3000,
  
  db: {
    url: process.env.DATABASE_URL,
    poolMin: parseInt(process.env.DB_POOL_MIN, 10) || 2,
    poolMax: parseInt(process.env.DB_POOL_MAX, 10) || 10,
  },
  
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN || '7d',
  },
  
  email: {
    host: process.env.SMTP_HOST || 'smtp.gmail.com',
    port: parseInt(process.env.SMTP_PORT, 10) || 587,
    user: process.env.SMTP_USER,
    pass: process.env.SMTP_PASS,
  },
  
  isDevelopment: process.env.NODE_ENV === 'development',
  isProduction: process.env.NODE_ENV === 'production',
  isTest: process.env.NODE_ENV === 'test',
};

module.exports = config;

// การใช้งาน:
const config = require('./config');
console.log('Running on port:', config.port);
console.log('Database:', config.db.url ? 'configured' : 'missing');
```

---

## Step 1123: Debugging Node.js

```bash
# ===== Node.js Debugger =====

# รัน debug mode
node --inspect server.js
# output: Debugger listening on ws://127.0.0.1:9229/...

# เปิด Chrome DevTools debugger:
# 1. เปิด Chrome
# 2. ไปที่ chrome://inspect
# 3. Click "Open dedicated DevTools for Node"

# รัน แล้วหยุดทันที
node --inspect-brk server.js

# ===== VS Code Debugging =====
# .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Launch Program",
      "skipFiles": ["<node_internals>/**"],
      "program": "${workspaceFolder}/src/index.js",
      "envFile": "${workspaceFolder}/.env"
    },
    {
      "type": "node",
      "request": "attach",
      "name": "Attach to Running Node",
      "port": 9229
    }
  ]
}
```

```javascript
// ===== console debugging techniques =====

// 1. console.log patterns
const obj = { a: 1, b: { c: 2 } };
console.log('obj:', obj);
console.log(JSON.stringify(obj, null, 2));
console.dir(obj, { depth: null });

// 2. console.time / timeEnd
console.time('database-query');
// ... query ...
console.timeEnd('database-query');
// Output: database-query: 23.456ms

// 3. console.trace
function a() { b(); }
function b() { console.trace('Trace from b'); }
a();

// 4. debugger statement
function processData(data) {
  debugger; // pause ตรงนี้เมื่อใช้ --inspect
  return data.map(x => x * 2);
}

// 5. util.inspect
const util = require('util');
console.log(util.inspect(complexObject, {
  depth: Infinity,
  colors: true
}));

// ===== Performance profiling =====
const { performance, PerformanceObserver } = require('perf_hooks');

// Mark and Measure
performance.mark('start');

for (let i = 0; i < 1000000; i++) {
  Math.sqrt(i);
}

performance.mark('end');
performance.measure('loop', 'start', 'end');

const obs = new PerformanceObserver((items) => {
  items.getEntries().forEach(entry => {
    console.log(`${entry.name}: ${entry.duration.toFixed(3)}ms`);
  });
  obs.disconnect();
});
obs.observe({ entryTypes: ['measure'] });

// ===== Memory leak detection =====
setInterval(() => {
  const mem = process.memoryUsage();
  console.log({
    rss: `${(mem.rss / 1024 / 1024).toFixed(2)}MB`,
    heap: `${(mem.heapUsed / 1024 / 1024).toFixed(2)}/${(mem.heapTotal / 1024 / 1024).toFixed(2)}MB`,
    external: `${(mem.external / 1024 / 1024).toFixed(2)}MB`
  });
}, 5000);
```

---

## Step 1124: npm Package Creation

```bash
# สร้าง package ใหม่
mkdir my-awesome-package
cd my-awesome-package
npm init
```

```javascript
// index.js - main entry point
'use strict';

/**
 * A simple utility library
 * @module my-awesome-package
 */

/**
 * Capitalize first letter of string
 * @param {string} str - Input string
 * @returns {string} Capitalized string
 */
function capitalize(str) {
  if (typeof str !== 'string') throw new TypeError('Expected string');
  if (!str) return str;
  return str.charAt(0).toUpperCase() + str.slice(1);
}

/**
 * Debounce function
 * @param {Function} fn - Function to debounce
 * @param {number} delay - Delay in milliseconds
 * @returns {Function} Debounced function
 */
function debounce(fn, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

/**
 * Deep clone an object
 * @param {*} obj - Object to clone
 * @returns {*} Deep cloned object
 */
function deepClone(obj) {
  if (obj === null || typeof obj !== 'object') return obj;
  if (obj instanceof Date) return new Date(obj.getTime());
  if (obj instanceof Array) return obj.map(deepClone);
  
  const cloned = {};
  for (const key of Object.keys(obj)) {
    cloned[key] = deepClone(obj[key]);
  }
  return cloned;
}

/**
 * Chunk array into smaller arrays
 * @param {Array} arr - Input array
 * @param {number} size - Chunk size
 * @returns {Array[]} Array of chunks
 */
function chunk(arr, size) {
  const chunks = [];
  for (let i = 0; i < arr.length; i += size) {
    chunks.push(arr.slice(i, i + size));
  }
  return chunks;
}

module.exports = {
  capitalize,
  debounce,
  deepClone,
  chunk
};
```

```json
// package.json สำหรับ publishing
{
  "name": "my-awesome-package",
  "version": "1.0.0",
  "description": "A simple utility library",
  "main": "index.js",
  "exports": {
    ".": {
      "require": "./index.js",
      "import": "./index.mjs"
    }
  },
  "files": ["index.js", "index.mjs", "README.md"],
  "keywords": ["utility", "helpers"],
  "author": "Your Name",
  "license": "MIT",
  "homepage": "https://github.com/username/my-awesome-package",
  "repository": {
    "type": "git",
    "url": "https://github.com/username/my-awesome-package.git"
  },
  "bugs": {
    "url": "https://github.com/username/my-awesome-package/issues"
  },
  "engines": {
    "node": ">=16.0.0"
  },
  "scripts": {
    "test": "jest",
    "lint": "eslint index.js"
  }
}
```

---

## Step 1125: Publishing to npm

```bash
# ===== Publishing to npm =====

# 1. สร้าง account ที่ npmjs.com

# 2. Login
npm login

# 3. ตรวจสอบ package ก่อน publish
npm pack --dry-run

# 4. Publish
npm publish

# Publish แบบ scoped (ต้องระบุ @scope)
# package name: @username/my-package
npm publish --access public  # scoped packages ต้อง explicit

# ===== Version management =====
# Semantic Versioning: MAJOR.MINOR.PATCH

npm version patch   # 1.0.0 -> 1.0.1 (bug fixes)
npm version minor   # 1.0.0 -> 1.1.0 (new features)
npm version major   # 1.0.0 -> 2.0.0 (breaking changes)
npm version 2.0.0   # set specific version

# publish new version
npm publish

# ===== .npmignore =====
# ไฟล์ที่ไม่ต้อง publish (คล้าย .gitignore)
```

```
# .npmignore
test/
tests/
*.test.js
*.spec.js
.eslintrc*
.prettierrc*
.github/
coverage/
docs/
examples/
```

```bash
# ===== npm deprecate =====
npm deprecate my-package@"< 2.0.0" "Please upgrade to v2.x"

# ===== npm unpublish =====
# ลบ package (ทำได้เฉพาะใน 72 ชั่วโมงแรก)
npm unpublish my-package@1.0.0

# ===== Local package development =====
# ทดสอบ package ใน local
cd my-awesome-package
npm link  # creates global symlink

cd ../my-project
npm link my-awesome-package  # links to local version

# เมื่อเสร็จแล้ว
npm unlink my-awesome-package
```

---

## Step 1126: Node.js Performance Optimization

```javascript
// ===== 1. Caching =====
const cache = new Map();

async function getUser(id) {
  if (cache.has(id)) {
    return cache.get(id);
  }
  
  const user = await fetchFromDB(id);
  cache.set(id, user);
  
  // expire cache หลัง 5 นาที
  setTimeout(() => cache.delete(id), 5 * 60 * 1000);
  
  return user;
}

// ===== 2. Stream Processing =====
// ใช้ streams แทนการโหลดทั้งหมดเข้า memory
const fs = require('fs');
const readline = require('readline');

async function processLargeCSV(filePath) {
  const fileStream = fs.createReadStream(filePath);
  const rl = readline.createInterface({
    input: fileStream,
    crlfDelay: Infinity
  });
  
  let count = 0;
  for await (const line of rl) {
    // process line
    count++;
  }
  
  return count;
}

// ===== 3. Avoid synchronous operations in hot paths =====
// BAD:
app.get('/api/data', (req, res) => {
  const data = fs.readFileSync('./data.json'); // blocks event loop!
  res.json(JSON.parse(data));
});

// GOOD:
const fs = require('fs').promises;
app.get('/api/data', async (req, res) => {
  const data = await fs.readFile('./data.json', 'utf8');
  res.json(JSON.parse(data));
});

// ===== 4. Efficient object creation =====
// BAD:
function processItems(items) {
  return items.map(item => {
    return {
      id: item.id,
      name: item.name,
      price: item.price * 0.9
    };
  });
}

// GOOD: reuse objects, avoid GC pressure
class ProcessedItem {
  constructor(item) {
    this.id = item.id;
    this.name = item.name;
    this.price = item.price * 0.9;
  }
}

// ===== 5. Use appropriate data structures =====
// O(1) lookup vs O(n) array search
const users = new Map(); // หรือ object

// BAD: O(n)
const found = userArray.find(u => u.id === id);

// GOOD: O(1)
const found2 = users.get(id);

// ===== 6. Batch operations =====
// BAD: many small DB queries
async function processUsersBad(userIds) {
  const results = [];
  for (const id of userIds) {
    const user = await db.findOne(id); // N queries!
    results.push(user);
  }
  return results;
}

// GOOD: batch query
async function processUsersGood(userIds) {
  return await db.findMany(userIds); // 1 query
}

// ===== 7. Connection pooling =====
// ใช้ connection pool สำหรับ databases
const pool = {
  min: 2,
  max: 10,
  acquire: 30000, // ms to wait for connection
  idle: 10000     // ms before releasing idle connection
};

// ===== 8. CPU-intensive work =====
// ย้าย CPU-intensive tasks ไป Worker Thread
const { Worker } = require('worker_threads');

function runCPUTask(data) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./cpu-worker.js', { workerData: data });
    worker.on('message', resolve);
    worker.on('error', reject);
  });
}

// ===== 9. Lazy loading =====
let heavyModule = null;

function getHeavyModule() {
  if (!heavyModule) {
    heavyModule = require('./heavy-module');
  }
  return heavyModule;
}

// ===== 10. Avoid Memory Leaks =====
class EventEmitter_ extends require('events') {
  constructor() {
    super();
    this.setMaxListeners(100);
  }
  
  // Always cleanup listeners
  destroy() {
    this.removeAllListeners();
  }
}

// WeakMap/WeakSet สำหรับ cache ที่ GC ได้
const weakCache = new WeakMap();

function getCachedData(obj) {
  if (weakCache.has(obj)) return weakCache.get(obj);
  
  const computed = expensiveComputation(obj);
  weakCache.set(obj, computed);
  return computed;
}
```

---

## Step 1127: Memory Management

```javascript
// ===== V8 Memory Structure =====
// - Stack: local variables, function calls
// - Heap: objects, closures
//   - Young Generation (Scavenger GC): new objects
//   - Old Generation (Mark-Sweep/Compact GC): long-lived objects

// ===== Memory Usage =====
function printMemory() {
  const used = process.memoryUsage();
  console.table({
    'RSS': `${(used.rss / 1024 / 1024).toFixed(2)} MB`,
    'Heap Total': `${(used.heapTotal / 1024 / 1024).toFixed(2)} MB`,
    'Heap Used': `${(used.heapUsed / 1024 / 1024).toFixed(2)} MB`,
    'External': `${(used.external / 1024 / 1024).toFixed(2)} MB`,
    'Array Buffers': `${(used.arrayBuffers / 1024 / 1024).toFixed(2)} MB`,
  });
}

// ===== Common Memory Leaks =====

// 1. Global variables
global.leakyData = []; // grows indefinitely if not cleared

// 2. Closures holding references
function createLeak() {
  const bigData = new Array(1000000).fill('data');
  
  return function() {
    // bigData ยังอยู่ใน memory แม้ function ใหม่ไม่ได้ใช้
    console.log('Hello');
    // Fix: ไม่ capture bigData
  };
}

// 3. Event listeners ที่ไม่ได้ remove
const EventEmitter = require('events');
const emitter = new EventEmitter();

function addListeners() {
  emitter.on('data', (data) => {
    // listener นี้ไม่ถูก remove -> memory leak
  });
}

// Fix: ใช้ once() หรือ removeListener()
function addListenersFixed() {
  const handler = (data) => { /* process */ };
  emitter.on('data', handler);
  
  // cleanup เมื่อเสร็จ
  return () => emitter.removeListener('data', handler);
}

// 4. Timers ที่ไม่ได้ clear
function startTimer() {
  const bigObj = new Array(10000).fill('data');
  
  return setInterval(() => {
    console.log(bigObj.length); // bigObj ถูก retain
  }, 1000);
}

const timer = startTimer();
// clearInterval(timer); // ต้อง clear เมื่อไม่ใช้

// 5. Cache ที่ไม่มีขนาดจำกัด
const unboundedCache = {};
function cacheData(key, value) {
  unboundedCache[key] = value; // grows forever!
}

// Fix: LRU Cache
class LRUCache {
  constructor(maxSize) {
    this.maxSize = maxSize;
    this.cache = new Map();
  }

  get(key) {
    if (!this.cache.has(key)) return null;
    
    // Move to end (most recently used)
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);
    return value;
  }

  set(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    } else if (this.cache.size >= this.maxSize) {
      // Remove least recently used (first item)
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
    
    this.cache.set(key, value);
  }

  has(key) { return this.cache.has(key); }
  get size() { return this.cache.size; }
}

const lru = new LRUCache(100);
lru.set('user:1', { name: 'Alice' });
```

---

## Step 1128: Profiling Node.js Apps

```bash
# ===== Node.js built-in profiler =====
node --prof app.js

# แปลงผลเป็น readable format
node --prof-process isolate-*.log > profile.txt

# ===== clinic.js - performance toolkit =====
npm install -g clinic

# Doctor: วิเคราะห์ performance bottlenecks
clinic doctor -- node app.js

# Flame: CPU flame graph
clinic flame -- node app.js

# Bubbleprof: async profiling
clinic bubbleprof -- node app.js

# ===== 0x - flame graphs =====
npm install -g 0x
0x app.js

# ===== autocannon - HTTP benchmarking =====
npm install -g autocannon
autocannon -c 100 -d 10 http://localhost:3000
```

```javascript
// ===== Custom profiling =====
class Profiler {
  constructor() {
    this.metrics = new Map();
  }

  start(label) {
    this.metrics.set(label, {
      start: process.hrtime.bigint(),
      calls: 0
    });
  }

  end(label) {
    const metric = this.metrics.get(label);
    if (!metric) return;
    
    const duration = Number(process.hrtime.bigint() - metric.start) / 1e6;
    metric.calls++;
    metric.totalTime = (metric.totalTime || 0) + duration;
    metric.lastTime = duration;
    
    return duration;
  }

  wrap(label, fn) {
    return async (...args) => {
      this.start(label);
      try {
        return await fn(...args);
      } finally {
        this.end(label);
      }
    };
  }

  report() {
    console.table(
      Object.fromEntries(
        [...this.metrics.entries()].map(([key, val]) => [
          key,
          {
            calls: val.calls,
            totalTime: `${val.totalTime?.toFixed(2)}ms`,
            avgTime: `${(val.totalTime / val.calls)?.toFixed(2)}ms`,
            lastTime: `${val.lastTime?.toFixed(2)}ms`
          }
        ])
      )
    );
  }
}

const profiler = new Profiler();

// ใช้งาน
async function main() {
  const profiledFetch = profiler.wrap('fetchUser', fetchUser);
  
  for (let i = 0; i < 100; i++) {
    await profiledFetch(i);
  }
  
  profiler.report();
}
```

---

## Step 1129: Monorepo Concepts

```bash
# Monorepo = หลาย packages/apps ใน single repository

# Structure ตัวอย่าง:
# my-monorepo/
#   package.json (root)
#   packages/
#     utils/          # shared utilities
#       package.json
#       src/
#     ui/             # UI components
#       package.json
#       src/
#   apps/
#     web/            # Web application
#       package.json
#       src/
#     api/            # API server
#       package.json
#       src/

# Root package.json
{
  "name": "my-monorepo",
  "private": true,
  "workspaces": ["packages/*", "apps/*"],
  "scripts": {
    "build": "npm run build --workspaces",
    "test": "npm run test --workspaces",
    "dev": "npm run dev --workspaces --if-present"
  }
}
```

```javascript
// packages/utils/src/index.js
module.exports = {
  formatDate: (date) => date.toISOString().split('T')[0],
  formatCurrency: (amount, currency = 'THB') => 
    new Intl.NumberFormat('th-TH', { style: 'currency', currency }).format(amount)
};

// packages/utils/package.json
{
  "name": "@myorg/utils",
  "version": "1.0.0",
  "main": "src/index.js"
}

// apps/api/package.json
{
  "name": "@myorg/api",
  "dependencies": {
    "@myorg/utils": "*",   // ใช้ local workspace package
    "express": "^4.18.2"
  }
}

// apps/api/src/index.js
const { formatDate, formatCurrency } = require('@myorg/utils');
const express = require('express');

const app = express();

app.get('/api/info', (req, res) => {
  res.json({
    date: formatDate(new Date()),
    price: formatCurrency(999.99)
  });
});
```

---

## Step 1130: Node.js Best Practices สรุป

```javascript
// ===== 1. Configuration =====
// ใช้ environment variables + config module
// ไม่ hardcode credentials

// ===== 2. Error Handling =====
// จัดการ error ทุกประเภท
// ใช้ custom error classes
// Log ด้วย proper logging library (winston, pino)

const pino = require('pino');
const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: {
    target: 'pino-pretty'
  }
});

// ===== 3. Graceful Shutdown =====
async function gracefulShutdown(signal) {
  logger.info(`Received ${signal}, starting graceful shutdown...`);
  
  // หยุดรับ requests ใหม่
  server.close(async () => {
    logger.info('HTTP server closed');
    
    // ปิด database connections
    await db.close();
    logger.info('Database connections closed');
    
    // ปิด other resources
    await cache.quit();
    logger.info('Cache connection closed');
    
    process.exit(0);
  });
  
  // Force exit ถ้า graceful shutdown ล้มเหลว
  setTimeout(() => {
    logger.error('Graceful shutdown timeout, forcing exit');
    process.exit(1);
  }, 10000);
}

process.on('SIGTERM', () => gracefulShutdown('SIGTERM'));
process.on('SIGINT', () => gracefulShutdown('SIGINT'));

// ===== 4. Security =====
// - ใช้ HTTPS เสมอใน production
// - Validate input ทุกครั้ง
// - ใช้ helmet สำหรับ HTTP headers
// - Rate limiting
// - ไม่ใช้ eval(), new Function()
// - Update dependencies เป็นประจำ

// ===== 5. Performance =====
// - ใช้ Streams สำหรับข้อมูลใหญ่
// - Caching ที่เหมาะสม
// - Connection pooling
// - Worker Threads สำหรับ CPU-intensive tasks

// ===== 6. Code Organization =====
// ตัวอย่าง project structure:
//
// src/
//   config/         - configuration
//   controllers/    - request handlers
//   middleware/     - express middleware
//   models/         - data models
//   routes/         - route definitions
//   services/       - business logic
//   utils/          - utilities
//   tests/          - test files
//   app.js          - express setup
//   server.js       - server startup
```

---

## สรุป Steps 1111-1130

| Step | หัวข้อ |
|------|--------|
| 1111 | Streams - ทำความรู้จัก |
| 1112 | Readable Streams |
| 1113 | Writable Streams |
| 1114 | Transform Streams |
| 1115 | Piping Streams |
| 1116 | Buffer Object |
| 1117 | Child Processes |
| 1118 | Cluster Module |
| 1119 | Worker Threads |
| 1120 | Event-driven Architecture |
| 1121 | Error Handling ขั้นสูง |
| 1122 | Environment Variables + dotenv |
| 1123 | Debugging Node.js |
| 1124 | npm Package Creation |
| 1125 | Publishing to npm |
| 1126 | Performance Optimization |
| 1127 | Memory Management |
| 1128 | Profiling |
| 1129 | Monorepo Concepts |
| 1130 | Best Practices สรุป |

---

## แบบฝึกหัดท้ายบท

### แบบฝึกหัดที่ 1: Log Processor
สร้างโปรแกรม stream-based ที่:
1. อ่าน log file ขนาดใหญ่ด้วย Readable stream
2. Parse แต่ละบรรทัด (JSON Lines format)
3. กรองเฉพาะ ERROR logs
4. แปลงรูปแบบด้วย Transform stream
5. เขียนผลลัพธ์ลงไฟล์ใหม่

### แบบฝึกหัดที่ 2: Parallel Computation
ใช้ Worker Threads เพื่อ:
1. คำนวณ prime numbers ใน range ที่กำหนด
2. แบ่งงานให้หลาย workers ตามจำนวน CPU
3. รวมผลลัพธ์จากทุก workers
4. เปรียบเทียบ performance กับ single-threaded version

### แบบฝึกหัดที่ 3: Memory Leak Hunt
1. สร้างโปรแกรมที่มี memory leak
2. ใช้ --inspect เพื่อ debug
3. หาสาเหตุและแก้ไข
4. Verify ว่า memory ไม่ leak แล้ว

### แบบฝึกหัดที่ 4: npm Package
สร้าง npm package ที่ประกอบด้วย:
1. Thai date formatter (ปีพุทธศักราช)
2. Thai baht formatter
3. Thai phone number validator
4. Unit tests ด้วย Jest
5. README ภาษาไทย

### แบบฝึกหัดที่ 5: Cluster Server
สร้าง HTTP server ที่:
1. ใช้ Cluster module
2. มี workers เท่ากับจำนวน CPU
3. Worker restart อัตโนมัติเมื่อ crash
4. Graceful shutdown
5. Log แสดง worker PID ที่รับ request

---

*จบ Part 57: Node.js ขั้นสูง - ในบทต่อไปเราจะเรียน Express.js framework*
