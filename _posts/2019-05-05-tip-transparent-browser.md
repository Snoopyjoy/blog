---
layout: post
title: 半透明的浏览器
category: Tip 
description: 因为工作中经常需要做一些前端的工作，在web页面设计稿的实现过程中总会有误差，导致最后UI审核时总会发现一些细节问题。我就找有没有可以使浏览器半透明的插件，可以把半透明的页面和设计稿进行对比，遗憾的是正式版chrome中是没有这个插件的。于是我找到同样基于Chromium的electron可以实现这个效果。
keywords: 半透明的浏览器
---

因为工作中经常需要做一些前端的工作，在web页面设计稿的实现过程中总会有误差，导致最后UI审核时总会发现一些细节问题。我就找有没有可以使浏览器半透明的插件，可以把半透明的页面和设计稿进行对比，遗憾的是正式版chrome中是没有这个插件的。于是我找到同样基于Chromium的electron可以实现这个效果。
主文件index.js：
```javascript
const { app, BrowserWindow } = require('electron')
let win;
function createWindow(){
  win = new BrowserWindow({ width: 390, height: 667 , opacity:0.8 , webPreferences:{
    enableRemoteModule: true,
    devTools:true,
    webSecurity: false,
    allowRunningInsecureContent: true
  }});
  
  win.on('closed', () => {
    win = null
  })
  
  // Load a remote URL
  win.loadURL('http://172.10.153.218:6303/xxx/');
  win.webContents.openDevTools();
}

app.on( 'ready', createWindow )
```
package文件：
```json
{
  "name": "elec-win",
  "version": "1.0.0",
  "description": "transparent window",
  "main": "index.js",
  "scripts": {
    "start": "electron . --inspect-brk=5858",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "he xiaoliang",
  "license": "ISC",
  "dependencies": {
    "electron": "^5.0.0"
  }
}
```
windows下vscode启动配置
```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
         {
            "name": "Debug Main Process",
            "type": "node",
            "request": "launch",
            "cwd": "${workspaceRoot}",
            "runtimeExecutable": "${workspaceRoot}/node_modules/.bin/electron",
            "windows": {
              "runtimeExecutable": "${workspaceRoot}/node_modules/.bin/electron.cmd"
            },
            "args" : ["."],
            "outputCapture": "std"
          }
    ]
}
```
