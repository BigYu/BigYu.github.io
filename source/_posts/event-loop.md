---
title: javascript event loop
date: 2019-02-05 14:47:19
tags: 
- javascript
---

## 单线程的JavaScript

- 为什么？javascript = ecmascript + domapi, 作用是和用户交互和操作dom，所以被设计成单线程：否则，比如说一个线程在一个dom节点上添加了内容，另一个线程删除了这个节点，就乱套了。
- worker线程？为了利用多核cpu，web worker标准允许JavaScript脚本创建worker线程，但是不与上面矛盾
  - 子线程完全受主线程控制。
  - 子线程不得操作dom

## JavaScript异步执行机制

- 同步任务： 在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行下一个任务。
- 异步任务： 不进入主线程，而进入taskqueue的任务。只有任务队列通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。

异步执行的机制为：
1. 所有同步任务在主线程上执行，形成执行栈（execution context stack）
2. 主线程之外，有任务队列（task queue）。只要异步任务有了运行结果，就在任务都列中放置一个事件。（比如   mouse clicks，keypresses，network events）
3. 当执行栈中的任务调用完毕，系统就会读取任务队列，看看里面有哪些事件。队形的异步任务，结束等待，进入主线程，开始执行。

## 任务

### 宏任务 MacroTask task
setTimeout,setInterval,setImmediate,requestAnimationFrame,I/O,UI rendering

### 微任务 MicroTask job
process.nextTick,promise,Object.observe,MutationObserver

- 每个浏览器环境，至多一个eventloop
- 一个eventloop有一个或多个taskqueue，仅有一个MicroTask queue
- 微任务收到了特殊的待遇，一旦全局任务执行完，马上开始执行完整个微任务队列。
