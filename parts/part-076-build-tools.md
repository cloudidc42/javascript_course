# Part 76: Build Tools - Webpack และ Vite (Steps 1491-1510)

## บทนำ: ทำไมถึงต้องใช้ Build Tools?

ในการพัฒนา JavaScript สมัยใหม่ เราไม่ได้เขียนโค้ดแล้วนำไปใช้งานโดยตรงอีกต่อไป เราต้องการ **Build Tools** เพื่อแปลงโค้ดของเราให้พร้อมสำหรับการใช้งานในโปรดักชัน

### ปัญหาที่ Build Tools แก้ไข

1. **Module System**: เบราว์เซอร์เก่าไม่รองรับ ES Modules
2. **Modern Syntax**: ต้องการแปลง TypeScript, JSX เป็น JavaScript ธรรมดา
3. **Performance**: ต้องรวม minify และ optimize โค้ด
4. **CSS Processing**: ต้องการ Sass, PostCSS, CSS Modules
5. **Asset Optimization**: รูปภาพ, fonts ต้องการการจัดการพิเศษ
6. **Developer Experience**: Hot reload, source maps สำหรับ debugging

```javascript
// โค้ดก่อน Build (modern JavaScript)
import { createApp } from 'vue'
import App from './App.vue'
import './styles/main.scss'

const app = createApp(App)
app.mount('#app')

// หลัง Build: โค้ดถูกรวม, minify, และพร้อมสำหรับ production
// (function(){...})() // แบบ IIFE ที่ browsers ทุกตัวเข้าใจได้
```

---

## Step 1491: แนวคิด Bundling

**Bundling** คือกระบวนการรวมไฟล์ JavaScript หลายๆ ไฟล์ให้เป็นไฟล์เดียว (หรือไม่กี่ไฟล์) เพื่อลด HTTP requests

### ทำไมต้อง Bundle?

```
// โครงสร้างโปรเจคของคุณ
src/
├── index.js        (import จาก utils, components)
├── utils.js        (import จาก lodash)
├── components/
│   ├── Header.js
│   ├── Footer.js
│   └── Button.js
└── styles/
    └── main.css

// โดยไม่มี Bundler: Browser ต้องทำ HTTP requests หลายครั้ง
// index.js -> utils.js -> lodash -> Header.js -> Footer.js -> Button.js
// = 6+ HTTP requests

// หลัง Bundle: ไฟล์เดียว
dist/
└── bundle.js  // ทุกอย่างอยู่ที่นี่
```

### Dependency Graph

Bundler จะวิเคราะห์ dependency graph ของโปรเจค:

```javascript
// entry.js
import { add } from './math'     // edge: entry -> math
import { format } from './utils' // edge: entry -> utils

// math.js
export function add(a, b) { return a + b }
export function subtract(a, b) { return a - b }

// utils.js
import { format } from 'date-fns' // edge: utils -> date-fns (node_modules)
export function format(date) { ... }
```

---

## Step 1492: Webpack - แนวคิดพื้นฐาน

**Webpack** เป็น Static Module Bundler ที่ได้รับความนิยมมากที่สุดตั้งแต่ปี 2014

### ติดตั้ง Webpack

```bash
# สร้างโปรเจคใหม่
mkdir my-webpack-project
cd my-webpack-project
npm init -y

# ติดตั้ง webpack
npm install --save-dev webpack webpack-cli

# ตรวจสอบ version
npx webpack --version
```

### โครงสร้างพื้นฐาน

```
my-webpack-project/
├── src/
│   └── index.js      ← Entry point
├── dist/
│   └── main.js       ← Output (auto-generated)
├── webpack.config.js ← Configuration
└── package.json
```

---

## Step 1493: Entry Point และ Output

### Entry Point

**Entry** คือจุดเริ่มต้นที่ webpack เริ่มสร้าง dependency graph

```javascript
// webpack.config.js

const path = require('path')

module.exports = {
  // Single entry
  entry: './src/index.js',
  
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
    clean: true, // ลบ dist ก่อน build ทุกครั้ง
  },
}
```

### Multiple Entry Points

```javascript
// webpack.config.js - หลาย entry points

module.exports = {
  entry: {
    app: './src/app.js',
    admin: './src/admin.js',
    vendor: './src/vendor.js',
  },
  
  output: {
    filename: '[name].bundle.js', // จะได้ app.bundle.js, admin.bundle.js
    path: path.resolve(__dirname, 'dist'),
    clean: true,
  },
}
```

### Output Configuration

```javascript
module.exports = {
  entry: './src/index.js',
  
  output: {
    // ชื่อไฟล์ output
    filename: '[name].[contenthash].js', // hash สำหรับ cache busting
    
    // โฟลเดอร์ output
    path: path.resolve(__dirname, 'dist'),
    
    // Public path สำหรับ assets
    publicPath: '/',
    
    // ลบ dist ก่อน build
    clean: true,
    
    // Library format (สำหรับ npm packages)
    library: {
      name: 'MyLibrary',
      type: 'umd',
    },
  },
}
```

---

## Step 1494: Loaders

**Loaders** ช่วยให้ webpack สามารถ process ไฟล์ที่ไม่ใช่ JavaScript เช่น CSS, TypeScript, รูปภาพ

### แนวคิด Loaders

```
// Webpack เข้าใจแค่ JavaScript และ JSON
// Loaders แปลงไฟล์อื่นๆ เป็น modules ที่ webpack เข้าใจได้

import './styles.css'    // ← Loader แปลง CSS → JS module
import logo from './logo.png'  // ← Loader แปลงรูปภาพ → URL/data URI
import { greet } from './greet.ts'  // ← TypeScript → JavaScript
```

### Babel Loader (แปลง Modern JS)

```bash
npm install --save-dev babel-loader @babel/core @babel/preset-env
```

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.m?js$/,          // ไฟล์ที่ match
        exclude: /node_modules/,  // ยกเว้น node_modules
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env']
          }
        }
      }
    ]
  }
}
```

```javascript
// babel.config.js
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        targets: {
          browsers: ['> 1%', 'last 2 versions', 'not dead'],
        },
        useBuiltIns: 'usage',
        corejs: 3,
      },
    ],
  ],
}
```

### CSS Loader และ Style Loader

```bash
npm install --save-dev css-loader style-loader
```

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [
          'style-loader', // 2. Inject CSS into DOM
          'css-loader',   // 1. Process @import และ url()
        ],
        // Note: Loaders execute right-to-left
      },
    ],
  },
}
```

```javascript
// src/index.js
import './styles.css'  // ทำงานได้แล้ว!

// styles.css จะถูก inject เข้าไปใน <head> ตอน runtime
```

### Sass Loader

```bash
npm install --save-dev sass-loader sass
```

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.s[ac]ss$/i,
        use: [
          'style-loader',
          'css-loader',
          'sass-loader', // แปลง Sass → CSS ก่อน
        ],
      },
    ],
  },
}
```

```scss
/* src/styles/main.scss */
$primary-color: #3498db;
$font-stack: 'Helvetica', sans-serif;

body {
  font: 100% $font-stack;
  color: $primary-color;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  
  &__header {
    background: darken($primary-color, 10%);
  }
}
```

### File Loader / Asset Modules

```javascript
// webpack.config.js (Webpack 5 - Asset Modules)
module.exports = {
  module: {
    rules: [
      // รูปภาพ
      {
        test: /\.(png|jpg|gif|svg)$/i,
        type: 'asset/resource', // ก็อปไฟล์ไปยัง output
      },
      
      // รูปภาพขนาดเล็ก (inline เป็น base64)
      {
        test: /\.(png|jpg|gif)$/i,
        type: 'asset', // ตัดสินใจอัตโนมัติ
        parser: {
          dataUrlCondition: {
            maxSize: 8 * 1024, // 8kb → inline, ใหญ่กว่า → file
          }
        }
      },
      
      // Fonts
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/i,
        type: 'asset/resource',
      },
    ],
  },
}
```

### TypeScript Loader

```bash
npm install --save-dev ts-loader typescript
```

```javascript
// webpack.config.js
module.exports = {
  entry: './src/index.ts',
  
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  
  resolve: {
    extensions: ['.tsx', '.ts', '.js'], // ลำดับการ resolve
  },
}
```

---

## Step 1495: Plugins

**Plugins** ทำสิ่งที่ Loaders ทำไม่ได้ - เช่น การสร้าง HTML, การ optimize bundle, การ extract CSS

### HtmlWebpackPlugin

```bash
npm install --save-dev html-webpack-plugin
```

```javascript
// webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html', // ใช้ template ของเรา
      filename: 'index.html',
      title: 'My App',
      
      // Minify HTML ใน production
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeRedundantAttributes: true,
      },
    }),
  ],
}
```

```html
<!-- src/index.html (template) -->
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title><%= htmlWebpackPlugin.options.title %></title>
</head>
<body>
  <div id="app"></div>
  <!-- Webpack จะ inject <script> tags อัตโนมัติ -->
</body>
</html>
```

### MiniCssExtractPlugin

แยก CSS ออกเป็นไฟล์แยกต่างหาก (แทน style-loader ที่ inject ใน `<style>`)

```bash
npm install --save-dev mini-css-extract-plugin
```

```javascript
// webpack.config.js
const MiniCssExtractPlugin = require('mini-css-extract-plugin')

module.exports = {
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].[contenthash].css',
    }),
  ],
  
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [
          MiniCssExtractPlugin.loader, // แทน style-loader
          'css-loader',
        ],
      },
    ],
  },
}
```

### DefinePlugin

กำหนด global constants ใน build time

```javascript
const webpack = require('webpack')

module.exports = {
  plugins: [
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
      'process.env.API_URL': JSON.stringify(process.env.API_URL),
      VERSION: JSON.stringify('1.0.0'),
      IS_PRODUCTION: process.env.NODE_ENV === 'production',
    }),
  ],
}
```

```javascript
// ในโค้ด
if (process.env.NODE_ENV === 'production') {
  console.log('Running in production')
}

console.log('API URL:', process.env.API_URL)
console.log('Version:', VERSION)
```

### CopyWebpackPlugin

```bash
npm install --save-dev copy-webpack-plugin
```

```javascript
const CopyPlugin = require('copy-webpack-plugin')

module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        { from: 'public', to: 'dist' }, // ก็อป public → dist
        { from: 'assets', to: 'assets' },
      ],
    }),
  ],
}
```

---

## Step 1496: Mode - Development vs Production

```javascript
// webpack.config.js
module.exports = (env, argv) => {
  const isDevelopment = argv.mode !== 'production'
  
  return {
    mode: argv.mode || 'development', // 'development' | 'production' | 'none'
    
    // Development: ไม่ minify, มี source maps ที่ดี
    // Production: minify, optimize, tree shake
    
    devtool: isDevelopment ? 'eval-source-map' : 'source-map',
    
    optimization: {
      minimize: !isDevelopment,
    },
  }
}
```

```json
// package.json
{
  "scripts": {
    "build": "webpack --mode=production",
    "dev": "webpack --mode=development",
    "watch": "webpack --mode=development --watch"
  }
}
```

---

## Step 1497: Source Maps

Source Maps ช่วยให้ debug โค้ดที่ถูก bundle และ minify ได้

```javascript
// webpack.config.js
module.exports = {
  // ตัวเลือก source map
  devtool: 'eval-source-map',  // Development: เร็ว, ข้อมูลครบ
  // devtool: 'source-map',    // Production: ไฟล์แยก .map
  // devtool: 'inline-source-map', // Inline ใน bundle (ไฟล์ใหญ่)
  // devtool: false,           // ไม่มี source map
}
```

### ตาราง Source Map Options

| Option | Build | Rebuild | Quality | ใช้เมื่อ |
|--------|-------|---------|---------|----------|
| eval | ++ | +++ | Generated | Dev - เร็วสุด |
| eval-source-map | - | ++ | Original | Dev - แนะนำ |
| source-map | -- | -- | Original | Production |
| hidden-source-map | -- | -- | Original | Production (ไม่แสดง) |
| nosources-source-map | -- | -- | Without source content | Production |

---

## Step 1498: webpack-dev-server และ HMR

### webpack-dev-server

```bash
npm install --save-dev webpack-dev-server
```

```javascript
// webpack.config.js
module.exports = {
  devServer: {
    static: {
      directory: path.join(__dirname, 'public'),
    },
    
    port: 3000,
    
    hot: true,           // Hot Module Replacement
    
    open: true,          // เปิด browser อัตโนมัติ
    
    compress: true,      // gzip compression
    
    historyApiFallback: true, // สำหรับ SPA routing
    
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        pathRewrite: { '^/api': '' },
      },
    },
    
    headers: {
      'X-Custom-Header': 'yes',
    },
  },
}
```

```json
// package.json
{
  "scripts": {
    "dev": "webpack serve --mode=development",
    "build": "webpack --mode=production"
  }
}
```

### Hot Module Replacement (HMR)

HMR อัปเดตโค้ดในเบราว์เซอร์โดยไม่ต้อง reload หน้า

```javascript
// src/index.js
import { greet } from './greet'

function render() {
  document.getElementById('app').innerHTML = greet('World')
}

render()

// รับ HMR updates
if (module.hot) {
  module.hot.accept('./greet', () => {
    // เมื่อ greet.js เปลี่ยน → ไม่ต้อง full reload
    render()
  })
}
```

---

## Step 1499: Code Splitting

Code Splitting แบ่ง bundle เป็นหลายๆ ชิ้น โหลดเฉพาะที่ต้องการ

### Dynamic Import

```javascript
// แทนที่จะ import ทั้งหมดตั้งแต่ต้น
// import HeavyChart from './HeavyChart'  ← โหลดทันที

// ใช้ dynamic import แทน
async function loadChart() {
  // โหลดเฉพาะตอนที่ต้องการ
  const { default: HeavyChart } = await import('./HeavyChart')
  return new HeavyChart()
}

// React lazy loading
const LazyComponent = React.lazy(() => import('./LazyComponent'))

function App() {
  return (
    <React.Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </React.Suspense>
  )
}
```

### SplitChunksPlugin

```javascript
// webpack.config.js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',          // ทุก chunks
      
      cacheGroups: {
        // แยก vendor code (node_modules)
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
          priority: 10,
        },
        
        // แยก common code ที่ใช้ซ้ำ
        common: {
          minChunks: 2,       // ใช้ใน 2 chunks ขึ้นไป
          chunks: 'async',
          priority: 5,
          name: 'common',
          reuseExistingChunk: true,
        },
      },
    },
    
    // แยก webpack runtime
    runtimeChunk: 'single',
  },
}
```

---

## Step 1500: Tree Shaking

**Tree Shaking** คือการลบ dead code (export ที่ไม่ถูก import) ออกจาก bundle

```javascript
// math.js
export function add(a, b) { return a + b }      // ← ใช้งาน → เก็บไว้
export function subtract(a, b) { return a - b } // ← ไม่ได้ใช้ → ลบออก
export function multiply(a, b) { return a * b } // ← ไม่ได้ใช้ → ลบออก

// index.js
import { add } from './math'  // import แค่ add
console.log(add(1, 2))

// หลัง tree shaking:
// subtract และ multiply ถูกลบออกจาก bundle
```

### เงื่อนไขสำหรับ Tree Shaking

```javascript
// ✅ ทำงาน: ES Module static imports
import { add } from './math'

// ❌ ไม่ทำงาน: CommonJS
const { add } = require('./math')

// ❌ ไม่ทำงาน: Dynamic imports
const module = await import('./math')
const fn = module[functionName]
```

```json
// package.json
{
  "sideEffects": false,     // บอก webpack ว่าทุกไฟล์ไม่มี side effects
  
  // หรือระบุไฟล์ที่มี side effects
  "sideEffects": [
    "*.css",                // CSS มี side effects (inject ใน DOM)
    "./src/polyfills.js"
  ]
}
```

---

## Step 1501: webpack.config.js ครบถ้วน

```javascript
// webpack.config.js (Production-ready)
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin')
const TerserPlugin = require('terser-webpack-plugin')
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer')

module.exports = (env, argv) => {
  const isDev = argv.mode === 'development'
  const isProd = !isDev
  
  return {
    mode: argv.mode || 'development',
    
    entry: {
      main: './src/index.js',
    },
    
    output: {
      filename: isProd ? '[name].[contenthash:8].js' : '[name].js',
      chunkFilename: isProd ? '[name].[contenthash:8].chunk.js' : '[name].chunk.js',
      path: path.resolve(__dirname, 'dist'),
      publicPath: '/',
      clean: true,
    },
    
    devtool: isDev ? 'eval-source-map' : 'source-map',
    
    resolve: {
      extensions: ['.js', '.jsx', '.ts', '.tsx'],
      alias: {
        '@': path.resolve(__dirname, 'src'),
        '@components': path.resolve(__dirname, 'src/components'),
      },
    },
    
    module: {
      rules: [
        // JavaScript / TypeScript
        {
          test: /\.[jt]sx?$/,
          exclude: /node_modules/,
          use: {
            loader: 'babel-loader',
            options: {
              cacheDirectory: true,
              presets: [
                ['@babel/preset-env', { targets: 'defaults' }],
                ['@babel/preset-react', { runtime: 'automatic' }],
                '@babel/preset-typescript',
              ],
            },
          },
        },
        
        // CSS
        {
          test: /\.css$/i,
          use: [
            isDev ? 'style-loader' : MiniCssExtractPlugin.loader,
            {
              loader: 'css-loader',
              options: {
                modules: {
                  auto: true, // *.module.css → CSS Modules
                },
              },
            },
            'postcss-loader',
          ],
        },
        
        // SCSS
        {
          test: /\.s[ac]ss$/i,
          use: [
            isDev ? 'style-loader' : MiniCssExtractPlugin.loader,
            'css-loader',
            'sass-loader',
          ],
        },
        
        // Images
        {
          test: /\.(png|jpg|jpeg|gif|webp)$/i,
          type: 'asset',
          parser: {
            dataUrlCondition: { maxSize: 8 * 1024 },
          },
          generator: {
            filename: 'images/[name].[hash:8][ext]',
          },
        },
        
        // SVG
        {
          test: /\.svg$/i,
          issuer: /\.[jt]sx?$/,
          use: ['@svgr/webpack'],
        },
        
        // Fonts
        {
          test: /\.(woff|woff2|eot|ttf|otf)$/i,
          type: 'asset/resource',
          generator: {
            filename: 'fonts/[name].[hash:8][ext]',
          },
        },
      ],
    },
    
    plugins: [
      new HtmlWebpackPlugin({
        template: './src/index.html',
        title: 'My App',
        favicon: './src/favicon.ico',
      }),
      
      isProd && new MiniCssExtractPlugin({
        filename: 'styles/[name].[contenthash:8].css',
        chunkFilename: 'styles/[name].[contenthash:8].chunk.css',
      }),
      
      // Analyze bundle (เปิดเมื่อต้องการ)
      process.env.ANALYZE && new BundleAnalyzerPlugin(),
    ].filter(Boolean),
    
    optimization: {
      minimize: isProd,
      minimizer: [
        new TerserPlugin({
          terserOptions: {
            compress: { drop_console: isProd },
          },
        }),
        new CssMinimizerPlugin(),
      ],
      
      splitChunks: {
        chunks: 'all',
        cacheGroups: {
          vendor: {
            test: /[\\/]node_modules[\\/]/,
            name: 'vendors',
            chunks: 'all',
          },
        },
      },
      
      runtimeChunk: 'single',
    },
    
    devServer: {
      hot: true,
      port: 3000,
      open: true,
      historyApiFallback: true,
      proxy: {
        '/api': 'http://localhost:8080',
      },
    },
    
    performance: {
      hints: isProd ? 'warning' : false,
      maxEntrypointSize: 512000,
      maxAssetSize: 512000,
    },
    
    stats: isDev ? 'errors-warnings' : 'normal',
  }
}
```

---

## Step 1502: Vite - แนวคิดพื้นฐาน

**Vite** (อ่านว่า "วีต") เป็น Build Tool รุ่นใหม่ที่เร็วกว่า Webpack มาก เพราะใช้ Native ESM ของเบราว์เซอร์ในการพัฒนา

### ทำไม Vite ถึงเร็วกว่า?

```
Webpack Dev Server:
┌─────────────────────────────────────────────┐
│  เริ่มต้น: Bundle ทุกไฟล์ก่อน start server │
│  ├── Entry point                             │
│  ├── Parse all imports                       │
│  ├── Bundle everything                       │
│  └── Serve bundle                           │
│  เวลา: 30-60 วินาที (โปรเจคใหญ่)           │
└─────────────────────────────────────────────┘

Vite Dev Server:
┌─────────────────────────────────────────────┐
│  เริ่มต้น: Pre-bundle node_modules เท่านั้น │
│  ├── Dependencies → esbuild (เร็วมาก)       │
│  └── Source files → serve as-is (ESM)       │
│  เวลา: < 300ms                              │
│  เมื่อ browser request ไฟล์ → transform     │
└─────────────────────────────────────────────┘
```

### สร้างโปรเจค Vite

```bash
# สร้างโปรเจคใหม่
npm create vite@latest my-vite-app

# หรือระบุ template
npm create vite@latest my-react-app -- --template react-ts
npm create vite@latest my-vue-app -- --template vue-ts
npm create vite@latest my-vanilla-app -- --template vanilla

# เข้าไปและ install
cd my-react-app
npm install

# เริ่ม dev server
npm run dev   # http://localhost:5173

# Build
npm run build

# Preview build
npm run preview
```

### โครงสร้าง Vite Project

```
my-react-app/
├── src/
│   ├── App.tsx
│   ├── main.tsx        ← Entry point
│   └── index.css
├── public/
│   └── vite.svg
├── index.html          ← Entry HTML (สำคัญ: อยู่ที่ root!)
├── vite.config.ts
├── tsconfig.json
└── package.json
```

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My Vite App</title>
</head>
<body>
  <div id="root"></div>
  <script type="module" src="/src/main.tsx"></script>
  <!-- type="module" → ESM! -->
</body>
</html>
```

---

## Step 1503: vite.config.ts

```typescript
// vite.config.ts
import { defineConfig, loadEnv } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig(({ command, mode }) => {
  const env = loadEnv(mode, process.cwd(), '')
  const isDev = command === 'serve'
  
  return {
    plugins: [
      react({
        // Fast Refresh options
        fastRefresh: true,
      }),
    ],
    
    // Resolve aliases
    resolve: {
      alias: {
        '@': path.resolve(__dirname, 'src'),
        '@components': path.resolve(__dirname, 'src/components'),
        '@hooks': path.resolve(__dirname, 'src/hooks'),
        '@utils': path.resolve(__dirname, 'src/utils'),
      },
    },
    
    // Dev server
    server: {
      port: 3000,
      open: true,
      proxy: {
        '/api': {
          target: 'http://localhost:8080',
          changeOrigin: true,
          rewrite: (path) => path.replace(/^\/api/, ''),
        },
      },
    },
    
    // Build options
    build: {
      outDir: 'dist',
      sourcemap: isDev,
      
      rollupOptions: {
        output: {
          // Code splitting
          manualChunks: {
            vendor: ['react', 'react-dom'],
            router: ['react-router-dom'],
          },
        },
      },
      
      // Chunk size warnings
      chunkSizeWarningLimit: 1000,
    },
    
    // CSS options
    css: {
      modules: {
        localsConvention: 'camelCaseOnly',
      },
      preprocessorOptions: {
        scss: {
          additionalData: '@import "@/styles/variables.scss";',
        },
      },
    },
    
    // Define global constants
    define: {
      __APP_VERSION__: JSON.stringify('1.0.0'),
      __DEV__: isDev,
    },
    
    // Optimize dependencies
    optimizeDeps: {
      include: ['react', 'react-dom'],
      exclude: ['my-local-package'],
    },
  }
})
```

---

## Step 1504: Vite Plugins

```typescript
// vite.config.ts - ตัวอย่าง Plugins ต่างๆ
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import vue from '@vitejs/plugin-vue'
import { VitePWA } from 'vite-plugin-pwa'
import viteCompression from 'vite-plugin-compression'
import { visualizer } from 'rollup-plugin-visualizer'

export default defineConfig({
  plugins: [
    // Framework plugins
    react(),  // หรือ vue()
    
    // PWA support
    VitePWA({
      registerType: 'autoUpdate',
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg}'],
      },
    }),
    
    // Gzip compression
    viteCompression({
      algorithm: 'gzip',
      ext: '.gz',
    }),
    
    // Bundle visualization
    process.env.ANALYZE && visualizer({
      open: true,
      gzipSize: true,
    }),
  ].filter(Boolean),
})
```

### สร้าง Custom Plugin

```typescript
// plugins/myPlugin.ts
import type { Plugin } from 'vite'

function myPlugin(): Plugin {
  return {
    name: 'my-plugin',
    
    // Hook: เมื่อ build เริ่มต้น
    buildStart() {
      console.log('Build started!')
    },
    
    // Hook: แปลงไฟล์
    transform(code, id) {
      if (!id.endsWith('.js')) return null
      
      // เพิ่ม comment ใน production build
      if (process.env.NODE_ENV === 'production') {
        return `/* Built by Vite */\n${code}`
      }
      
      return null
    },
    
    // Hook: เมื่อ build เสร็จ
    buildEnd() {
      console.log('Build complete!')
    },
    
    // Dev server hook
    configureServer(server) {
      server.middlewares.use('/health', (req, res) => {
        res.end('OK')
      })
    },
  }
}

export default myPlugin
```

---

## Step 1505: Vite - Environment Variables

```bash
# .env (ทุก environments)
VITE_API_URL=https://api.example.com
VITE_APP_TITLE=My App

# .env.development
VITE_API_URL=http://localhost:8080
VITE_DEBUG=true

# .env.production
VITE_API_URL=https://api.example.com
VITE_DEBUG=false

# .env.local (local override, ไม่ commit)
VITE_API_KEY=my-secret-key
```

```typescript
// ใช้ใน code - ต้องขึ้นต้นด้วย VITE_
const apiUrl = import.meta.env.VITE_API_URL
const isProduction = import.meta.env.PROD
const isDevelopment = import.meta.env.DEV
const mode = import.meta.env.MODE  // 'development' | 'production'
const baseUrl = import.meta.env.BASE_URL

// TypeScript type safety
/// <reference types="vite/client" />
interface ImportMetaEnv {
  readonly VITE_API_URL: string
  readonly VITE_APP_TITLE: string
}

interface ImportMeta {
  readonly env: ImportMetaEnv
}
```

---

## Step 1506: Vite - Multi-Page App

```typescript
// vite.config.ts - Multi-Page Application
import { defineConfig } from 'vite'
import { resolve } from 'path'

export default defineConfig({
  build: {
    rollupOptions: {
      input: {
        main: resolve(__dirname, 'index.html'),
        admin: resolve(__dirname, 'admin/index.html'),
        blog: resolve(__dirname, 'blog/index.html'),
      },
    },
  },
})
```

```
โครงสร้างโปรเจค Multi-Page:
├── index.html          ← หน้าหลัก
├── admin/
│   └── index.html      ← Admin panel
├── blog/
│   └── index.html      ← Blog
├── src/
│   ├── main.ts
│   ├── admin.ts
│   └── blog.ts
└── vite.config.ts
```

---

## Step 1507: Vite - Library Mode

```typescript
// vite.config.ts - สร้าง npm package
import { defineConfig } from 'vite'
import { resolve } from 'path'
import dts from 'vite-plugin-dts'

export default defineConfig({
  plugins: [
    dts({  // สร้าง .d.ts files
      insertTypesEntry: true,
    }),
  ],
  
  build: {
    lib: {
      entry: resolve(__dirname, 'src/index.ts'),
      name: 'MyLibrary',
      formats: ['es', 'cjs', 'umd'],
      fileName: (format) => `my-library.${format}.js`,
    },
    
    rollupOptions: {
      // ไม่ bundle dependencies
      external: ['react', 'react-dom'],
      
      output: {
        globals: {
          react: 'React',
          'react-dom': 'ReactDOM',
        },
      },
    },
  },
})
```

```json
// package.json สำหรับ library
{
  "name": "my-library",
  "version": "1.0.0",
  "main": "./dist/my-library.cjs.js",
  "module": "./dist/my-library.es.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/my-library.es.js",
      "require": "./dist/my-library.cjs.js"
    }
  },
  "files": ["dist"]
}
```

---

## Step 1508: Babel - Transpiling Modern JavaScript

**Babel** แปลง JavaScript ใหม่ให้ทำงานบนเบราว์เซอร์เก่า

```bash
npm install --save-dev @babel/core @babel/cli @babel/preset-env
npm install --save-dev @babel/preset-react @babel/preset-typescript
npm install --save-dev @babel/plugin-proposal-decorators
npm install core-js
```

```javascript
// babel.config.js (ครบถ้วน)
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        targets: {
          // เป้าหมาย browser
          browsers: ['> 0.5%', 'last 2 versions', 'Firefox ESR', 'not dead'],
          // หรือ Node.js
          node: 'current',
        },
        
        // Polyfills
        useBuiltIns: 'usage', // เพิ่ม polyfills เฉพาะที่ใช้
        corejs: 3,
        
        // ไม่ transform modules (webpack/vite จัดการเอง)
        modules: false,
      },
    ],
    
    // TypeScript support
    '@babel/preset-typescript',
    
    // React support
    [
      '@babel/preset-react',
      {
        runtime: 'automatic', // React 17+ (ไม่ต้อง import React)
        development: process.env.NODE_ENV === 'development',
      },
    ],
  ],
  
  plugins: [
    // Class decorators (MobX, TypeORM)
    ['@babel/plugin-proposal-decorators', { legacy: true }],
    
    // Optional chaining
    '@babel/plugin-proposal-optional-chaining',
    
    // Nullish coalescing
    '@babel/plugin-proposal-nullish-coalescing-operator',
  ],
  
  env: {
    test: {
      presets: [
        ['@babel/preset-env', { targets: { node: 'current' } }],
      ],
    },
  },
}
```

### ตัวอย่าง Babel Transform

```javascript
// โค้ดก่อน Babel (Modern JS)
const greet = async (name) => {
  const user = await fetchUser(name)
  const { firstName, lastName } = user ?? {}
  return `Hello ${firstName?.toUpperCase() ?? 'Guest'}!`
}

class Component {
  @observable count = 0
  
  #privateMethod() {
    return this.count
  }
}

// หลัง Babel (ES5-compatible)
"use strict";

var _asyncToGenerator = require("@babel/runtime/helpers/asyncToGenerator");
var _classCallCheck = require("@babel/runtime/helpers/classCallCheck");

var greet = function() {
  var _ref = _asyncToGenerator(function* (name) {
    var user = yield fetchUser(name);
    var _ref2 = user !== null && user !== void 0 ? user : {};
    var firstName = _ref2.firstName;
    var lastName = _ref2.lastName;
    return "Hello " + ((_firstName = firstName) === null || _firstName === void 0
      ? void 0 : _firstName.toUpperCase()) + " " + "World" + "!";
  });
  return function greet(_x) {
    return _ref.apply(this, arguments);
  };
}();
```

---

## Step 1509: ESLint Configuration

**ESLint** ตรวจสอบคุณภาพโค้ดโดยอัตโนมัติ

```bash
# ติดตั้ง ESLint
npm install --save-dev eslint
npx eslint --init  # สร้าง config แบบ interactive

# หรือติดตั้ง packages โดยตรง
npm install --save-dev eslint
npm install --save-dev @eslint/js eslint-plugin-react eslint-plugin-react-hooks
npm install --save-dev typescript-eslint
npm install --save-dev eslint-config-prettier  # ป้องกัน conflict กับ Prettier
```

```javascript
// eslint.config.js (ESLint 9 flat config)
import js from '@eslint/js'
import globals from 'globals'
import reactHooks from 'eslint-plugin-react-hooks'
import reactRefresh from 'eslint-plugin-react-refresh'
import tseslint from 'typescript-eslint'
import prettierConfig from 'eslint-config-prettier'

export default tseslint.config(
  { ignores: ['dist', 'node_modules'] },
  
  {
    extends: [
      js.configs.recommended,
      ...tseslint.configs.recommended,
    ],
    
    files: ['**/*.{ts,tsx}'],
    
    languageOptions: {
      ecmaVersion: 2020,
      globals: globals.browser,
    },
    
    plugins: {
      'react-hooks': reactHooks,
      'react-refresh': reactRefresh,
    },
    
    rules: {
      // React Hooks rules
      ...reactHooks.configs.recommended.rules,
      
      // React Refresh
      'react-refresh/only-export-components': [
        'warn',
        { allowConstantExport: true },
      ],
      
      // TypeScript rules
      '@typescript-eslint/no-explicit-any': 'warn',
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      '@typescript-eslint/explicit-function-return-type': 'off',
      
      // General rules
      'no-console': ['warn', { allow: ['warn', 'error'] }],
      'prefer-const': 'error',
      'no-var': 'error',
      'eqeqeq': ['error', 'always'],
      
      // Disable conflicting rules (Prettier จัดการแทน)
      ...prettierConfig.rules,
    },
  },
)
```

```json
// package.json scripts
{
  "scripts": {
    "lint": "eslint src --ext .ts,.tsx,.js,.jsx",
    "lint:fix": "eslint src --ext .ts,.tsx,.js,.jsx --fix",
    "lint:ci": "eslint src --ext .ts,.tsx --max-warnings 0"
  }
}
```

---

## Step 1510: Prettier, Husky และ lint-staged

### Prettier

**Prettier** จัดรูปแบบโค้ดให้สม่ำเสมอ

```bash
npm install --save-dev prettier
```

```json
// .prettierrc
{
  "semi": false,
  "singleQuote": true,
  "trailingComma": "es5",
  "tabWidth": 2,
  "printWidth": 100,
  "arrowParens": "avoid",
  "endOfLine": "lf",
  "bracketSpacing": true,
  "jsxSingleQuote": false,
  "overrides": [
    {
      "files": "*.md",
      "options": {
        "printWidth": 80,
        "proseWrap": "always"
      }
    }
  ]
}
```

```
# .prettierignore
node_modules
dist
build
coverage
*.min.js
*.min.css
public
```

### Husky - Pre-commit Hooks

**Husky** รัน scripts ก่อน git commit

```bash
# ติดตั้ง Husky
npm install --save-dev husky
npx husky init

# สร้าง pre-commit hook
echo "npx lint-staged" > .husky/pre-commit
```

```bash
# .husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npx lint-staged
```

```bash
# .husky/commit-msg
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

# ตรวจสอบ commit message format
npx --no -- commitlint --edit ${1}
```

### lint-staged

**lint-staged** รัน linters เฉพาะไฟล์ที่ staged

```json
// package.json
{
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{js,jsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{css,scss}": [
      "prettier --write"
    ],
    "*.{json,md,yml,yaml}": [
      "prettier --write"
    ]
  }
}
```

### commitlint

```bash
npm install --save-dev @commitlint/cli @commitlint/config-conventional
```

```javascript
// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',    // ฟีเจอร์ใหม่
        'fix',     // แก้ bug
        'docs',    // แก้ documentation
        'style',   // formatting (ไม่ใช่ CSS)
        'refactor',// refactor code
        'perf',    // performance improvement
        'test',    // เพิ่ม/แก้ tests
        'build',   // build system
        'ci',      // CI configuration
        'chore',   // อื่นๆ
        'revert',  // revert commit
      ],
    ],
    'subject-max-length': [2, 'always', 72],
  },
}
```

### ตัวอย่าง package.json ครบถ้วน

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "lint": "eslint src --ext .ts,.tsx",
    "lint:fix": "eslint src --ext .ts,.tsx --fix",
    "format": "prettier --write src",
    "format:check": "prettier --check src",
    "typecheck": "tsc --noEmit",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage",
    "prepare": "husky"
  },
  "devDependencies": {
    "@babel/core": "^7.23.0",
    "@babel/preset-env": "^7.23.0",
    "@babel/preset-react": "^7.22.0",
    "@babel/preset-typescript": "^7.23.0",
    "@commitlint/cli": "^18.0.0",
    "@commitlint/config-conventional": "^18.0.0",
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "@vitejs/plugin-react": "^4.2.0",
    "eslint": "^8.54.0",
    "eslint-config-prettier": "^9.0.0",
    "husky": "^8.0.3",
    "lint-staged": "^15.1.0",
    "prettier": "^3.1.0",
    "typescript": "^5.2.2",
    "vite": "^5.0.0",
    "vitest": "^1.0.0"
  }
}
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง Webpack Config จากศูนย์

สร้าง webpack project ที่รองรับ:
- TypeScript + React
- CSS Modules
- สร้าง HTML อัตโนมัติ
- Optimize สำหรับ production

```bash
# โครงสร้างที่ต้องการ
my-app/
├── src/
│   ├── index.tsx
│   ├── App.tsx
│   └── App.module.css
├── public/
│   └── index.html
└── webpack.config.js
```

### แบบฝึกหัดที่ 2: Vite Multi-Page App

สร้าง Vite project ที่มี:
- หน้า Home (/)
- หน้า About (/about)
- หน้า Admin (/admin)

แต่ละหน้ามี entry point แยกกัน

### แบบฝึกหัดที่ 3: Setup Complete Workflow

1. สร้าง Vite + React + TypeScript project
2. ตั้งค่า ESLint
3. ตั้งค่า Prettier
4. ตั้งค่า Husky
5. ตั้งค่า lint-staged
6. ตรวจสอบว่า pre-commit hook ทำงาน

### แบบฝึกหัดที่ 4: Webpack Bundle Analysis

1. เพิ่ม webpack-bundle-analyzer
2. Build project
3. วิเคราะห์ bundle size
4. ใช้ Code Splitting ลด bundle size

---

## สรุป Part 76

ในบทนี้เราได้เรียนรู้:

1. **Build Tools** แก้ปัญหา bundling, transpiling, optimization
2. **Webpack**: entry/output, loaders, plugins, code splitting
3. **Vite**: เร็วกว่า webpack ด้วย Native ESM, rollup สำหรับ production
4. **Babel**: แปลง modern JS ให้ browser เก่า รองรับ
5. **ESLint**: ตรวจสอบคุณภาพโค้ด
6. **Prettier**: จัดรูปแบบโค้ด
7. **Husky + lint-staged**: automation pre-commit checks
