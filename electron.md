## electron
为将公司级服务拓展外部公司使用，BPO​​ 是 ​​Business Process Outsourcing​​ 的缩写，中文通常翻译为 ​​“业务流程外包”​​。
外部的业务需要保证安全性，将原本内部的云智服系统进行electron封装，实现桌面端应用给外部人使用

主进程渲染内部网址


##### electron如何控制进程
##### electron如何反编译
##### electron如何做安全防范
##### electron如何隐藏敏感信息
##### electron如何打包
项目建立初期，使用的是官放的electron-forge，后面因为签名配置和无法选择安装路径，自动更新等方面存在异常，转为使用更简便的的electron-builder。
打包命令
拆分模块打包
```
 "build:pro": "yarn build:renderer && yarn build:preload && dotenv -v NODE_ENV_ELECTRON_VITE=production -- yarn build:main && yarn build:ntf",
```
##### electron更新包版本
```
const updateVersion = (isBeta) => {
  const __filename = fileURLToPath(import.meta.url)  // 获取当前文件的绝对路径
  const __dirname = path.dirname(__filename)

// 获取 package.json 文件中的版本号
  const packageJsonPath = path.resolve(__dirname, '../package.json')
  const packageJsonContent = fs.readFileSync(packageJsonPath, 'utf8')
  const packageJson = JSON.parse(packageJsonContent)

  const currentVersion = packageJson.version

  const newVersion = isBeta ? semver.inc(currentVersion, 'prerelease', 'beta') : semver.inc(currentVersion, 'patch')
// 更新 package.json 文件中的版本号

  packageJson.version = newVersion
  fs.writeFileSync(packageJsonPath, JSON.stringify(packageJson, null, 2))
  console.log(`版本号已从 ${currentVersion} 更新为 ${newVersion}`)
}

export default updateVersion
```

##### electron如何检查更新
在获取到登录态后做一次检查更新，autoUpdater.checkForUpdates自动检查更新，在打包时配置了发布服务器地址，检查更新时用semver包来检查版本号是否变更升级，所以要求在打包构建产物时版本号的变更需要用semver包来升级。
针对升级包监测有无更新做对应的弹窗提示，更新策略，七天内无需强制更新包



import { autoUpdater } from 'electron-updater'
```
 // 获得登录态后检查一次更新
  ipcMain.on(T_IPMainChannel.SYSTEM_LOAD_SUCCESS, () => {
    autoUpdater.checkForUpdates()
  })
```

检查包版本信息后做逻辑判断是否强制更新
```
  // 自动下载后更新提示
  autoUpdater.on('update-downloaded', ({ releaseNotes, releaseName, releaseDate }) => {
    // 判断是否是7天内的更新
    const canUpdateLater = dayjs().subtract(7, 'day').isBefore(dayjs(releaseDate))

    // 七天内允许稍后更新
    const buttons = [t('autoUpdater.restart')]
    if (canUpdateLater) buttons.push(t('autoUpdater.later'))

    const dialogOpts: MessageBoxOptions = {
      type: 'info' as 'info',
      buttons,
      message: process.platform === 'win32' ? releaseNotes as string : releaseName,
      detail: t('autoUpdater.updaterDownloaded'),
    }

    dialog.showMessageBox(dialogOpts).then((returnValue) => {
      if (returnValue.response === 0) {
        autoUpdater.quitAndInstall()
        app.exit()
      }
    })
  })
```
##### 编译模块
webpack 与 vite
项目初期，使用的webpack的编译方式，但是因为webpack本身的编译速度和热更新的问题上，存在一些问题。后来参考electron-vite重构了整个项目。

将main，preload，renderer模块，分别建立vite.config.js文件。
配置vite编译出口到dist文件。
electron main/dist/main.js 启用本地调试模式
受限于electron本身无法热更新main代码，需要监听main文件夹中dist目录的文件变化。做响应式处理。具体代码可参考scripts/watch.mjs
至此，main代码更新会重启客户端，preload需要手动刷新触发，renderer则会自动接入热更新。


##### 拦截代理请求
```
export const useProxyRequest = (win: BrowserWindow) => {
  win.webContents.session.webRequest.onBeforeSendHeaders((details, callback) => {

    let isGo = details.resourceType === 'image'

    if (!isGo) {
      const domain = parseDomain(details.url)
      isGo = !!filterOrigin.find((origin) => domain.includes(origin))
    }
    if (!isGo) {
      import.meta.env.VITE_YZF_MODE !== 'pro' && info(`拦截的url地址: ${details.url}`)
      return callback({ cancel: true })
    }
    callback(details)
  })
}
```

##### 在主进程创建窗口加载url

```
  globalManager.mainWindow
    .loadURL(MAIN_URL)
    .then(() => {
      info(`main window loaded ${MAIN_URL}`)
    })
```

##### electron主进程和渲染进程如何通信
主进程和渲染进程之间的通信是通过 IPC（Inter-Process Communication，进程间通信）机制实现的。
Electron 提供了 ipcMain和 ipcRenderer模块来 facilitate 这种通信


主进程：使用 ipcMain模块
渲染进程：使用 ipcRenderer模块
通信模式主要分为两种：
单向通信：一个进程发送消息，另一个进程接收（类似事件触发）
双向通信：一个进程发送消息并等待回复（类似函数调用）

##### IPC进程通信
一、单向通信不回复
渲染进程->主进程
```

// 渲染进程 (renderer.js)
const { ipcRenderer } = require('electron');

// 发送消息给主进程，不需要回复
ipcRenderer.send('user-action', { 
  action: 'button-clicked', 
  timestamp: Date.now() 
});

// 发送到特定窗口
ipcRenderer.sendToHost('message-to-host', 'Hello from renderer');

// 主进程 (main.js)
const { ipcMain } = require('electron');

// 监听来自渲染进程的消息
ipcMain.on('user-action', (event, data) => {
  console.log('收到渲染进程消息:', data);
  // 执行一些操作，但不需要回复
});

```

主进程->渲染进程
```
// 主进程 (main.js)
const { BrowserWindow } = require('electron');

// 获取窗口实例后，可以向其发送消息
mainWindow.webContents.send('main-process-message', {
  message: 'Hello from main process!',
  status: 'ok'
});

// 渲染进程 (renderer.js)
const { ipcRenderer } = require('electron');

// 监听主进程发来的消息
ipcRenderer.on('main-process-message', (event, data) => {
  console.log('收到主进程消息:', data);
  document.getElementById('status').textContent = data.message;
});
```

二、双向通信需回复
渲染进程->主进程
```
// 渲染进程 (renderer.js)
const { ipcRenderer } = require('electron');

// 发送消息并等待回复
async function getAppVersion() {
  try {
    const version = await ipcRenderer.invoke('get-app-version');
    console.log('应用版本:', version);
    return version;
  } catch (error) {
    console.error('获取版本失败:', error);
  }
}

// 或者使用回调方式（旧API）
ipcRenderer.send('get-app-version-old');
ipcRenderer.once('app-version-reply', (event, version) => {
  console.log('应用版本:', version);
});

// 主进程 (main.js)
const { ipcMain, app } = require('electron');

// 使用 handle() 处理来自渲染进程的调用
ipcMain.handle('get-app-version', async (event) => {
  // 可以执行异步操作
  const someInfo = await getSomeInfo();
  return {
    version: app.getVersion(),
    info: someInfo
  };
});

// 旧API方式（不推荐）
ipcMain.on('get-app-version-old', (event) => {
  event.reply('app-version-reply', app.getVersion());
});
```

##### contextBridge通信
在启用了上下文隔离的情况下，推荐使用 contextBridge来安全地暴露 API。
 渲染进程 (renderer.js) 现在可以通过 window.electronAPI 安全地访问暴露的方法
 主进程ipcMain.handle
```
// preload.js
const { contextBridge, ipcRenderer } = require('electron');

// 向渲染进程暴露安全的 API
contextBridge.exposeInMainWorld('electronAPI', {
  // 应用相关
  getAppVersion: () => ipcRenderer.invoke('get-app-version'),
  
  // 文件操作
  readFile: (filePath) => ipcRenderer.invoke('read-file', filePath),
  writeFile: (filePath, content) => ipcRenderer.invoke('write-file', filePath, content),
  
  // 窗口操作
  minimizeWindow: () => ipcRenderer.send('window-minimize'),
  maximizeWindow: () => ipcRenderer.send('window-maximize'),
  closeWindow: () => ipcRenderer.send('window-close'),
  
  // 事件监听
  onUpdateAvailable: (callback) => {
    ipcRenderer.on('update-available', callback);
    return () => ipcRenderer.removeListener('update-available', callback);
  },

  // 双向通信示例
  sendPing: () => ipcRenderer.invoke('ping'),
  
  // 平台信息
  platform: process.platform
});
// main.js（主进程）
const { ipcMain, app, BrowserWindow } = require('electron');
const fs = require('fs').promises;

// 应用 API
ipcMain.handle('get-app-version', () => {
  return app.getVersion();
});

// 文件操作 API
ipcMain.handle('read-file', async (event, filePath) => {
  try {
    const content = await fs.readFile(filePath, 'utf-8');
    return { success: true, content };
  } catch (error) {
    return { success: false, error: error.message };
  }
});

// 窗口控制 API
ipcMain.on('window-minimize', (event) => {
  const window = BrowserWindow.fromWebContents(event.sender);
  window.minimize();
});

ipcMain.handle('ping', () => {
  return 'pong from main process!';
});
```