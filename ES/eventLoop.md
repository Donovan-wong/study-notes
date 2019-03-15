# Event Loop

[彻底弄懂Event Loop](https://juejin.im/post/5b8f76675188255c7c653811)

## 宏任务和微任务

异步任务的分类，不同的API注册的异步任务会依次进入自身对应的队列中，然后等待**Event Loop**将它们依次压入执行栈中执行

* **宏任务 - macrotask，也叫tasks**

1. * setTimeout
2. setInterval
3. setImmediate (Node独有)
4. requestAnimationFrame (浏览器独有)
5. I/O
6. UI rendering (浏览器独有)

* **微任务，microtask，也叫jobs**

1. * process.nextTick (Node独有)
2. Promise
3. Object.observe
4. MutationObserver

## *浏览器的Event Loop*

**JavaScript代码的具体流程** - **即浏览器的事件循环Event Loop**

1. 执行全局Script同步代码，这些同步代码有一些是同步语句，有一些是异步语句（比如setTimeout等）；
2. 全局Script代码执行完毕后，调用栈Stack会清空；
3. 从微队列microtask queue中取出位于队首的回调任务，放入调用栈Stack中执行，执行完后microtask queue长度减1；
4. 继续取出位于队首的任务，放入调用栈Stack中执行，以此类推，直到直到把microtask queue中的所有任务都执行完毕。**注意，如果在执行microtask的过程中，又产生了microtask，那么会加入到队列的末尾，也会在这个周期被调用执行**；
5. microtask queue中的所有任务都执行完毕，此时microtask queue为空队列，调用栈Stack也为空；
6. 取出宏队列macrotask queue中位于队首的任务，放入Stack中执行；
7. 执行完毕后，调用栈Stack为空；
8. 重复第3-7个步骤；
9. 重复第3-7个步骤；

**在执行微队列microtask queue中任务的时候，如果又产生了microtask，那么会继续添加到队列的末尾，也会在这个周期执行，直到microtask queue为空停止。**

## NodeJS中的Event Loop

[The Node.js Event Loop, Timers, and process.nextTick() | Node.js](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)

### libuv

![](&&&SFLOCALFILEPATH&&&libuv.tiff)

（1）V8引擎解析JavaScript脚本。
（2）解析后的代码，调用Node API。
（3） [libuv库](https://github.com/joyent/libuv) 负责Node API的执行。它将不同的任务分配给不同的线程，形成一个Event Loop（事件循环），以异步的方式将任务的执行结果返回给V8引擎。
（4）V8引擎再将结果返回给用户。


### NodeJS中的宏队列和微队列

![](&&&SFLOCALFILEPATH&&&nodeEventLoop.tiff)

各个阶段执行的任务如下：

* **timers阶段**：这个阶段执行setTimeout和setInterval预定的callback
* **I/O callback阶段**：执行除了close事件的callbacks、被timers设定的callbacks、setImmediate()设定的callbacks这些之外的callbacks
* **idle, prepare阶段**：仅node内部使用
* **poll阶段：获取新的I/O事件**，适当的条件下node将阻塞在这里
* **check阶段**：执行setImmediate()设定的callbacks
* **close callbacks阶段**：执行socket.on(‘close’, ….)这些callbacks

**NodeJS中宏队列主要有4个**

1. Timers Queue
2. IO Callbacks Queue
3. Check Queue
4. Close Callbacks Queue

**NodeJS中微队列主要有2个**：

1. Next Tick Queue：是放置process.nextTick(callback)的回调任务的
2. Other Micro Queue：放置其他microtask，比如Promise等


![](&&&SFLOCALFILEPATH&&&nodeEventloop2.jpeg)

### NodeJS的Event Loop过程：

1. 执行全局Script的同步代码
2. 执行microtask微任务，先执行所有Next Tick Queue中的所有任务，再执行Other Microtask Queue中的所有任务
3. 开始执行macrotask宏任务，共6个阶段，从第1个阶段开始执行相应每一个阶段macrotask中的所有任务，注意，这里是所有每个阶段宏任务队列的所有任务，在浏览器的Event Loop中是只取宏队列的第一个任务出来执行，每一个阶段的macrotask任务执行完毕后，开始执行微任务，也就是步骤2
4. Timers Queue -> 步骤2 -> I/O Queue -> 步骤2 -> Check Queue -> 步骤2 -> Close Callback Queue -> 步骤2 -> Timers Queue ……
5. 这就是Node的Event Loop
