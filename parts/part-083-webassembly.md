# Part 83: WebAssembly (WASM)
## Steps 1631-1650

WebAssembly (WASM) คือ binary instruction format ที่ทำงานได้เร็วใกล้เคียง native code
โดยทำงานร่วมกับ JavaScript ได้อย่างไร้รอยต่อ เปิดโอกาสให้นำโค้ดจาก C, C++, Rust มาใช้บน web

---

## Step 1631: What is WebAssembly

```
WebAssembly (WASM) คืออะไร?
- Binary instruction format สำหรับ stack-based virtual machine
- ออกแบบมาเพื่อ performance ใกล้เคียง native code
- รองรับโดย browser หลักทุกตัว: Chrome, Firefox, Safari, Edge
- ทำงานใน sandboxed environment ที่ปลอดภัย
- ไม่ใช่ replacement สำหรับ JavaScript แต่เป็น complement
- สามารถ import/export functions ระหว่าง WASM และ JavaScript ได้

ทำไมถึงเร็วกว่า JavaScript?
1. Binary format - ไม่ต้อง parse text
2. Statically typed - ไม่ต้องทำ type checking at runtime
3. Compiled ahead of time - ไม่ต้อง JIT compile
4. Direct memory access - linear memory model
5. No garbage collection - manual memory management

เมื่อใดควรใช้ WASM?
✓ การประมวลผลภาพ (Image processing)
✓ การเข้ารหัส (Cryptography)
✓ Game engines
✓ Physics simulations
✓ Audio/Video processing
✓ Machine learning inference
✓ Scientific computing

เมื่อใดไม่ควรใช้ WASM?
✗ Simple UI interactions
✗ DOM manipulation
✗ Network requests
✗ Simple string operations
✗ Code ที่ต้องใช้ JavaScript APIs มาก
```

---

## Step 1632: WASM vs JavaScript Performance

```javascript
// Benchmark: Fibonacci sequence
// JavaScript version
function fibJS(n) {
  if (n <= 1) return n;
  return fibJS(n - 1) + fibJS(n - 2);
}

// ทดสอบ performance
console.time("JS Fibonacci");
console.log(fibJS(45));
console.timeEnd("JS Fibonacci");
// ~8 วินาที (recursive ช้า แต่ใช้สำหรับ demo)

// WASM ทำได้เร็วกว่า ~3-10x สำหรับ compute-heavy tasks

// ตัวอย่าง Benchmark ที่ดีกว่า: Matrix multiplication
function multiplyMatricesJS(a, b, size) {
  const result = new Array(size).fill(null).map(() => new Array(size).fill(0));
  
  for (let i = 0; i < size; i++) {
    for (let j = 0; j < size; j++) {
      for (let k = 0; k < size; k++) {
        result[i][j] += a[i][k] * b[k][j];
      }
    }
  }
  
  return result;
}

// การสร้าง matrix
const size = 500;
const matrixA = Array.from({ length: size }, () =>
  Array.from({ length: size }, () => Math.random())
);
const matrixB = Array.from({ length: size }, () =>
  Array.from({ length: size }, () => Math.random())
);

// Benchmark
console.time("JS Matrix Multiply");
multiplyMatricesJS(matrixA, matrixB, size);
console.timeEnd("JS Matrix Multiply");
// ~2-5 วินาที

// WASM version จะเร็วกว่า ~5-10x
```

---

## Step 1633: WebAssembly Text Format (WAT)

```wat
;; WAT (WebAssembly Text Format) - human-readable WASM

;; ไฟล์: math.wat

(module
  ;; Import จาก JavaScript
  (import "env" "log" (func $log (param i32)))
  
  ;; Export memory ให้ JavaScript ใช้
  (memory (export "memory") 1)
  
  ;; Function: บวกเลขสองตัว
  (func $add (export "add") (param $a i32) (param $b i32) (result i32)
    local.get $a
    local.get $b
    i32.add
  )
  
  ;; Function: คำนวณ factorial
  (func $factorial (export "factorial") (param $n i32) (result i32)
    (local $result i32)
    (local $i i32)
    
    i32.const 1
    local.set $result
    
    i32.const 1
    local.set $i
    
    ;; loop: i <= n
    (block $break
      (loop $continue
        ;; if i > n, break
        local.get $i
        local.get $n
        i32.gt_s
        br_if $break
        
        ;; result *= i
        local.get $result
        local.get $i
        i32.mul
        local.set $result
        
        ;; i++
        local.get $i
        i32.const 1
        i32.add
        local.set $i
        
        br $continue
      )
    )
    
    local.get $result
  )
  
  ;; Function: Fibonacci
  (func $fibonacci (export "fibonacci") (param $n i32) (result i32)
    (if (i32.le_s (local.get $n) (i32.const 1))
      (then (return (local.get $n)))
    )
    
    (i32.add
      (call $fibonacci (i32.sub (local.get $n) (i32.const 1)))
      (call $fibonacci (i32.sub (local.get $n) (i32.const 2)))
    )
  )
  
  ;; Function: คำนวณ sum array
  (func $sumArray (export "sumArray") (param $offset i32) (param $length i32) (result i32)
    (local $i i32)
    (local $sum i32)
    (local $value i32)
    
    i32.const 0
    local.set $i
    
    i32.const 0
    local.set $sum
    
    (block $break
      (loop $continue
        ;; if i >= length, break
        local.get $i
        local.get $length
        i32.ge_s
        br_if $break
        
        ;; value = memory[offset + i * 4] (i32 = 4 bytes)
        local.get $offset
        local.get $i
        i32.const 4
        i32.mul
        i32.add
        i32.load
        local.set $value
        
        ;; sum += value
        local.get $sum
        local.get $value
        i32.add
        local.set $sum
        
        ;; i++
        local.get $i
        i32.const 1
        i32.add
        local.set $i
        
        br $continue
      )
    )
    
    local.get $sum
  )
)
```

---

## Step 1634: Compiling from C/C++ with Emscripten

```c
// ไฟล์: image_processing.c

#include <stdint.h>
#include <stdlib.h>
#include <math.h>
#include <emscripten/emscripten.h>

// Grayscale conversion
EMSCRIPTEN_KEEPALIVE
void toGrayscale(uint8_t* data, int width, int height) {
    int pixelCount = width * height;
    for (int i = 0; i < pixelCount; i++) {
        int idx = i * 4;
        uint8_t r = data[idx];
        uint8_t g = data[idx + 1];
        uint8_t b = data[idx + 2];
        
        // ใช้สูตร luminance
        uint8_t gray = (uint8_t)(0.299 * r + 0.587 * g + 0.114 * b);
        
        data[idx] = gray;
        data[idx + 1] = gray;
        data[idx + 2] = gray;
        // alpha ไม่เปลี่ยน
    }
}

// Blur filter (box blur)
EMSCRIPTEN_KEEPALIVE
void applyBlur(
    uint8_t* input,
    uint8_t* output,
    int width,
    int height,
    int radius
) {
    for (int y = 0; y < height; y++) {
        for (int x = 0; x < width; x++) {
            long rSum = 0, gSum = 0, bSum = 0, count = 0;
            
            for (int dy = -radius; dy <= radius; dy++) {
                for (int dx = -radius; dx <= radius; dx++) {
                    int nx = x + dx;
                    int ny = y + dy;
                    
                    if (nx >= 0 && nx < width && ny >= 0 && ny < height) {
                        int idx = (ny * width + nx) * 4;
                        rSum += input[idx];
                        gSum += input[idx + 1];
                        bSum += input[idx + 2];
                        count++;
                    }
                }
            }
            
            int outIdx = (y * width + x) * 4;
            output[outIdx] = (uint8_t)(rSum / count);
            output[outIdx + 1] = (uint8_t)(gSum / count);
            output[outIdx + 2] = (uint8_t)(bSum / count);
            output[outIdx + 3] = input[outIdx + 3];
        }
    }
}

// Edge detection (Sobel)
EMSCRIPTEN_KEEPALIVE
void sobelEdgeDetection(
    uint8_t* input,
    uint8_t* output,
    int width,
    int height
) {
    int Gx[3][3] = {{-1, 0, 1}, {-2, 0, 2}, {-1, 0, 1}};
    int Gy[3][3] = {{-1, -2, -1}, {0, 0, 0}, {1, 2, 1}};
    
    for (int y = 1; y < height - 1; y++) {
        for (int x = 1; x < width - 1; x++) {
            long gxR = 0, gyR = 0;
            long gxG = 0, gyG = 0;
            long gxB = 0, gyB = 0;
            
            for (int ky = -1; ky <= 1; ky++) {
                for (int kx = -1; kx <= 1; kx++) {
                    int idx = ((y + ky) * width + (x + kx)) * 4;
                    gxR += Gx[ky + 1][kx + 1] * input[idx];
                    gyR += Gy[ky + 1][kx + 1] * input[idx];
                    gxG += Gx[ky + 1][kx + 1] * input[idx + 1];
                    gyG += Gy[ky + 1][kx + 1] * input[idx + 1];
                    gxB += Gx[ky + 1][kx + 1] * input[idx + 2];
                    gyB += Gy[ky + 1][kx + 1] * input[idx + 2];
                }
            }
            
            int outIdx = (y * width + x) * 4;
            output[outIdx] = (uint8_t)fmin(255, sqrt(gxR*gxR + gyR*gyR));
            output[outIdx + 1] = (uint8_t)fmin(255, sqrt(gxG*gxG + gyG*gyG));
            output[outIdx + 2] = (uint8_t)fmin(255, sqrt(gxB*gxB + gyB*gyB));
            output[outIdx + 3] = 255;
        }
    }
}
```

```bash
# การ compile ด้วย Emscripten

# ติดตั้ง Emscripten
git clone https://github.com/emscripten-core/emsdk.git
cd emsdk
./emsdk install latest
./emsdk activate latest
source ./emsdk_env.sh

# Compile เป็น WASM
emcc image_processing.c -o image_processing.js \
  -s EXPORTED_FUNCTIONS='["_toGrayscale","_applyBlur","_sobelEdgeDetection","_malloc","_free"]' \
  -s EXPORTED_RUNTIME_METHODS='["ccall","cwrap"]' \
  -s ALLOW_MEMORY_GROWTH=1 \
  -O3 \
  -lm

# หรือ compile เป็น .wasm โดยตรง
emcc image_processing.c -o image_processing.wasm \
  -s SIDE_MODULE=1 \
  -O3 \
  -lm
```

```javascript
// การใช้งาน Emscripten output ใน JavaScript

// โหลด Emscripten module
async function loadImageProcessor() {
  const Module = await createModule();
  
  const toGrayscale = Module.cwrap("toGrayscale", null, ["number", "number", "number"]);
  const applyBlur = Module.cwrap("applyBlur", null, ["number", "number", "number", "number", "number"]);
  
  return {
    async processImage(canvas) {
      const ctx = canvas.getContext("2d");
      const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
      const data = imageData.data;
      
      // จัดสรร memory ใน WASM
      const numBytes = data.length;
      const ptr = Module._malloc(numBytes);
      
      // copy data ไปยัง WASM memory
      Module.HEAPU8.set(data, ptr);
      
      // เรียก WASM function
      toGrayscale(ptr, canvas.width, canvas.height);
      
      // copy ผลลัพธ์กลับมา
      const result = new Uint8ClampedArray(Module.HEAPU8.buffer, ptr, numBytes);
      const newImageData = new ImageData(result.slice(), canvas.width, canvas.height);
      ctx.putImageData(newImageData, 0, 0);
      
      // ปล่อย memory
      Module._free(ptr);
    }
  };
}
```

---

## Step 1635: Compiling from Rust with wasm-pack

```rust
// ไฟล์: src/lib.rs

use wasm_bindgen::prelude::*;
use js_sys::Float64Array;

// เรียกใช้ JavaScript function จาก Rust
#[wasm_bindgen]
extern "C" {
    fn alert(s: &str);
    
    #[wasm_bindgen(js_namespace = console)]
    fn log(s: &str);
    
    #[wasm_bindgen(js_namespace = console, js_name = log)]
    fn log_u32(a: u32);
}

// Export functions ไปยัง JavaScript
#[wasm_bindgen]
pub fn greet(name: &str) {
    alert(&format!("สวัสดี, {}!", name));
}

// Math functions
#[wasm_bindgen]
pub fn fibonacci(n: u32) -> u32 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}

#[wasm_bindgen]
pub fn is_prime(n: u32) -> bool {
    if n < 2 { return false; }
    if n == 2 { return true; }
    if n % 2 == 0 { return false; }
    
    let sqrt_n = (n as f64).sqrt() as u32;
    for i in (3..=sqrt_n).step_by(2) {
        if n % i == 0 { return false; }
    }
    true
}

// Struct ที่ export ไปยัง JavaScript
#[wasm_bindgen]
pub struct Matrix {
    data: Vec<f64>,
    rows: usize,
    cols: usize,
}

#[wasm_bindgen]
impl Matrix {
    #[wasm_bindgen(constructor)]
    pub fn new(rows: usize, cols: usize) -> Matrix {
        Matrix {
            data: vec![0.0; rows * cols],
            rows,
            cols,
        }
    }
    
    pub fn get(&self, row: usize, col: usize) -> f64 {
        self.data[row * self.cols + col]
    }
    
    pub fn set(&mut self, row: usize, col: usize, value: f64) {
        self.data[row * self.cols + col] = value;
    }
    
    pub fn multiply(&self, other: &Matrix) -> Option<Matrix> {
        if self.cols != other.rows {
            return None;
        }
        
        let mut result = Matrix::new(self.rows, other.cols);
        
        for i in 0..self.rows {
            for j in 0..other.cols {
                let mut sum = 0.0;
                for k in 0..self.cols {
                    sum += self.get(i, k) * other.get(k, j);
                }
                result.set(i, j, sum);
            }
        }
        
        Some(result)
    }
    
    pub fn data_ptr(&self) -> *const f64 {
        self.data.as_ptr()
    }
    
    pub fn rows(&self) -> usize {
        self.rows
    }
    
    pub fn cols(&self) -> usize {
        self.cols
    }
}

// Image processing
#[wasm_bindgen]
pub fn grayscale(data: &mut [u8]) {
    for pixel in data.chunks_exact_mut(4) {
        let r = pixel[0] as f32;
        let g = pixel[1] as f32;
        let b = pixel[2] as f32;
        
        let gray = (0.299 * r + 0.587 * g + 0.114 * b) as u8;
        
        pixel[0] = gray;
        pixel[1] = gray;
        pixel[2] = gray;
    }
}

// SHA-256 hash (ใช้เป็น example เท่านั้น)
#[wasm_bindgen]
pub fn hash_string(input: &str) -> String {
    use std::collections::hash_map::DefaultHasher;
    use std::hash::{Hash, Hasher};
    
    let mut hasher = DefaultHasher::new();
    input.hash(&mut hasher);
    format!("{:016x}", hasher.finish())
}
```

```toml
# Cargo.toml

[package]
name = "wasm-math"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib"]

[dependencies]
wasm-bindgen = "0.2"

[profile.release]
opt-level = "s"  # Optimize for size
```

```bash
# ติดตั้งและ build

# ติดตั้ง wasm-pack
curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh

# Build สำหรับ web
wasm-pack build --target web

# Build สำหรับ Node.js
wasm-pack build --target nodejs

# Build สำหรับ bundlers (webpack, vite)
wasm-pack build --target bundler
```

```javascript
// การใช้งาน Rust WASM ใน JavaScript

import init, { fibonacci, is_prime, Matrix, grayscale, hash_string } from "./pkg/wasm_math.js";

async function main() {
  // initialize WASM module
  await init();
  
  // ใช้งาน functions
  console.log(fibonacci(40));  // 102334155
  console.log(is_prime(97));   // true
  
  // ใช้งาน Matrix struct
  const matA = new Matrix(3, 3);
  const matB = new Matrix(3, 3);
  
  // ตั้งค่า
  matA.set(0, 0, 1); matA.set(0, 1, 2); matA.set(0, 2, 3);
  matA.set(1, 0, 4); matA.set(1, 1, 5); matA.set(1, 2, 6);
  matA.set(2, 0, 7); matA.set(2, 1, 8); matA.set(2, 2, 9);
  
  matB.set(0, 0, 9); matB.set(0, 1, 8); matB.set(0, 2, 7);
  matB.set(1, 0, 6); matB.set(1, 1, 5); matB.set(1, 2, 4);
  matB.set(2, 0, 3); matB.set(2, 1, 2); matB.set(2, 2, 1);
  
  const result = matA.multiply(matB);
  console.log("ผลคูณ:", result?.get(0, 0));
  
  // Hash string
  console.log(hash_string("Hello World")); // "abc123def456..."
  
  // Image processing
  const canvas = document.getElementById("canvas");
  const ctx = canvas.getContext("2d");
  const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
  
  grayscale(imageData.data);
  ctx.putImageData(imageData, 0, 0);
}

main();
```

---

## Step 1636: AssemblyScript

```typescript
// AssemblyScript - TypeScript ที่ compile เป็น WASM

// ไฟล์: assembly/index.ts

// Types ที่ใช้ใน AssemblyScript
// i32, i64, f32, f64, u8, u16, u32, u64
// bool, string, Array, StaticArray, ArrayBuffer

export function add(a: i32, b: i32): i32 {
  return a + b;
}

export function fibonacci(n: i32): i32 {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

// String operations
export function reverseString(str: string): string {
  let result = "";
  for (let i = str.length - 1; i >= 0; i--) {
    result += str.charAt(i);
  }
  return result;
}

// Array operations
export function sumArray(arr: Int32Array): i32 {
  let sum: i32 = 0;
  for (let i = 0; i < arr.length; i++) {
    sum += arr[i];
  }
  return sum;
}

// Memory management
export function allocBuffer(size: i32): usize {
  return heap.alloc(size);
}

export function freeBuffer(ptr: usize): void {
  heap.free(ptr);
}

// Image processing
export function grayscaleImage(
  data: Uint8ClampedArray,
  width: i32,
  height: i32
): void {
  const pixelCount = width * height;
  for (let i = 0; i < pixelCount; i++) {
    const idx = i * 4;
    const r = data[idx];
    const g = data[idx + 1];
    const b = data[idx + 2];
    
    const gray: u8 = u8(0.299 * f32(r) + 0.587 * f32(g) + 0.114 * f32(b));
    
    data[idx] = gray;
    data[idx + 1] = gray;
    data[idx + 2] = gray;
  }
}

// Math operations
export function isPrime(n: i32): bool {
  if (n < 2) return false;
  if (n === 2) return true;
  if (n % 2 === 0) return false;
  
  const sqrtN = i32(Math.sqrt(f64(n)));
  for (let i = 3; i <= sqrtN; i += 2) {
    if (n % i === 0) return false;
  }
  return true;
}
```

```bash
# Build AssemblyScript
npm install --save-dev assemblyscript
npx asc assembly/index.ts --target release -o build/release.wasm

# หรือใช้ package.json scripts
# "build": "asc assembly/index.ts --target release"
```

---

## Step 1637: Loading WASM in JavaScript

```javascript
// วิธีต่างๆ ในการโหลด WASM

// Method 1: WebAssembly.instantiateStreaming (แนะนำ)
async function loadWasmStreaming(url) {
  const importObject = {
    env: {
      log: (value) => console.log("WASM log:", value),
      abort: (msg, file, line, col) => {
        console.error("WASM abort:", msg);
      }
    }
  };
  
  const { instance, module } = await WebAssembly.instantiateStreaming(
    fetch(url),
    importObject
  );
  
  return instance.exports;
}

// Method 2: WebAssembly.instantiate (สำหรับ ArrayBuffer)
async function loadWasmFromBuffer(buffer) {
  const importObject = {
    env: {
      memory: new WebAssembly.Memory({ initial: 1, maximum: 100 })
    }
  };
  
  const { instance } = await WebAssembly.instantiate(buffer, importObject);
  return instance.exports;
}

// Method 3: ใช้กับ bundler (Webpack/Vite)
import wasmModule from "./math.wasm?init";

async function initWasm() {
  const instance = await wasmModule({
    env: { abort: () => {} }
  });
  return instance.exports;
}

// Method 4: โหลด inline (สำหรับ small modules)
const wasmCode = Uint8Array.from([
  0x00, 0x61, 0x73, 0x6d, // magic bytes: \0asm
  0x01, 0x00, 0x00, 0x00, // version: 1
  // ... binary code
]);

async function loadWasmInline() {
  const { instance } = await WebAssembly.instantiate(wasmCode.buffer);
  return instance.exports;
}

// ตัวอย่างสมบูรณ์
async function main() {
  const exports = await loadWasmStreaming("/math.wasm");
  
  console.log(exports.add(1, 2));       // 3
  console.log(exports.fibonacci(10));   // 55
  console.log(exports.factorial(5));    // 120
}

main().catch(console.error);
```

---

## Step 1638: WebAssembly.Module และ Memory

```javascript
// WebAssembly.Module - pre-compile module
async function precompileWasm(url) {
  const response = await fetch(url);
  const buffer = await response.arrayBuffer();
  
  // Compile module (สามารถ cache ไว้ใน IndexedDB)
  const module = await WebAssembly.compile(buffer);
  
  // ตรวจสอบ imports/exports
  console.log("Imports:", WebAssembly.Module.imports(module));
  console.log("Exports:", WebAssembly.Module.exports(module));
  
  // Instantiate หลายครั้งจาก module เดียวกัน (ประหยัดเวลา)
  const instance1 = await WebAssembly.instantiate(module, importObject);
  const instance2 = await WebAssembly.instantiate(module, importObject);
  
  return { module, instance1, instance2 };
}

// Linear Memory
function workWithMemory() {
  // สร้าง memory (unit: pages, 1 page = 64KB)
  const memory = new WebAssembly.Memory({
    initial: 1,  // 64KB
    maximum: 10  // 640KB
  });
  
  // อ่านและเขียน memory ผ่าน typed arrays
  const int32View = new Int32Array(memory.buffer);
  const uint8View = new Uint8Array(memory.buffer);
  const float64View = new Float64Array(memory.buffer);
  
  // เขียนข้อมูล
  int32View[0] = 42;
  int32View[1] = 100;
  
  // อ่านข้อมูล
  console.log(int32View[0]); // 42
  
  // เขียน string ลง memory
  function writeString(memory, ptr, str) {
    const encoder = new TextEncoder();
    const bytes = encoder.encode(str + "\0"); // null-terminated
    new Uint8Array(memory.buffer).set(bytes, ptr);
  }
  
  // อ่าน string จาก memory
  function readString(memory, ptr) {
    const bytes = new Uint8Array(memory.buffer, ptr);
    const end = bytes.indexOf(0); // หา null terminator
    return new TextDecoder().decode(bytes.subarray(0, end));
  }
  
  writeString(memory, 100, "Hello, WASM!");
  console.log(readString(memory, 100)); // "Hello, WASM!"
  
  // Grow memory
  console.log(memory.buffer.byteLength); // 65536 (64KB)
  memory.grow(1); // เพิ่ม 1 page = 64KB
  console.log(memory.buffer.byteLength); // 131072 (128KB)
  
  return memory;
}
```

---

## Step 1639: Importing JavaScript Functions into WASM

```wat
;; WASM ที่ import JavaScript functions

(module
  ;; Import console.log
  (import "js" "log_i32" (func $log_i32 (param i32)))
  (import "js" "log_f64" (func $log_f64 (param f64)))
  
  ;; Import Math.random
  (import "js" "random" (func $random (result f64)))
  
  ;; Import custom function
  (import "js" "onProgress" (func $onProgress (param i32 i32)))
  
  ;; ใช้งาน imports
  (func $demo (export "demo")
    ;; Log integer
    i32.const 42
    call $log_i32
    
    ;; Log float
    f64.const 3.14
    call $log_f64
    
    ;; Get random number
    call $random
    call $log_f64
  )
  
  ;; Loop พร้อม progress callback
  (func $processLargeData (export "processLargeData") (param $total i32)
    (local $i i32)
    
    i32.const 0
    local.set $i
    
    (block $break
      (loop $continue
        local.get $i
        local.get $total
        i32.ge_s
        br_if $break
        
        ;; Report progress ทุก 1000 iterations
        (if (i32.rem_u (local.get $i) (i32.const 1000))
          (then
            local.get $i
            local.get $total
            call $onProgress
          )
        )
        
        ;; ... process data ...
        
        local.get $i
        i32.const 1
        i32.add
        local.set $i
        
        br $continue
      )
    )
  )
)
```

```javascript
// JavaScript side
const importObject = {
  js: {
    log_i32: (value) => console.log("i32:", value),
    log_f64: (value) => console.log("f64:", value),
    random: () => Math.random(),
    onProgress: (current, total) => {
      const percent = Math.floor((current / total) * 100);
      progressBar.style.width = `${percent}%`;
      progressText.textContent = `${percent}%`;
    }
  }
};

const { instance } = await WebAssembly.instantiateStreaming(
  fetch("/demo.wasm"),
  importObject
);

instance.exports.demo();
instance.exports.processLargeData(1000000);
```

---

## Step 1640: wasm-bindgen for Rust

```rust
// wasm-bindgen ช่วยให้ Rust และ JavaScript คุยกันได้ง่าย

use wasm_bindgen::prelude::*;
use web_sys::{Document, HtmlElement, Window};

// Access DOM API จาก Rust
#[wasm_bindgen(start)]
pub fn main() -> Result<(), JsValue> {
    let window = web_sys::window().expect("no global `window` exists");
    let document = window.document().expect("should have a document on window");
    let body = document.body().expect("document should have a body");
    
    let val = document.create_element("p")?;
    val.set_inner_html("Hello from Rust! 🦀");
    
    body.append_child(&val)?;
    
    Ok(())
}

// Event listeners
#[wasm_bindgen]
pub struct Button {
    element: HtmlElement,
    click_count: u32,
}

#[wasm_bindgen]
impl Button {
    #[wasm_bindgen(constructor)]
    pub fn new(id: &str) -> Result<Button, JsValue> {
        let window = web_sys::window().unwrap();
        let document = window.document().unwrap();
        
        let element = document
            .get_element_by_id(id)
            .ok_or("element not found")?
            .dyn_into::<HtmlElement>()?;
        
        Ok(Button { element, click_count: 0 })
    }
    
    pub fn on_click(&mut self) {
        self.click_count += 1;
        self.element.set_inner_html(&format!(
            "คลิกแล้ว {} ครั้ง!", 
            self.click_count
        ));
    }
    
    pub fn click_count(&self) -> u32 {
        self.click_count
    }
}

// Fetch API จาก Rust (async)
#[wasm_bindgen]
pub async fn fetch_users() -> Result<JsValue, JsValue> {
    let window = web_sys::window().unwrap();
    
    let response: web_sys::Response = JsFuture::from(
        window.fetch_with_str("https://api.example.com/users")
    ).await?.dyn_into()?;
    
    let json = JsFuture::from(response.json()?).await?;
    Ok(json)
}
```

---

## Step 1641: Practical Use Case - Image Processing

```javascript
// Image processing application ด้วย WASM

class ImageProcessor {
  #wasm = null;
  #memory = null;
  
  async initialize() {
    const importObject = {
      env: {
        memory: new WebAssembly.Memory({ initial: 32 }), // 2MB
        abort: () => { throw new Error("WASM aborted"); }
      }
    };
    
    const { instance } = await WebAssembly.instantiateStreaming(
      fetch("/image-processor.wasm"),
      importObject
    );
    
    this.#wasm = instance.exports;
    this.#memory = importObject.env.memory;
    
    console.log("ImageProcessor initialized");
  }
  
  #copyToWasm(data) {
    const ptr = this.#wasm.malloc(data.length);
    new Uint8Array(this.#memory.buffer).set(data, ptr);
    return ptr;
  }
  
  #copyFromWasm(ptr, length) {
    return new Uint8ClampedArray(this.#memory.buffer, ptr, length).slice();
  }
  
  toGrayscale(imageData) {
    const ptr = this.#copyToWasm(imageData.data);
    
    this.#wasm.toGrayscale(ptr, imageData.width, imageData.height);
    
    const result = this.#copyFromWasm(ptr, imageData.data.length);
    this.#wasm.free(ptr);
    
    return new ImageData(result, imageData.width, imageData.height);
  }
  
  applyBlur(imageData, radius = 3) {
    const inputPtr = this.#copyToWasm(imageData.data);
    const outputPtr = this.#wasm.malloc(imageData.data.length);
    
    this.#wasm.applyBlur(
      inputPtr,
      outputPtr,
      imageData.width,
      imageData.height,
      radius
    );
    
    const result = this.#copyFromWasm(outputPtr, imageData.data.length);
    
    this.#wasm.free(inputPtr);
    this.#wasm.free(outputPtr);
    
    return new ImageData(result, imageData.width, imageData.height);
  }
  
  detectEdges(imageData) {
    const inputPtr = this.#copyToWasm(imageData.data);
    const outputPtr = this.#wasm.malloc(imageData.data.length);
    
    this.#wasm.sobelEdgeDetection(
      inputPtr,
      outputPtr,
      imageData.width,
      imageData.height
    );
    
    const result = this.#copyFromWasm(outputPtr, imageData.data.length);
    
    this.#wasm.free(inputPtr);
    this.#wasm.free(outputPtr);
    
    return new ImageData(result, imageData.width, imageData.height);
  }
}

// การใช้งาน
const processor = new ImageProcessor();
await processor.initialize();

const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");

// โหลดรูปภาพ
const img = new Image();
img.onload = () => {
  canvas.width = img.width;
  canvas.height = img.height;
  ctx.drawImage(img, 0, 0);
  
  const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
  
  // ประมวลผล
  const grayscale = processor.toGrayscale(imageData);
  const blurred = processor.applyBlur(grayscale, 5);
  const edges = processor.detectEdges(blurred);
  
  ctx.putImageData(edges, 0, 0);
};
img.src = "/sample.jpg";
```

---

## Step 1642: WASM Cryptography

```javascript
// ใช้ WASM สำหรับ cryptographic operations

// ใช้ libsodium-wasm (library ยอดนิยม)
import sodium from "libsodium-wrappers";

async function cryptoExample() {
  await sodium.ready;
  
  // Key generation
  const keyPair = sodium.crypto_sign_keypair();
  console.log("Public Key:", sodium.to_hex(keyPair.publicKey));
  
  // Sign message
  const message = new TextEncoder().encode("ข้อความที่ต้องการเซ็น");
  const signature = sodium.crypto_sign_detached(message, keyPair.privateKey);
  
  // Verify signature
  const isValid = sodium.crypto_sign_verify_detached(
    signature,
    message,
    keyPair.publicKey
  );
  console.log("ลายเซ็นถูกต้อง:", isValid); // true
  
  // Symmetric encryption
  const key = sodium.crypto_secretbox_keygen();
  const nonce = sodium.randombytes_buf(sodium.crypto_secretbox_NONCEBYTES);
  
  const plaintext = new TextEncoder().encode("ข้อความลับ");
  const ciphertext = sodium.crypto_secretbox_easy(plaintext, nonce, key);
  
  // Decrypt
  const decrypted = sodium.crypto_secretbox_open_easy(ciphertext, nonce, key);
  console.log("Decrypted:", new TextDecoder().decode(decrypted));
  
  // Password hashing
  const password = "รหัสผ่านของฉัน";
  const hash = sodium.crypto_pwhash_str(
    password,
    sodium.crypto_pwhash_OPSLIMIT_INTERACTIVE,
    sodium.crypto_pwhash_MEMLIMIT_INTERACTIVE
  );
  
  // Verify password
  const passwordValid = sodium.crypto_pwhash_str_verify(hash, password);
  console.log("รหัสผ่านถูกต้อง:", passwordValid);
}

cryptoExample();
```

---

## Step 1643: WASM in Node.js

```javascript
// การใช้ WASM ใน Node.js

const fs = require("fs");
const path = require("path");

// โหลด WASM file
async function loadWasm(filename) {
  const wasmPath = path.join(__dirname, filename);
  const wasmBuffer = fs.readFileSync(wasmPath);
  
  const importObject = {
    env: {
      memory: new WebAssembly.Memory({ initial: 1 }),
      log: (value) => console.log("WASM:", value)
    }
  };
  
  const { instance } = await WebAssembly.instantiate(
    wasmBuffer,
    importObject
  );
  
  return instance.exports;
}

// ใช้กับ Node.js streams
const { createReadStream, createWriteStream } = require("fs");
const { pipeline } = require("stream/promises");

async function processLargeFile(inputPath, outputPath) {
  const wasm = await loadWasm("processor.wasm");
  
  const readStream = createReadStream(inputPath);
  const writeStream = createWriteStream(outputPath);
  
  let offset = 0;
  const chunkSize = 64 * 1024; // 64KB chunks
  
  for await (const chunk of readStream) {
    // ประมวลผลแต่ละ chunk ด้วย WASM
    const buffer = new Uint8Array(wasm.memory.buffer, offset, chunk.length);
    buffer.set(chunk);
    
    wasm.processChunk(offset, chunk.length);
    
    const result = new Uint8Array(wasm.memory.buffer, offset, chunk.length);
    writeStream.write(Buffer.from(result));
    
    offset = (offset + chunk.length) % (64 * 1024 * 1024); // circular buffer
  }
  
  writeStream.end();
}

// Worker threads + WASM
const { Worker, isMainThread, parentPort, workerData } = require("worker_threads");

if (isMainThread) {
  // Main thread: แจกงานให้ workers
  async function processParallel(data) {
    const numWorkers = require("os").cpus().length;
    const chunkSize = Math.ceil(data.length / numWorkers);
    
    const workers = Array.from({ length: numWorkers }, (_, i) => {
      const chunk = data.slice(i * chunkSize, (i + 1) * chunkSize);
      return new Promise((resolve, reject) => {
        const worker = new Worker(__filename, {
          workerData: { chunk }
        });
        worker.on("message", resolve);
        worker.on("error", reject);
      });
    });
    
    const results = await Promise.all(workers);
    return results.flat();
  }
} else {
  // Worker thread: ประมวลผลด้วย WASM
  (async () => {
    const wasm = await loadWasm("processor.wasm");
    const result = workerData.chunk.map(wasm.processItem);
    parentPort.postMessage(result);
  })();
}
```

---

## Step 1644: WASM Threads

```c
// C code สำหรับ WASM threads

#include <pthread.h>
#include <emscripten/emscripten.h>

// Parallel computation
typedef struct {
  double* data;
  int start;
  int end;
  double* result;
} WorkerArgs;

void* processChunk(void* args) {
  WorkerArgs* wa = (WorkerArgs*)args;
  double sum = 0;
  
  for (int i = wa->start; i < wa->end; i++) {
    sum += wa->data[i] * wa->data[i];
  }
  
  *wa->result = sum;
  return NULL;
}

EMSCRIPTEN_KEEPALIVE
double parallelSum(double* data, int length) {
  int numThreads = 4;
  pthread_t threads[4];
  WorkerArgs args[4];
  double results[4];
  
  int chunkSize = length / numThreads;
  
  for (int i = 0; i < numThreads; i++) {
    args[i].data = data;
    args[i].start = i * chunkSize;
    args[i].end = (i == numThreads - 1) ? length : (i + 1) * chunkSize;
    args[i].result = &results[i];
    
    pthread_create(&threads[i], NULL, processChunk, &args[i]);
  }
  
  for (int i = 0; i < numThreads; i++) {
    pthread_join(threads[i], NULL);
  }
  
  double total = 0;
  for (int i = 0; i < numThreads; i++) {
    total += results[i];
  }
  
  return total;
}
```

```bash
# Compile พร้อม threads
emcc threaded.c -o threaded.js \
  -s USE_PTHREADS=1 \
  -s PTHREAD_POOL_SIZE=4 \
  -s ALLOW_MEMORY_GROWTH=1 \
  -O3

# HTML ต้องมี headers:
# Cross-Origin-Embedder-Policy: require-corp
# Cross-Origin-Opener-Policy: same-origin
```

```javascript
// JavaScript สำหรับ WASM threads

// ต้องมี headers สำหรับ SharedArrayBuffer
// Cross-Origin-Embedder-Policy: require-corp
// Cross-Origin-Opener-Policy: same-origin

async function useWasmThreads() {
  const module = await createModule({
    // Specify shared memory
    wasmMemory: new WebAssembly.Memory({
      initial: 16,
      maximum: 256,
      shared: true
    })
  });
  
  const data = new Float64Array(1000000).fill(1.5);
  
  // Copy data to WASM memory
  const ptr = module._malloc(data.byteLength);
  module.HEAPF64.set(data, ptr / 8);
  
  console.time("Parallel Sum");
  const result = module._parallelSum(ptr, data.length);
  console.timeEnd("Parallel Sum");
  
  console.log("Sum:", result);
  module._free(ptr);
}
```

---

## Step 1645-1650: ตัวอย่างสมบูรณ์ - Physics Simulation

```javascript
// Physics simulation ด้วย WASM

// JavaScript fallback (ช้ากว่า)
class ParticleSystemJS {
  constructor(count) {
    this.count = count;
    this.x = new Float32Array(count);
    this.y = new Float32Array(count);
    this.vx = new Float32Array(count);
    this.vy = new Float32Array(count);
    this.mass = new Float32Array(count).fill(1);
    
    // random initial positions
    for (let i = 0; i < count; i++) {
      this.x[i] = Math.random() * 800;
      this.y[i] = Math.random() * 600;
      this.vx[i] = (Math.random() - 0.5) * 100;
      this.vy[i] = (Math.random() - 0.5) * 100;
    }
  }
  
  update(dt) {
    const GRAVITY = 9.8;
    const DAMPING = 0.99;
    
    for (let i = 0; i < this.count; i++) {
      // Apply gravity
      this.vy[i] += GRAVITY * dt;
      
      // Update position
      this.x[i] += this.vx[i] * dt;
      this.y[i] += this.vy[i] * dt;
      
      // Boundary collision
      if (this.x[i] < 0) { this.x[i] = 0; this.vx[i] *= -DAMPING; }
      if (this.x[i] > 800) { this.x[i] = 800; this.vx[i] *= -DAMPING; }
      if (this.y[i] < 0) { this.y[i] = 0; this.vy[i] *= -DAMPING; }
      if (this.y[i] > 600) { this.y[i] = 600; this.vy[i] *= -DAMPING; }
    }
    
    // Particle collisions (O(n²) - ช้ามากเมื่อ n ใหญ่)
    for (let i = 0; i < this.count; i++) {
      for (let j = i + 1; j < this.count; j++) {
        const dx = this.x[j] - this.x[i];
        const dy = this.y[j] - this.y[i];
        const dist = Math.sqrt(dx * dx + dy * dy);
        
        if (dist < 10) { // radius = 5
          // Simple elastic collision
          const nx = dx / dist;
          const ny = dy / dist;
          
          const dvx = this.vx[j] - this.vx[i];
          const dvy = this.vy[j] - this.vy[i];
          const dot = dvx * nx + dvy * ny;
          
          if (dot < 0) {
            this.vx[i] += dot * nx;
            this.vy[i] += dot * ny;
            this.vx[j] -= dot * nx;
            this.vy[j] -= dot * ny;
          }
        }
      }
    }
  }
}

// WASM version (เร็วกว่ามาก)
class ParticleSystemWASM {
  #wasm;
  #particlePtr;
  #count;
  
  async initialize(count) {
    this.#count = count;
    
    const { instance } = await WebAssembly.instantiateStreaming(
      fetch("/particles.wasm"),
      { env: { memory: new WebAssembly.Memory({ initial: 64 }) } }
    );
    
    this.#wasm = instance.exports;
    this.#particlePtr = this.#wasm.createParticles(count);
    this.#wasm.initRandom(this.#particlePtr, count, 800, 600);
  }
  
  update(dt) {
    this.#wasm.updateParticles(this.#particlePtr, this.#count, dt);
  }
  
  render(ctx) {
    const memory = this.#wasm.getMemory();
    const view = new Float32Array(memory.buffer, this.#particlePtr, this.#count * 4);
    
    ctx.clearRect(0, 0, 800, 600);
    ctx.fillStyle = "rgba(100, 200, 255, 0.8)";
    
    for (let i = 0; i < this.#count; i++) {
      const x = view[i * 4];
      const y = view[i * 4 + 1];
      
      ctx.beginPath();
      ctx.arc(x, y, 5, 0, Math.PI * 2);
      ctx.fill();
    }
  }
  
  destroy() {
    this.#wasm.freeParticles(this.#particlePtr);
  }
}

// Game loop
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");

const system = new ParticleSystemWASM();
await system.initialize(1000);

let lastTime = 0;
function gameLoop(time) {
  const dt = (time - lastTime) / 1000;
  lastTime = time;
  
  system.update(dt);
  system.render(ctx);
  
  requestAnimationFrame(gameLoop);
}

requestAnimationFrame(gameLoop);
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: WAT Calculator
เขียน WAT module ที่มี operations: add, subtract, multiply, divide, power, sqrt

### แบบฝึกหัดที่ 2: Rust String Processing
เขียน Rust/WASM function ที่:
- Count word frequency
- Find longest common substring
- Compress string (simple RLE)

### แบบฝึกหัดที่ 3: Image Filter
สร้าง image filter ด้วย C/Emscripten:
- Brightness/Contrast adjustment
- Color channel manipulation
- Pixelate effect

### แบบฝึกหัดที่ 4: Sorting Comparison
เปรียบเทียบ performance ของ:
- JavaScript quicksort
- WASM quicksort
สำหรับ arrays ขนาด 10K, 100K, 1M elements

### แบบฝึกหัดที่ 5: WASM Module Cache
สร้าง system ที่:
- Cache compiled WASM modules ใน IndexedDB
- Invalidate cache เมื่อ WASM file เปลี่ยน
- Load จาก cache เมื่อ available

---

## สรุป (Summary)

ใน Part 83 เราได้เรียนรู้ WebAssembly:

1. **WASM คืออะไร** - Binary format, use cases, pros/cons
2. **Performance** - เร็วกว่า JS 3-10x สำหรับ compute-heavy tasks
3. **WAT Format** - Human-readable WASM text format
4. **Emscripten** - Compile C/C++ เป็น WASM
5. **Rust + wasm-pack** - Compile Rust เป็น WASM
6. **AssemblyScript** - TypeScript-like language สำหรับ WASM
7. **Loading WASM** - instantiateStreaming, instantiate, modules
8. **Memory Management** - Linear memory, typed arrays
9. **JS <-> WASM Communication** - imports, exports, memory sharing
10. **wasm-bindgen** - High-level Rust-WASM bindings
11. **Image Processing** - Practical use case
12. **Cryptography** - libsodium-wasm
13. **Node.js** - WASM ใน server-side JS
14. **Threads** - WASM + SharedArrayBuffer + pthreads
15. **Physics Simulation** - Real-world example

WASM เป็นเครื่องมือที่ทรงพลังสำหรับ performance-critical applications
แต่ควรใช้เมื่อ JavaScript ไม่เพียงพอ ไม่ใช่แทนที่ JavaScript ทั้งหมด
