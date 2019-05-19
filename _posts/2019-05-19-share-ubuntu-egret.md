---
layout: post
title: 在Ubuntu上开发Egret应用
category: Share 
description: 当年基于Flash的WebGame红红火火的日子仿佛还在昨天，如今Flash已经势微，各大浏览器都不默认运行Flash插件，Adobe官方更是宣布2020年停止对Flash的更新。如今基于Canvas和Webgl的富媒体方案才是各开发商的首选。
keywords: Node.jsle Promise
---
当年基于Flash的WebGame红红火火的日子仿佛还在昨天，如今Flash已经势微，各大浏览器都不默认运行Flash插件，Adobe官方更是宣布2020年停止对Flash的更新。如今基于Canvas和Webgl的富媒体方案才是各开发商的首选。  
  
其实相对于Flash的时代，如今的工作流开发工具并不如Flash时那么成熟，Flash时代有Adobe Flash CS系列，Adobe Flash Builder作为开发IDE，还有各个公司基于Adobe Flash Air开发的工具。  
当然既然Flash已经快被淘汰，新技术就一定会驱动大家开发一套合适的工作流。国内有Egret, LayaBox都开发出了游戏的H5游戏引擎。Cocos2d-x框架也很优秀。Adobe Flash CS系列的续作Adobe Flash Animate系列也支持了导出H5动画，它是基于create.js和sound.js。开源框架有Pixi.js。  
最近做外包项目需要用到Egret。最近我刚把笔记本装了Ubuntu, 虽然Egret官方只有Windows和Mac下安装Egret引擎的介绍，但是我还是找到了在Ubuntu上开发的方法。
## IDE
打开Egret Wing，第一眼就觉得这UI好熟悉，和VSCode很像。其实Egret Wing是有基于VSCode开发的。那么，Ubuntu下就用VSCode作为开发IDE吧。  
## 引擎安装
Egret核心代码是开源的，代码在github上[egret-core](https://github.com/egret-labs/egret-core)。Egret核心库是基于node.js的。所以首先我们需要安装node.js。
### 安装Node.js
Ubuntu上使用apt install nodejs即可。如果需要下载最新版我们先上https://nodejs.org/ node.js官网，下载最新版本的Source Code（推荐LTS的版本）。解压源码后进入目录执行命令：
```shell
./configure
```
然后执行命令:
```shell
make && make install
```
这个过程可能很长请耐心等待。  
安装完成后配置环境变量：  
执行命令编辑：
```shell
vim ~/.profile
```
添加环境变量：
```shell
export NODE_HOME=/home/node  #你的node目录的路径

export PATH=$PATH:$NODE_HOME/bin 

export NODE_PATH=$NODE_HOME/lib/node_modules
```
使环境变量生效：  
```shell
source ~/.profile
```
执行代码查看node安装是否成功
```shell
node -v
```
此时应该输出你安装的node版本号。  
由于egret采用typescript开发，我们还需要安装typescript库。执行代码：
```shell
npm install -g typescript
```
  
### 安装egert核心库
代码地址https://github.com/egret-labs/egret-core安装时我们可以直接clone，或者下载release的压缩包。在下载好之后解压，比如我解压后是 egret-core-5.2.19 这个文件夹。首先将文件夹改名为：
egret-core，否则安装后会报错。然后进入egret-core文件夹执行命令：
```shell
npm install -g
```
最后执行命令：
```shell
egret info
```
查看输出信息是否安装了正确版本egret引擎。

## 代码调试
首先我们创建一个egret项目，执行代码：
```shell
 egret create 项目名字
```
 创建一个新的egret项目。使用VSCode打开项目。Egret开发是使用的TypeScrit语言，且运行在浏览器中。我们需要先安装Chrome浏览器。然后再安装Debugger for Chrome插件。按F5开启调试，IDE会提示调试方式，此时选择chrome，此时在项目目录下会自动创建.vscode文件夹和.vscode/launch.json文件。将launch.json文件的内容编辑为：
 ```json
 {
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Chrome 调试",
            "type": "chrome",
            "request": "launch",
            "url": "http://127.0.0.1:3000/index.html",
            "sourceMaps": true,
            "webRoot": "${workspaceRoot}",
            "preLaunchTask":"build",
            "userDataDir":"${tmpdir}"
        }
    ]
}
```
此时按F5调试会报错，并且在.vscode目录下会生成一个tasks.json文件。将tasks.json文件编辑为：
```json
{
   "version": "2.0.0",
   "tasks": [
       {
           "label": "build",
           "type": "shell",
           "command": "egret",
           "args": ["build","-sourcemap"],
           "problemMatcher": "$tsc"
       },
       {
           "label": "clean",
           "type": "shell",
           "command": "egret",
           "args": ["build","-e"],
           "problemMatcher": "$tsc"
       },
       {
           "label": "publish",
           "type": "shell",
           "command": "egret",
           "args": ["publish"],
           "problemMatcher": "$tsc"
       }
   ]
}
```
由于是采用TypeScript开发，编译为js文件后需要调试的话需要开启sourceMap。我们需要将项目的tsconfig.json文件编辑为：
```json
{
    "compilerOptions": {
        "target": "es5",
        "outDir": "bin-debug",
        "experimentalDecorators": true,
        "lib": [
            "es5",
            "dom",
            "es2015.promise"
        ],
        "types": []
    },
    "include": [
        "src",
        "libs"
    ]
}
```
这样我们的开发环境就配置好了。我们调试前需要先开启本地服务：
```shell
egret run
```
启动好之后，在vscode中按F5就可以调试了，并且可以直接在ts代码中断点调试。
## 总结
上面的步骤只是让我们有在Ubuntu上开发egret应用的能力，但是并不能体验到egret官方提供的完整的工作流工具。这些工具只能在windows或者mac下体验了。