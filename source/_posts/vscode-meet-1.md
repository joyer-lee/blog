---
title: vscode源码初识（一）
tags: []
id: '93'
categories:
  - - vscode源码解读
  - - 前端
date: 2020-11-11 15:16:38
---



## 导读

_公司规划要做一个IDE，经过前期产品调研，确认以vscode源码为研究对象，基于vscode开发一款新的IDE产品，所以我有较长的时间需要啃懂vscode相关的技术实现，由于前期对vscode涉及的相关技术，如typescript、electron、nodejs缺乏系统了解，也无相关开发经验，因此读懂vscode源码对我来说是个大工程，希望通过博客能记录学习过程、梳理相关概念、总结技术难点，帮助我更好的上手IDE开发_

> 本系列前期仅能对vscode源码进行一些相对浅显的理解，后期随着项目深入，将由浅入深的分享一些开发细节。

## 本章目标

前期研究目标

> 1.  启动vscode源码
> 2.  了解vscode源码基本结构
> 3.  了解vscode模块与模块之前的运作
> 4.  了解导航栏的基本操作

## vscode源码运行

不管有没有vscode相关技术，首先需要先认识一下vscode源码的基本结构和启动流程，便于在随后的学习过程中有个侧重点。 关于启动vscode源码，网上有很多参考教程，我这里大致做一个流程总结。具体参考[在这里](https://www.cnblogs.com/liulun/p/11037537.html "在这里")。

```txt
安装git，nodejs和yarn
安装Python27，3.x版本的不行，确保它在你的环境变量里；
安装gulp
npm install --global gulp-cli

安装windows build tools：
npm install --global windows-build-tools --2017

安装node-gyp
npm install -g node-gyp

**用管理员的方式打开powershell**，不是管理员身份不行
在源码根目录下执行：
yarn 安装相关包
yarn watch 编译项目

yarn web 在浏览器中打开

webstorm可通过配置启动项启动vscode客户端，由于我使用的是vscode编辑器，目前是通过双击打开vscode源码目录下的scripts/code.bat文件打开客户端，mac用户可双击scripts/code.sh文件
```

## vscode基本认识

### 核心技术

![](../images/uploads/sites/2/2020/11/QQ截图20201111100348.png) - 使用 Web 技术来编写 UI，用 chrome 浏览器内核来运行 - 使用 NodeJS 来操作文件系统和发起网络请求 - 使用 NodeJS C++ Addon 去调用操作系统的 native API

### 目录结构

```txt
.
├── azure-pipelines.yml
├── build/
├── extensions/
├── gulpfile.js
├── out/
├── package.json
├── product.json
├── resources/
├── scripts/
├── src/
│   ├── main.js
│   └── vs
│       ├── base/
│       ├── code/
│            ├── browser/
│                 ├── workbench.ts
│                 └── workbench/workbench.html
│            ├── electron-browser/
│                 ├── workbench.js
│                 └── workbench/workbench.html
│            └── electron-main/
│                 ├── main.ts
│                 └── window.ts
│       ├── editor/
│       ├── platform/
│       ├── server/
│       └── workbench/
└── test/ //放的是各种自动化、冒烟、UI 测试脚本，这里值得学习和研究下
```

目录解析 - azure-pipelines.yml，它是一个 CI/CD 的配置，自动测试、构建、打包 - build/，这里面放的是 VS Code 项目的构建工具，相对来说还是比较复杂的，主要是因为它顾及了 Linux/Mac/Windows 三个平台 - extensions/，VS Code 的内置模块，包含各种语言高亮的 LSP 相关模块 - gulpfile.js，构建脚本，暂时不用细看，可以关注 package.json 的 scripts，里面放着一些程序的快捷启动方式，而且针对内存溢出做了防御，如 --max\_old\_space\_size=4095 - out/，构建的结果都放在这个目录下 - package.json，需要着重看看 main 和 scripts 两个字段 - product.json，如果你想根据 VS Code 进行二次开发，建立自己的品牌，建议搞懂这个文件，因为你需要修改它 - resource/，打包构建生成安装包（exe/dmg/deb 等）的时候需要依赖的额外资源 - scripts/，开发过程各种会用到的脚本，用的比较多的可能是 ./scripts/code.sh - test/，放的是各种自动化、冒烟、UI 测试脚本，这里值得学习和研究下 - src/，核心源码

### 结构细分

一下内容均摘抄至[《从 VSCode 看大型 IDE 技术架构》](https://zhuanlan.zhihu.com/p/96041706 "从 VSCode 看大型 IDE 技术架构")。 1. 隔离内核 (src) 与插件 (extensions)，内核分层模块化

```txt
/src/vs：分层和模块化的 core
/src/vs/base: 通用的公共方法和公共视图组件
/src/vs/code: VSCode 应用主入口
/src/vs/platform：可被依赖注入的各种纯服务
/src/vs/editor: 文本编辑器
/src/vs/workbench：整体视图框架
/src/typings: 公共基础类型
/extensions：内置插件
```

2.  每层按环境隔离 内核里面每一层代码都会遵守 electron 规范，按不同环境细分文件夹:

```txt
common: 公共的 js 方法，在哪里都可以运行的
browser: 只使用浏览器 API 的代码，可以调用 common
node: 只使用 NodeJS API 的代码，可以调用 common
electron-browser: 使用 electron 渲染线程和浏览器 API 的代码，可以调用 common，browser，node
electron-main: 使用 electron 主线程和 NodeJS API 的代码，可以调用 common， node
test: 测试代码
```

3.  内核代码本身也采用扩展机制: Contrib 可以看到 /src/vs/workbench/contrib 这个目录下存放着非常多的 VSCode 的小的功能单元：

```txt
├── backup
├── callHierarchy
├── cli
├── codeActions
├── codeEditor
├── comments
├── configExporter
├── customEditor
├── debug
├── emmet
├──....中间省略无数....
├── watermark
├── webview
└── welcome
```

Contrib 有一些特点：

```txt
Contrib 目录下的所有代码不允许依赖任何本文件夹之外的文件
Contrib 主要是使用 Core 暴露的一些扩展点来做事情
每一个 Contrib 如果要对外暴露，将API 在一个出口文件里面导出 eg: contrib/search/common/search.ts
一个 Contrib 如果要和另一个 Contrib 发生调用，不允许使用除了出口 API 文件之外的其它文件
接上一条，即使 Contrib 事实上可以调用另一个 Contrib 的出口 API，也要审慎的考虑并尽量避免两个 Contrib 互相依赖
```

## vscode启动流程

具体可参照[《从 VSCode 看大型 IDE 技术架构》](https://zhuanlan.zhihu.com/p/96041706 "从 VSCode 看大型 IDE 技术架构")关于启动流程的精简链路解读。 **简要描述：** - 入口是 src/main.js - 主进程（Main Process） 的实际调用路径是: `main.js -> vs/code/electron-main/main.ts -> vs/code/electron-main/window.ts` 在 `window.ts` 启动了一个`BrowserWindow` 加载了 `vs/code/electron-browser/workbench/workbench.html` - 渲染进程（Renderer Process）的实际路径是： `vs/code/electron-browser/workbench/workbench.html -> vs/code/electron-browser/workbench/workbench.js` 核心逻辑在 `workbench.js` 中，而 src 下还有一个核心目录 `browser`，它是 Web 版本的启动入口

### 未完待续。。。

## vscode代码调试

## vscode导航栏调试分析