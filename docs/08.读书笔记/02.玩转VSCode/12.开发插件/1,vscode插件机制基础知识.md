---
title: 1,vscode插件机制基础知识
date: 2022-01-17 17:54:47
permalink: /pages/12da24/
categories:
  - 读书笔记
  - 玩转VSCode
  - 开发插件
tags:
  - 
---
## VS Code 插件架构

​	VS Code 是通过 Electron 实现跨平台的，而 Electron 则是基于 Chromium 和 Node.js，比如 VS Code 的界面，就是通过 Chromium 进行渲染的。

##### VS Code 是多进程架构

主要有五类进程

- main process：第一次被启动时会创建的**主进程**
- Renderer Process：每个窗口，都会创建一个**渲染进程**
- Extension Host：每个窗口中执行插件的进程
- Debug Adapter ：为调试器专门创建的调试进程
- Language Server：语言支持进程

他们大致架构如下，主进程和渲染进程是一对多的关系，渲染进程和插件进程是一对一的关系，彼此通过IPC或者RPC进行进程间通信，调试进程由node根据`v8 Debugger Protocol`协议实现，与渲染进程一对一关系；而语言支持进程则依据`vscode Language Protocol`实现，嵌入插件进程。

![img](https://tva1.sinaimg.cn/large/008i3skNly1gslzfiuvzmj30oh0f5abc.jpg)

在上图中，绿色的就是插件进程 Extension Host 了。VS Code 创建 Extension Host 进程的方式，就是创建一个新的 Electron 进程，并且以 Node.js 的形式运行。也就是说，这个进程就是一个完整的 Node.js 进程，Node.js 版本就是你使用的 Electron 中的 Node.js 。

对于一个插件作者而言，你可以无需关心 VS Code 的这套架构，在书写 VS Code 插件的时候，你只需知道：

- 首先，这个插件就是一个 Node.js 应用；
- 其次，在这个 Node.js 应用中，你可以直接访问 VS Code 的 API，通过这些 API 来操作 VS Code，你并不需要知道插件进程是怎么跟渲染进程通讯的；
- 最后，每当你打开一个窗口时，VS Code 会为这个窗口创建插件进程，并且按需要激活插件。也就是说，同一时间，你的代码有可能被运行多次。