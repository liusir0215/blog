---
title: 事件队列
top: false
cover: false
toc: true
mathjax: true
tags:
  - nodejs
  - event
categories:
  - nodejs
date: 2020-07-16 20:28:31
password:
summary:
---
## 运行顺序
1. 定时器 timer：setTimeout、setInterval的调度回调函数
2. 待定回调 pending callback：执行延迟到下一次循环迭代的i/o回调。例如，如果 TCP 套接字在尝试连接时接收到 ECONNREFUSED，则某些 *nix 的系统希望等待报告错误
3. idle、prepare：仅系统内部使用
4. 轮询 poll：检索新的 I/O 事件;执行与 I/O 相关的回调（几乎所有情况下，除了关闭的回调函数，那些由计时器和 setImmediate() 调度的之外），其余情况 node 将在适当的时候在此阻塞。
当事件循环进入 轮询 阶段且 没有被调度的计时器时 ，将发生以下情况：
+ 如果 轮询 队列 是空的 ，还有两件事发生
	
	+ 如果脚本被 setImmediate() 调度，则事件循环将结束 轮询 阶段，并继续 检查 阶段以执行那些被调度的脚本。

	+ 如果脚本 未被 setImmediate()调度，则事件循环将等待回调被添加到队列中，然后立即执行。
5. 检测 check：setImmediate() 回调函数在这里执行。进入这个阶段的条件是轮询队列是空且执行了setImmediate
6. 关闭的回调函数 closed callback：一些关闭的回调函数，如：socket.on('close', ...)。如果套接字或处理函数突然关闭（例如 socket.destroy()），则'close' 事件将在这个阶段发出。否则它将通过 process.nextTick() 发出。

它都将在当前操作完成后处理 nextTickQueue， 而不管事件循环的当前阶段如何

process.nextTick具有不让事件循环继续的优点，适用于让事件循环继续之前，警告用户发生错误的情况

任何时候在给定的阶段中调用 process.nextTick()，所有传递到 process.nextTick() 的回调将在事件循环继续之前解析

process.nextTick它都将在当前操作完成后处理 nextTickQueue， 而不管事件循环的当前阶段如何

nodejs最大回调数

它有一个事件轮询线程负责任务编排，和一个专门处理繁重任务的工作线程池。

Node.js 模块中有如下这些 API 用到了工作线程池：

1. I/O 密集型任务：

	1. DNS：dns.lookup()，dns.lookupService()。
	2. 文件系统：所有的文件系统 API。除 fs.FSWatcher() 和那些显式同步调用的 API 之外，都使用 libuv 的线程池。
2. CPU 密集型任务：
	1. Crypto：crypto.pbkdf2()、crypto.scrypt()、crypto.randomBytes()、crypto.randomFill()、crypto.generateKeyPair()。
	2. Zlib：所有 Zlib 相关函数，除那些显式同步调用的 API 之外，都适用 libuv 的线程池。

	
事件循环阶段通过执行对应回调函数来对客户端请求做出回应，此回调将同步执行，并且可能在完成之后继续注册新的异步请求

事件轮询线程本身并不维护队列，它持有一堆要求操作系统使用诸如 epoll (Linux)，kqueue (OSX)，event ports (Solaris) 或者 IOCP (Windows) 等机制去监听的文件描述符。 这些文件描述符可能代表一个网络套接字，一个监听的文件等等。 当操作系统确定某个文件的描述符发生变化，事件轮询线程将把它转换成合适的事件，然后触发与该事件对应的回调函数

如果运行以下不在 I/O 周期（即主模块）内的脚本，则执行两个计时器的顺序是非确定性的，因为它受进程性能的约束

```javascript
// timeout_vs_immediate.js
setTimeout(() => {
  console.log('timeout');
}, 0);

setImmediate(() => {
  console.log('immediate');
});
```

```shell
$ node timeout_vs_immediate.js
timeout
immediate

$ node timeout_vs_immediate.js
immediate
timeout
```

但是，如果你把这两个函数放入一个 I/O 循环内调用，setImmediate 总是被优先调用：

```javascript
// timeout_vs_immediate.js
const fs = require('fs');

fs.readFile(__filename, () => {
  setTimeout(() => {
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  });
});
```

```shell
$ node timeout_vs_immediate.js
immediate
timeout

$ node timeout_vs_immediate.js
immediate
timeout
```