# Part 97: Desktop Apps ด้วย Electron (Steps 1911-1930)

## บทนำ

Electron คือ framework ที่ช่วยให้เราสร้าง desktop application ด้วย web technologies (HTML, CSS, JavaScript) โดยใช้ Chromium เป็น renderer และ Node.js เป็น backend มีแอปพลิเคชันชื่อดังมากมายที่ใช้ Electron เช่น VS Code, Slack, Discord, GitHub Desktop และ Figma

---

## Step 1911: What is Electron?

### Electron Architecture

```
┌─────────────────────────────────────────┐
│              Electron App               │
├──────────────────┬──────────────────────┤
│   Main Process   │  Renderer Process(es)│
│   (Node.js)      │  (Chromium)          │
│                  │                      │
│  - BrowserWindow │  - HTML/CSS/JS       │
│  - App lifecycle │  - DOM               │
│  - File System   │  - Web APIs          │
│  - IPC           │  - IPC               │
│  - Native APIs   │                      │
└──────────────────┴──────────────────────┘
```

### ความแตกต่างระหว่าง Process

```javascript
// Main Process - Node.js environment
// มีสิทธิ์เข้าถึง OS APIs, File system, Native APIs

// Renderer Process - Browser environment
// แสดงผล UI, จำกัดด้วย sandbox
// ติดต่อ Main Process ผ่าน IPC
```

---

## Step 1912: Setup Electron Project

### สร้างโปรเจค Electron

```bash
# สร้าง project ใหม่
mkdir my-electron-app
cd my-electron-app
npm init -y

# ติดตั้ง Electron
npm install --save-dev electron

# หรือใช้ electron-forge (แนะนำ)
npm init electron-app@latest my-app -- --template=webpack
cd my-app
npm start
```

### โครงสร้างโปรเจค

```
my-electron-app/
├── package.json
├── main.js          # Main process
├── preload.js       # Preload script
├── renderer/
│   ├── index.html
│   ├── renderer.js
│   └── styles.css
└── assets/
    └── icon.png
```

### package.json

```json
{
  "name": "my-electron-app",
  "version": "1.0.0",
  "description": "My Electron Application",
  "main": "main.js",
  "scripts": {
    "start": "electron .",
    "dev": "electron . --watch",
    "build": "electron-builder",
    "build:win": "electron-builder --win",
    "build:mac": "electron-builder --mac",
    "build:linux": "electron-builder --linux"
  },
  "devDependencies": {
    "electron": "^28.0.0",
    "electron-builder": "^24.0.0"
  },
  "dependencies": {
    "electron-store": "^8.0.0",
    "electron-updater": "^6.0.0"
  },
  "build": {
    "appId": "com.mycompany.myapp",
    "productName": "My App",
    "copyright": "Copyright 2024",
    "directories": {
      "output": "dist"
    },
    "files": [
      "main.js",
      "preload.js",
      "renderer/**/*",
      "assets/**/*"
    ],
    "win": {
      "target": ["nsis", "zip"],
      "icon": "assets/icon.ico"
    },
    "mac": {
      "target": ["dmg", "zip"],
      "icon": "assets/icon.icns",
      "category": "public.app-category.productivity"
    },
    "linux": {
      "target": ["AppImage", "deb"],
      "icon": "assets/icon.png"
    }
  }
}
```

---

## Step 1913: Main Process

### main.js - หัวใจของ Electron App

```javascript
// main.js
const { app, BrowserWindow, ipcMain, Menu, dialog, shell, Tray, nativeImage } = require('electron');
const path = require('path');
const fs = require('fs');

// ป้องกัน GC ของ window
let mainWindow = null;
let tray = null;

// ตรวจสอบว่า dev mode หรือไม่
const isDev = process.env.NODE_ENV === 'development' || !app.isPackaged;

function createWindow() {
  mainWindow = new BrowserWindow({
    width: 1200,
    height: 800,
    minWidth: 800,
    minHeight: 600,
    title: 'My Electron App',
    icon: path.join(__dirname, 'assets/icon.png'),
    
    // Window frame options
    frame: true,           // false สำหรับ frameless window
    titleBarStyle: 'default', // 'hidden', 'hiddenInset' (macOS)
    
    // Background
    backgroundColor: '#1e1e1e',
    
    // Show ช้าเพื่อป้องกัน flash
    show: false,
    
    // Center on screen
    center: true,
    
    // Transparent (ต้อง frame: false)
    transparent: false,
    
    webPreferences: {
      // Security settings
      nodeIntegration: false,        // ห้าม node ใน renderer
      contextIsolation: true,        // แยก context
      sandbox: true,                 // sandbox renderer
      webSecurity: true,             // ป้องกัน cross-origin
      
      // Preload script
      preload: path.join(__dirname, 'preload.js'),
      
      // Dev tools
      devTools: isDev
    }
  });

  // โหลด HTML
  if (isDev) {
    mainWindow.loadURL('http://localhost:3000');
    mainWindow.webContents.openDevTools();
  } else {
    mainWindow.loadFile(path.join(__dirname, 'renderer/index.html'));
  }

  // แสดง window หลังจากโหลดเสร็จ
  mainWindow.once('ready-to-show', () => {
    mainWindow.show();
    
    if (isDev) {
      mainWindow.webContents.openDevTools({ mode: 'detach' });
    }
  });

  // จัดการ close event
  mainWindow.on('closed', () => {
    mainWindow = null;
  });

  // ป้องกัน navigation ออกไปภายนอก
  mainWindow.webContents.on('will-navigate', (event, url) => {
    if (!url.startsWith('file://') && !url.startsWith('http://localhost')) {
      event.preventDefault();
      shell.openExternal(url);
    }
  });

  // จัดการ new window
  mainWindow.webContents.setWindowOpenHandler(({ url }) => {
    shell.openExternal(url);
    return { action: 'deny' };
  });
}

// App lifecycle
app.whenReady().then(() => {
  createWindow();
  createTray();
  createMenu();

  app.on('activate', () => {
    if (BrowserWindow.getAllWindows().length === 0) {
      createWindow();
    }
  });
});

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit();
  }
});

// ป้องกัน multiple instances
const gotTheLock = app.requestSingleInstanceLock();
if (!gotTheLock) {
  app.quit();
} else {
  app.on('second-instance', () => {
    if (mainWindow) {
      if (mainWindow.isMinimized()) mainWindow.restore();
      mainWindow.focus();
    }
  });
}

// Security: ป้องกัน remote content
app.on('web-contents-created', (event, contents) => {
  contents.on('will-navigate', (event, url) => {
    const allowedUrls = ['http://localhost:3000', 'file://'];
    if (!allowedUrls.some(allowed => url.startsWith(allowed))) {
      event.preventDefault();
    }
  });

  contents.setWindowOpenHandler(({ url }) => {
    if (url.startsWith('https://')) {
      shell.openExternal(url);
    }
    return { action: 'deny' };
  });
});
```

---

## Step 1914: BrowserWindow Configuration

### การตั้งค่า BrowserWindow ขั้นสูง

```javascript
// BrowserWindow options ขั้นสูง
function createAdvancedWindow() {
  const window = new BrowserWindow({
    // ขนาดและตำแหน่ง
    width: 1200,
    height: 800,
    x: 100,
    y: 100,
    minWidth: 400,
    minHeight: 300,
    maxWidth: 1920,
    maxHeight: 1080,

    // การแสดงผล
    fullscreen: false,
    fullscreenable: true,
    resizable: true,
    movable: true,
    minimizable: true,
    maximizable: true,
    closable: true,

    // Visual
    opacity: 1.0,
    hasShadow: true,
    roundedCorners: true, // macOS

    // Title bar (macOS)
    titleBarStyle: 'hiddenInset', // ซ่อน title bar แต่ยังมี traffic lights
    trafficLightPosition: { x: 12, y: 16 },

    // Vibrancy effect (macOS)
    vibrancy: 'under-window',
    visualEffectState: 'active',

    // Windows Acrylic effect
    backgroundMaterial: 'acrylic', // Windows 11

    webPreferences: {
      preload: path.join(__dirname, 'preload.js'),
      nodeIntegration: false,
      contextIsolation: true,
      sandbox: true,
      additionalArguments: ['--my-arg=value'],
      backgroundThrottling: false,
      spellcheck: true,
      zoomFactor: 1.0
    }
  });

  return window;
}

// Child window
function createChildWindow(parent) {
  const child = new BrowserWindow({
    parent,
    modal: true,
    width: 400,
    height: 300,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js'),
      contextIsolation: true
    }
  });

  child.loadFile('renderer/dialog.html');
  return child;
}

// Splash screen
function createSplashScreen() {
  const splash = new BrowserWindow({
    width: 400,
    height: 300,
    frame: false,
    transparent: true,
    alwaysOnTop: true,
    skipTaskbar: true
  });

  splash.loadFile('renderer/splash.html');

  // ปิด splash screen หลัง 3 วินาที
  setTimeout(() => {
    splash.close();
    createWindow();
  }, 3000);
}

// Window state persistence
const { screen } = require('electron');
const Store = require('electron-store');

const store = new Store();

function getWindowState() {
  const defaultState = {
    width: 1200,
    height: 800,
    x: undefined,
    y: undefined,
    isMaximized: false
  };

  const savedState = store.get('windowState', defaultState);

  // ตรวจสอบว่า window อยู่ในหน้าจอ
  if (savedState.x !== undefined) {
    const displays = screen.getAllDisplays();
    const isVisible = displays.some(display => {
      const { bounds } = display;
      return (
        savedState.x >= bounds.x &&
        savedState.y >= bounds.y &&
        savedState.x + savedState.width <= bounds.x + bounds.width &&
        savedState.y + savedState.height <= bounds.y + bounds.height
      );
    });

    if (!isVisible) {
      return defaultState;
    }
  }

  return savedState;
}

function saveWindowState(window) {
  const bounds = window.getBounds();
  store.set('windowState', {
    ...bounds,
    isMaximized: window.isMaximized()
  });
}
```

---

## Step 1915: IPC Communication

### Inter-Process Communication

```javascript
// preload.js - สะพานระหว่าง Main และ Renderer
const { contextBridge, ipcRenderer } = require('electron');

// Expose ฟังก์ชันที่ปลอดภัยให้ renderer
contextBridge.exposeInMainWorld('electronAPI', {
  // File operations
  readFile: (filePath) => ipcRenderer.invoke('file:read', filePath),
  writeFile: (filePath, content) => ipcRenderer.invoke('file:write', filePath, content),
  openFileDialog: (options) => ipcRenderer.invoke('dialog:openFile', options),
  saveFileDialog: (options) => ipcRenderer.invoke('dialog:saveFile', options),

  // App info
  getAppVersion: () => ipcRenderer.invoke('app:version'),
  getPlatform: () => process.platform,
  
  // Theme
  getTheme: () => ipcRenderer.invoke('theme:get'),
  setTheme: (theme) => ipcRenderer.invoke('theme:set', theme),
  
  // Window
  minimize: () => ipcRenderer.send('window:minimize'),
  maximize: () => ipcRenderer.send('window:maximize'),
  close: () => ipcRenderer.send('window:close'),
  
  // Events (renderer listening to main)
  onUpdateAvailable: (callback) => {
    ipcRenderer.on('update:available', (_, data) => callback(data));
    return () => ipcRenderer.removeAllListeners('update:available');
  },
  
  onFileChanged: (callback) => {
    const handler = (_, data) => callback(data);
    ipcRenderer.on('file:changed', handler);
    return () => ipcRenderer.removeListener('file:changed', handler);
  },

  // Store
  storeGet: (key) => ipcRenderer.invoke('store:get', key),
  storeSet: (key, value) => ipcRenderer.invoke('store:set', key, value),
  storeDelete: (key) => ipcRenderer.invoke('store:delete', key),

  // Shell
  openExternal: (url) => ipcRenderer.invoke('shell:openExternal', url),
  showItemInFolder: (path) => ipcRenderer.invoke('shell:showItemInFolder', path),
  
  // Notifications
  showNotification: (options) => ipcRenderer.invoke('notification:show', options),
  
  // Clipboard
  writeToClipboard: (text) => ipcRenderer.invoke('clipboard:write', text),
  readFromClipboard: () => ipcRenderer.invoke('clipboard:read')
});

// main.js - IPC Handlers
const { ipcMain, app, dialog, shell, clipboard, nativeTheme } = require('electron');
const Store = require('electron-store');
const store = new Store();

// File operations
ipcMain.handle('file:read', async (event, filePath) => {
  try {
    const content = fs.readFileSync(filePath, 'utf-8');
    return { success: true, content };
  } catch (error) {
    return { success: false, error: error.message };
  }
});

ipcMain.handle('file:write', async (event, filePath, content) => {
  try {
    fs.writeFileSync(filePath, content, 'utf-8');
    return { success: true };
  } catch (error) {
    return { success: false, error: error.message };
  }
});

// Dialog
ipcMain.handle('dialog:openFile', async (event, options = {}) => {
  const result = await dialog.showOpenDialog(mainWindow, {
    title: 'เปิดไฟล์',
    properties: ['openFile', 'multiSelections'],
    filters: [
      { name: 'Text Files', extensions: ['txt', 'md'] },
      { name: 'All Files', extensions: ['*'] }
    ],
    ...options
  });

  return result;
});

ipcMain.handle('dialog:saveFile', async (event, options = {}) => {
  const result = await dialog.showSaveDialog(mainWindow, {
    title: 'บันทึกไฟล์',
    defaultPath: 'document.txt',
    filters: [
      { name: 'Text Files', extensions: ['txt'] },
      { name: 'All Files', extensions: ['*'] }
    ],
    ...options
  });

  return result;
});

// App info
ipcMain.handle('app:version', () => app.getVersion());

// Theme
ipcMain.handle('theme:get', () => nativeTheme.themeSource);
ipcMain.handle('theme:set', (event, theme) => {
  nativeTheme.themeSource = theme; // 'dark', 'light', 'system'
});

// Window controls
ipcMain.on('window:minimize', () => mainWindow?.minimize());
ipcMain.on('window:maximize', () => {
  if (mainWindow?.isMaximized()) {
    mainWindow.unmaximize();
  } else {
    mainWindow?.maximize();
  }
});
ipcMain.on('window:close', () => mainWindow?.close());

// Store
ipcMain.handle('store:get', (event, key) => store.get(key));
ipcMain.handle('store:set', (event, key, value) => store.set(key, value));
ipcMain.handle('store:delete', (event, key) => store.delete(key));

// Shell
ipcMain.handle('shell:openExternal', async (event, url) => {
  await shell.openExternal(url);
});

ipcMain.handle('shell:showItemInFolder', async (event, path) => {
  shell.showItemInFolder(path);
});

// Clipboard
ipcMain.handle('clipboard:write', (event, text) => {
  clipboard.writeText(text);
});

ipcMain.handle('clipboard:read', () => {
  return clipboard.readText();
});
```

---

## Step 1916: Application Menu

### สร้าง Menu

```javascript
// main.js - สร้าง Application Menu
function createMenu() {
  const template = [
    {
      label: 'ไฟล์',
      submenu: [
        {
          label: 'ใหม่',
          accelerator: 'CmdOrCtrl+N',
          click: () => {
            mainWindow.webContents.send('menu:newFile');
          }
        },
        {
          label: 'เปิด...',
          accelerator: 'CmdOrCtrl+O',
          click: async () => {
            const result = await dialog.showOpenDialog(mainWindow, {
              properties: ['openFile'],
              filters: [{ name: 'Text', extensions: ['txt', 'md', 'js'] }]
            });

            if (!result.canceled) {
              const content = fs.readFileSync(result.filePaths[0], 'utf-8');
              mainWindow.webContents.send('menu:openFile', {
                path: result.filePaths[0],
                content
              });
            }
          }
        },
        {
          label: 'บันทึก',
          accelerator: 'CmdOrCtrl+S',
          click: () => {
            mainWindow.webContents.send('menu:save');
          }
        },
        { type: 'separator' },
        {
          label: 'ตั้งค่า',
          accelerator: 'CmdOrCtrl+,',
          click: () => {
            createSettingsWindow();
          }
        },
        { type: 'separator' },
        {
          label: 'ออก',
          accelerator: 'CmdOrCtrl+Q',
          click: () => app.quit()
        }
      ]
    },
    {
      label: 'แก้ไข',
      submenu: [
        { label: 'ยกเลิก', role: 'undo' },
        { label: 'ทำซ้ำ', role: 'redo' },
        { type: 'separator' },
        { label: 'ตัด', role: 'cut' },
        { label: 'คัดลอก', role: 'copy' },
        { label: 'วาง', role: 'paste' },
        { label: 'เลือกทั้งหมด', role: 'selectAll' }
      ]
    },
    {
      label: 'มุมมอง',
      submenu: [
        { label: 'โหลดซ้ำ', role: 'reload' },
        { label: 'โหลดซ้ำ (ไม่ใช้ cache)', role: 'forceReload' },
        { label: 'Developer Tools', role: 'toggleDevTools' },
        { type: 'separator' },
        { label: 'เพิ่มขนาด', role: 'zoomIn' },
        { label: 'ลดขนาด', role: 'zoomOut' },
        { label: 'ขนาดปกติ', role: 'resetZoom' },
        { type: 'separator' },
        { label: 'เต็มจอ', role: 'togglefullscreen' }
      ]
    },
    {
      label: 'ช่วยเหลือ',
      submenu: [
        {
          label: 'เกี่ยวกับ',
          click: () => {
            dialog.showMessageBox(mainWindow, {
              type: 'info',
              title: 'เกี่ยวกับ',
              message: 'My Electron App',
              detail: `Version: ${app.getVersion()}\nElectron: ${process.versions.electron}\nNode.js: ${process.versions.node}`,
              buttons: ['OK']
            });
          }
        },
        {
          label: 'เอกสาร',
          click: () => {
            shell.openExternal('https://example.com/docs');
          }
        }
      ]
    }
  ];

  // macOS ต้องการ menu แรกเป็น app name
  if (process.platform === 'darwin') {
    template.unshift({
      label: app.getName(),
      submenu: [
        { label: 'เกี่ยวกับ', role: 'about' },
        { type: 'separator' },
        { label: 'ซ่อน', role: 'hide' },
        { label: 'ซ่อนโปรแกรมอื่น', role: 'hideOthers' },
        { label: 'แสดงทั้งหมด', role: 'unhide' },
        { type: 'separator' },
        { label: 'ออก', role: 'quit' }
      ]
    });
  }

  const menu = Menu.buildFromTemplate(template);
  Menu.setApplicationMenu(menu);
}

// Context Menu
function createContextMenu(x, y) {
  const contextMenu = Menu.buildFromTemplate([
    {
      label: 'ตัด',
      role: 'cut'
    },
    {
      label: 'คัดลอก',
      role: 'copy'
    },
    {
      label: 'วาง',
      role: 'paste'
    },
    { type: 'separator' },
    {
      label: 'เปิดใน Browser',
      click: () => {
        shell.openExternal('https://example.com');
      }
    },
    {
      label: 'Inspect Element',
      click: () => {
        mainWindow.webContents.inspectElement(x, y);
      }
    }
  ]);

  contextMenu.popup({ window: mainWindow, x, y });
}

// Tray icon
function createTray() {
  const icon = nativeImage.createFromPath(path.join(__dirname, 'assets/tray-icon.png'));
  tray = new Tray(icon);

  const trayMenu = Menu.buildFromTemplate([
    {
      label: 'เปิดแอป',
      click: () => {
        if (mainWindow) {
          mainWindow.show();
          mainWindow.focus();
        } else {
          createWindow();
        }
      }
    },
    { type: 'separator' },
    {
      label: 'ออก',
      click: () => app.quit()
    }
  ]);

  tray.setContextMenu(trayMenu);
  tray.setToolTip('My Electron App');

  tray.on('click', () => {
    if (mainWindow) {
      if (mainWindow.isVisible()) {
        mainWindow.hide();
      } else {
        mainWindow.show();
      }
    }
  });
}
```

---

## Step 1917: Native Dialogs

### Dialog APIs

```javascript
// main.js - Various Dialogs

// Open File Dialog
ipcMain.handle('dialog:openFile', async (event, options) => {
  return await dialog.showOpenDialog(mainWindow, {
    title: 'เปิดไฟล์',
    defaultPath: app.getPath('documents'),
    properties: ['openFile', 'multiSelections'],
    filters: options?.filters || [
      { name: 'All Files', extensions: ['*'] }
    ]
  });
});

// Save File Dialog
ipcMain.handle('dialog:saveFile', async (event, options) => {
  return await dialog.showSaveDialog(mainWindow, {
    title: 'บันทึกไฟล์',
    defaultPath: path.join(app.getPath('documents'), 'untitled.txt'),
    filters: options?.filters || [
      { name: 'Text', extensions: ['txt'] }
    ],
    properties: ['createDirectory']
  });
});

// Open Directory Dialog
ipcMain.handle('dialog:openDirectory', async () => {
  return await dialog.showOpenDialog(mainWindow, {
    title: 'เลือกโฟลเดอร์',
    properties: ['openDirectory', 'createDirectory']
  });
});

// Message Box
ipcMain.handle('dialog:message', async (event, options) => {
  return await dialog.showMessageBox(mainWindow, {
    type: options.type || 'info', // 'none', 'info', 'error', 'question', 'warning'
    title: options.title || 'แจ้งเตือน',
    message: options.message,
    detail: options.detail,
    buttons: options.buttons || ['OK'],
    defaultId: 0,
    cancelId: 1,
    checkboxLabel: options.checkboxLabel,
    checkboxChecked: false
  });
});

// Error Dialog
ipcMain.handle('dialog:error', async (event, title, content) => {
  dialog.showErrorBox(title, content);
});

// ตัวอย่างการใช้ใน renderer
// renderer.js
async function openFile() {
  const result = await window.electronAPI.openFileDialog({
    filters: [
      { name: 'JavaScript', extensions: ['js', 'ts'] },
      { name: 'All Files', extensions: ['*'] }
    ]
  });

  if (!result.canceled) {
    const filePaths = result.filePaths;
    // โหลด file
    for (const filePath of filePaths) {
      const { success, content } = await window.electronAPI.readFile(filePath);
      if (success) {
        console.log('File content:', content);
      }
    }
  }
}

async function confirmDelete(itemName) {
  const result = await window.electronAPI.showMessageDialog({
    type: 'question',
    title: 'ยืนยันการลบ',
    message: `ต้องการลบ "${itemName}" ใช่หรือไม่?`,
    detail: 'การกระทำนี้ไม่สามารถยกเลิกได้',
    buttons: ['ลบ', 'ยกเลิก'],
    defaultId: 1,
    cancelId: 1
  });

  return result.response === 0; // 0 = 'ลบ'
}
```

---

## Step 1918: File System Operations

### การจัดการ File System

```javascript
// main.js - File System Handler
const fs = require('fs').promises;
const path = require('path');
const chokidar = require('chokidar'); // npm install chokidar

// อ่านไฟล์
ipcMain.handle('fs:readFile', async (event, filePath) => {
  try {
    const content = await fs.readFile(filePath, 'utf-8');
    const stats = await fs.stat(filePath);
    return {
      success: true,
      content,
      stats: {
        size: stats.size,
        modified: stats.mtime,
        created: stats.birthtime
      }
    };
  } catch (error) {
    return { success: false, error: error.message };
  }
});

// เขียนไฟล์
ipcMain.handle('fs:writeFile', async (event, filePath, content) => {
  try {
    // สร้าง directory ถ้าไม่มี
    await fs.mkdir(path.dirname(filePath), { recursive: true });
    await fs.writeFile(filePath, content, 'utf-8');
    return { success: true };
  } catch (error) {
    return { success: false, error: error.message };
  }
});

// อ่าน directory
ipcMain.handle('fs:readDir', async (event, dirPath) => {
  try {
    const entries = await fs.readdir(dirPath, { withFileTypes: true });
    const items = await Promise.all(
      entries.map(async (entry) => {
        const fullPath = path.join(dirPath, entry.name);
        const stats = await fs.stat(fullPath);
        return {
          name: entry.name,
          path: fullPath,
          isDirectory: entry.isDirectory(),
          isFile: entry.isFile(),
          size: stats.size,
          modified: stats.mtime
        };
      })
    );
    return { success: true, items };
  } catch (error) {
    return { success: false, error: error.message };
  }
});

// ลบไฟล์/โฟลเดอร์
ipcMain.handle('fs:delete', async (event, filePath) => {
  try {
    const stats = await fs.stat(filePath);
    if (stats.isDirectory()) {
      await fs.rm(filePath, { recursive: true, force: true });
    } else {
      await fs.unlink(filePath);
    }
    return { success: true };
  } catch (error) {
    return { success: false, error: error.message };
  }
});

// คัดลอกไฟล์
ipcMain.handle('fs:copy', async (event, src, dest) => {
  try {
    await fs.cp(src, dest, { recursive: true });
    return { success: true };
  } catch (error) {
    return { success: false, error: error.message };
  }
});

// เฝ้าดูการเปลี่ยนแปลงของไฟล์
let watcher = null;
ipcMain.handle('fs:watch', async (event, dirPath) => {
  if (watcher) {
    watcher.close();
  }

  watcher = chokidar.watch(dirPath, {
    ignored: /(^|[\/\\])\../, // ละเว้น hidden files
    persistent: true
  });

  watcher
    .on('add', (path) => {
      mainWindow.webContents.send('fs:change', { type: 'add', path });
    })
    .on('change', (path) => {
      mainWindow.webContents.send('fs:change', { type: 'change', path });
    })
    .on('unlink', (path) => {
      mainWindow.webContents.send('fs:change', { type: 'delete', path });
    })
    .on('addDir', (path) => {
      mainWindow.webContents.send('fs:change', { type: 'addDir', path });
    })
    .on('unlinkDir', (path) => {
      mainWindow.webContents.send('fs:change', { type: 'deleteDir', path });
    });
});

// App paths
ipcMain.handle('app:paths', () => ({
  home: app.getPath('home'),
  appData: app.getPath('appData'),
  userData: app.getPath('userData'),
  temp: app.getPath('temp'),
  desktop: app.getPath('desktop'),
  documents: app.getPath('documents'),
  downloads: app.getPath('downloads'),
  pictures: app.getPath('pictures'),
  music: app.getPath('music'),
  exe: app.getPath('exe')
}));
```

---

## Step 1919: System Notifications

### การแสดง Notification

```javascript
// main.js
const { Notification } = require('electron');

ipcMain.handle('notification:show', async (event, options) => {
  if (!Notification.isSupported()) {
    return { success: false, error: 'Notifications not supported' };
  }

  const notification = new Notification({
    title: options.title,
    body: options.body,
    icon: options.icon || path.join(__dirname, 'assets/icon.png'),
    silent: options.silent || false,
    urgency: options.urgency || 'normal', // 'low', 'normal', 'critical' (Linux)
    timeoutType: 'default', // 'default', 'never'
    actions: options.actions || [] // Windows 10+ Action Center
  });

  notification.on('click', () => {
    mainWindow.focus();
    mainWindow.webContents.send('notification:clicked', options.id);
  });

  notification.on('close', () => {
    mainWindow.webContents.send('notification:closed', options.id);
  });

  notification.show();
  return { success: true };
});

// renderer.js
async function showNotification(title, body) {
  // วิธีที่ 1: ผ่าน IPC
  await window.electronAPI.showNotification({
    id: Date.now(),
    title,
    body,
    icon: 'assets/icon.png'
  });
}

// วิธีที่ 2: Web Notifications API (ต้องขอ permission)
async function showWebNotification(title, body) {
  if (!('Notification' in window)) {
    console.log('Notifications not supported');
    return;
  }

  const permission = await Notification.requestPermission();
  if (permission === 'granted') {
    new Notification(title, {
      body,
      icon: 'assets/icon.png'
    });
  }
}
```

---

## Step 1920: Auto Updater

### Electron Auto-Updater

```javascript
// main.js
const { autoUpdater } = require('electron-updater');
const log = require('electron-log');

autoUpdater.logger = log;
autoUpdater.logger.transports.file.level = 'info';

function setupAutoUpdater() {
  // ตั้งค่า update server
  autoUpdater.setFeedURL({
    provider: 'github',
    owner: 'your-github-username',
    repo: 'your-app-name',
    private: false
  });

  // Events
  autoUpdater.on('checking-for-update', () => {
    log.info('Checking for update...');
    mainWindow.webContents.send('update:checking');
  });

  autoUpdater.on('update-available', (info) => {
    log.info('Update available:', info);
    mainWindow.webContents.send('update:available', {
      version: info.version,
      releaseDate: info.releaseDate,
      releaseNotes: info.releaseNotes
    });
  });

  autoUpdater.on('update-not-available', () => {
    log.info('Update not available');
    mainWindow.webContents.send('update:not-available');
  });

  autoUpdater.on('error', (error) => {
    log.error('Update error:', error);
    mainWindow.webContents.send('update:error', error.message);
  });

  autoUpdater.on('download-progress', (progress) => {
    mainWindow.webContents.send('update:progress', {
      percent: progress.percent,
      transferred: progress.transferred,
      total: progress.total,
      bytesPerSecond: progress.bytesPerSecond
    });
  });

  autoUpdater.on('update-downloaded', (info) => {
    log.info('Update downloaded:', info);
    mainWindow.webContents.send('update:downloaded', {
      version: info.version
    });
  });

  // ตรวจสอบ update เมื่อ app เปิด (หลัง 3 วินาที)
  app.whenReady().then(() => {
    setTimeout(() => {
      autoUpdater.checkForUpdates();
    }, 3000);
  });

  // ตรวจสอบทุก 4 ชั่วโมง
  setInterval(() => {
    autoUpdater.checkForUpdates();
  }, 4 * 60 * 60 * 1000);
}

// IPC handlers สำหรับ updater
ipcMain.handle('update:check', () => autoUpdater.checkForUpdates());
ipcMain.handle('update:install', () => autoUpdater.quitAndInstall());

// renderer.js - Update UI
window.electronAPI.onUpdateAvailable((data) => {
  const updateBanner = document.getElementById('update-banner');
  updateBanner.innerHTML = `
    <div class="update-notification">
      <p>พบ Version ใหม่: ${data.version}</p>
      <button onclick="downloadUpdate()">ดาวน์โหลดอัปเดต</button>
      <button onclick="dismissUpdate()">ข้ามไปก่อน</button>
    </div>
  `;
  updateBanner.style.display = 'block';
});

window.electronAPI.onUpdateProgress((data) => {
  const progressBar = document.getElementById('update-progress');
  progressBar.value = data.percent;
  progressBar.textContent = `${Math.round(data.percent)}%`;
});

window.electronAPI.onUpdateDownloaded((data) => {
  if (confirm(`อัปเดต ${data.version} พร้อมแล้ว ต้องการรีสตาร์ทตอนนี้หรือไม่?`)) {
    window.electronAPI.installUpdate();
  }
});
```

---

## Step 1921: Electron กับ React

### Electron + React + TypeScript

```bash
# สร้างโปรเจคด้วย electron-vite
npm create @quick-start/electron my-app -- --template react-ts
cd my-app
npm install
npm run dev
```

```typescript
// src/main/index.ts
import { app, BrowserWindow, ipcMain } from 'electron';
import { join } from 'path';
import { is } from '@electron-toolkit/utils';

function createWindow(): void {
  const mainWindow = new BrowserWindow({
    width: 1200,
    height: 800,
    webPreferences: {
      preload: join(__dirname, '../preload/index.js'),
      sandbox: false
    }
  });

  if (is.dev && process.env['ELECTRON_RENDERER_URL']) {
    mainWindow.loadURL(process.env['ELECTRON_RENDERER_URL']);
  } else {
    mainWindow.loadFile(join(__dirname, '../renderer/index.html'));
  }
}

app.whenReady().then(() => {
  createWindow();
});

// src/preload/index.ts
import { contextBridge, ipcRenderer } from 'electron';
import { electronAPI } from '@electron-toolkit/preload';

contextBridge.exposeInMainWorld('electron', electronAPI);
contextBridge.exposeInMainWorld('api', {
  getData: () => ipcRenderer.invoke('get-data'),
  saveData: (data: unknown) => ipcRenderer.invoke('save-data', data)
});

// src/renderer/src/App.tsx
import React, { useState, useEffect } from 'react';

declare global {
  interface Window {
    api: {
      getData: () => Promise<unknown>;
      saveData: (data: unknown) => Promise<void>;
    };
  }
}

function App(): JSX.Element {
  const [data, setData] = useState(null);

  useEffect(() => {
    window.api.getData().then(setData);
  }, []);

  return (
    <div className="app">
      <h1>My Electron + React App</h1>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
}

export default App;
```

---

## Step 1922: Security Best Practices

### Security ใน Electron

```javascript
// main.js - Security checklist

// 1. ปิด nodeIntegration
const secureWindow = new BrowserWindow({
  webPreferences: {
    nodeIntegration: false,       // ❌ อย่าเปิด
    contextIsolation: true,       // ✅ เปิดเสมอ
    sandbox: true,                // ✅ เปิดเสมอ
    webSecurity: true,            // ✅ เปิดเสมอ
    allowRunningInsecureContent: false, // ❌ อย่าเปิด
    experimentalFeatures: false,  // ❌ อย่าเปิดนอกจากจำเป็น
    enableRemoteModule: false,    // ❌ deprecated แล้ว อย่าใช้
  }
});

// 2. Validate IPC messages
ipcMain.handle('safe-operation', async (event, data) => {
  // ตรวจสอบ sender
  const webContents = event.sender;
  const window = BrowserWindow.fromWebContents(webContents);
  
  if (!window) {
    throw new Error('Invalid sender');
  }
  
  // ตรวจสอบ URL ของ sender
  const url = webContents.getURL();
  if (!url.startsWith('file://') && !url.startsWith('http://localhost')) {
    throw new Error('Unauthorized sender');
  }
  
  // Validate data
  if (!isValidData(data)) {
    throw new Error('Invalid data');
  }
  
  return processData(data);
});

function isValidData(data) {
  if (typeof data !== 'object' || data === null) return false;
  if (typeof data.id !== 'string') return false;
  if (!['create', 'update', 'delete'].includes(data.action)) return false;
  return true;
}

// 3. CSP Headers
mainWindow.webContents.session.webRequest.onHeadersReceived((details, callback) => {
  callback({
    responseHeaders: {
      ...details.responseHeaders,
      'Content-Security-Policy': [
        "default-src 'self'",
        "script-src 'self'",
        "style-src 'self' 'unsafe-inline'",
        "img-src 'self' data: https:",
        "font-src 'self'",
        "connect-src 'self' https://api.example.com"
      ].join('; ')
    }
  });
});

// 4. ป้องกัน protocol hijacking
app.on('web-contents-created', (event, contents) => {
  contents.on('will-navigate', (event, url) => {
    if (!isAllowedUrl(url)) {
      event.preventDefault();
    }
  });
  
  contents.setWindowOpenHandler(({ url }) => {
    if (url.startsWith('https://')) {
      shell.openExternal(url);
    }
    return { action: 'deny' };
  });
});

function isAllowedUrl(url) {
  const allowedOrigins = [
    'file://',
    'http://localhost:3000',
    'https://myapi.example.com'
  ];
  return allowedOrigins.some(origin => url.startsWith(origin));
}

// 5. ปิด webview
app.on('web-contents-created', (event, contents) => {
  if (contents.getType() === 'webview') {
    contents.on('will-attach-webview', (event) => {
      event.preventDefault(); // ไม่อนุญาต webview
    });
  }
});
```

---

## Step 1923: Building และ Packaging

### electron-builder Configuration

```javascript
// electron-builder.config.js
module.exports = {
  appId: 'com.company.app',
  productName: 'My App',
  copyright: 'Copyright 2024 Company',
  
  // ไฟล์ที่จะ pack
  files: [
    'dist/**/*',
    'main.js',
    'preload.js',
    'package.json',
    '!node_modules/**/*'
  ],
  
  asar: true, // pack ไฟล์ใน .asar archive
  asarUnpack: [
    'node_modules/some-native-module/**/*' // native modules ต้อง unpack
  ],
  
  // Windows
  win: {
    target: [
      { target: 'nsis', arch: ['x64', 'ia32', 'arm64'] },
      { target: 'portable', arch: ['x64'] },
      { target: 'zip', arch: ['x64'] }
    ],
    icon: 'assets/icon.ico',
    requestedExecutionLevel: 'asInvoker',
    publisherName: 'Company Name',
    verifyUpdateCodeSignature: true
  },
  
  nsis: {
    oneClick: false,
    allowToChangeInstallationDirectory: true,
    createDesktopShortcut: true,
    createStartMenuShortcut: true,
    shortcutName: 'My App',
    installerIcon: 'assets/installer.ico',
    uninstallerIcon: 'assets/uninstaller.ico',
    installerHeaderIcon: 'assets/icon.ico',
    deleteAppDataOnUninstall: false,
    runAfterFinish: true
  },
  
  // macOS
  mac: {
    target: [
      { target: 'dmg', arch: ['x64', 'arm64', 'universal'] },
      { target: 'zip', arch: ['x64', 'arm64', 'universal'] }
    ],
    icon: 'assets/icon.icns',
    category: 'public.app-category.productivity',
    hardenedRuntime: true,
    gatekeeperAssess: false,
    entitlements: 'entitlements.mac.plist',
    entitlementsInherit: 'entitlements.mac.plist',
    notarize: {
      teamId: 'YOUR_TEAM_ID'
    }
  },
  
  dmg: {
    contents: [
      { x: 410, y: 150, type: 'link', path: '/Applications' },
      { x: 130, y: 150, type: 'file' }
    ]
  },
  
  // Linux
  linux: {
    target: [
      { target: 'AppImage', arch: ['x64'] },
      { target: 'deb', arch: ['x64'] },
      { target: 'rpm', arch: ['x64'] },
      { target: 'snap', arch: ['x64'] }
    ],
    icon: 'assets/icon.png',
    category: 'Utility',
    desktop: {
      Name: 'My App',
      Comment: 'My Electron Application'
    }
  },
  
  // Auto-update
  publish: [
    {
      provider: 'github',
      owner: 'github-username',
      repo: 'app-repo',
      releaseType: 'release'
    },
    {
      provider: 's3',
      bucket: 'my-bucket',
      region: 'us-east-1'
    }
  ]
};
```

```bash
# Build commands
npm run build           # build สำหรับ platform ปัจจุบัน
npm run build:win       # build สำหรับ Windows
npm run build:mac       # build สำหรับ macOS
npm run build:linux     # build สำหรับ Linux

# Build ด้วย electron-builder โดยตรง
npx electron-builder --win --x64
npx electron-builder --mac --universal
npx electron-builder --linux --x64

# Publish
npx electron-builder --publish always
npx electron-builder --publish onTagOrDraft
```

---

## Step 1924-1930: Native Features

### Native OS Integration

```javascript
// main.js - Native Features

// 1. Badge (macOS)
ipcMain.on('badge:set', (event, count) => {
  app.setBadgeCount(count);
  app.dock?.setBadge(count > 0 ? count.toString() : '');
});

// 2. Recent Files (Windows/macOS)
ipcMain.on('recent:add', (event, { name, path }) => {
  app.addRecentDocument(path);
});

ipcMain.on('recent:clear', () => {
  app.clearRecentDocuments();
});

// macOS Dock menu
if (process.platform === 'darwin') {
  const dockMenu = Menu.buildFromTemplate([
    {
      label: 'ไฟล์ล่าสุด',
      submenu: [] // เพิ่ม dynamic menu items
    },
    {
      label: 'สร้างใหม่',
      click: () => createWindow()
    }
  ]);
  app.dock.setMenu(dockMenu);
}

// 3. Jump List (Windows)
if (process.platform === 'win32') {
  app.setJumpList([
    {
      type: 'custom',
      name: 'ไฟล์ล่าสุด',
      items: [
        {
          type: 'file',
          path: 'C:\\path\\to\\file.txt'
        }
      ]
    },
    {
      type: 'frequent'
    },
    {
      type: 'recent'
    },
    {
      type: 'tasks',
      items: [
        {
          type: 'task',
          title: 'สร้างไฟล์ใหม่',
          description: 'สร้างไฟล์เปล่า',
          program: process.execPath,
          args: '--new-file',
          iconPath: process.execPath,
          iconIndex: 0
        }
      ]
    }
  ]);
}

// 4. Power Monitor
const { powerMonitor, powerSaveBlocker } = require('electron');

powerMonitor.on('suspend', () => {
  console.log('System suspending...');
  // บันทึก state ก่อน suspend
});

powerMonitor.on('resume', () => {
  console.log('System resumed');
  // sync data
});

// ป้องกัน screen sleep (เช่น ระหว่าง download)
let blockId;
ipcMain.handle('power:preventSleep', () => {
  blockId = powerSaveBlocker.start('prevent-display-sleep');
});

ipcMain.handle('power:allowSleep', () => {
  if (blockId !== undefined) {
    powerSaveBlocker.stop(blockId);
  }
});

// 5. Screen info
const { screen } = require('electron');

ipcMain.handle('screen:getAll', () => {
  return screen.getAllDisplays().map(display => ({
    id: display.id,
    bounds: display.bounds,
    workArea: display.workArea,
    scaleFactor: display.scaleFactor,
    primary: display.id === screen.getPrimaryDisplay().id
  }));
});

// 6. Keyboard shortcuts (Global)
const { globalShortcut } = require('electron');

app.whenReady().then(() => {
  globalShortcut.register('CommandOrControl+Shift+X', () => {
    if (mainWindow.isVisible()) {
      mainWindow.hide();
    } else {
      mainWindow.show();
      mainWindow.focus();
    }
  });

  globalShortcut.register('CommandOrControl+Shift+D', () => {
    mainWindow.webContents.openDevTools();
  });
});

app.on('will-quit', () => {
  globalShortcut.unregisterAll();
});
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Text Editor
สร้าง Text Editor ด้วย Electron:
- เปิด/บันทึกไฟล์
- Syntax highlighting (ใช้ highlight.js หรือ CodeMirror)
- Find & Replace
- Multiple tabs
- Recent files

### แบบฝึกหัดที่ 2: File Manager
สร้าง File Manager:
- Browse directories
- Copy/Move/Delete files
- Preview files
- Quick search

### แบบฝึกหัดที่ 3: System Monitor
สร้าง System Monitor:
- CPU/Memory usage (ใช้ `os` module)
- Process list
- Network usage
- Graphs ด้วย Chart.js

### แบบฝึกหัดที่ 4: Screenshot App
สร้าง Screenshot Tool:
- Capture screen/window
- Annotate screenshots
- Save/Copy to clipboard
- Upload to cloud

---

## สรุป

Electron เป็น framework ที่ทรงพลังสำหรับการสร้าง desktop application ด้วย web technologies ในบทนี้เราเรียนรู้:

1. **Main Process** - จัดการ app lifecycle, native APIs
2. **Renderer Process** - UI ด้วย web technologies
3. **IPC** - การสื่อสารระหว่าง processes
4. **contextBridge** - Bridge ที่ปลอดภัยสำหรับ preload
5. **BrowserWindow** - การสร้างและจัดการ windows
6. **Native Menus** - Application menu, context menu, tray
7. **Dialogs** - Native file dialogs, message boxes
8. **Auto Updater** - อัปเดตอัตโนมัติ
9. **Security** - Best practices สำหรับ Electron security
10. **Packaging** - Build และ distribute ด้วย electron-builder
