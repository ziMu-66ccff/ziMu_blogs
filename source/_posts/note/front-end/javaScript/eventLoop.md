---
title: 理解事件循环
subtitle: 浏览器中的事件循环
link: eventLoop
catalog: true
date: 2023-02-16 10:20:17
tags:
  - 前端
  - JS高级
  - 浏览器
  - 事件循环
categories:
  - [笔记, 前端, JavaScript]
---

# 浏览器中的事件循环

## 同步与异步

浏览器是**单线程**的，所以为了不堵塞代码的运行，我们将任务分为了**同步任务**，和**异步任务**（`setTimeout`和`setInterval`、`axios`、事件绑定等这种带回调函数的），那么同步任务和异步任务又是按照怎样的顺序进入主线程执行的呢，废话不多说，直接上图好叭 😎
![事件循环](https://i.328888.xyz/2023/02/16/mJuAZ.png)

先看一段代码，我们会结合这张图和代码来先简单认识一下同步和异步的执行

![演示代码](https://i.328888.xyz/2023/02/16/mKZNF.png)
执行的结果很简单 start -> end -> 时间到了

1. 整个 `script` 代码进入主线程，遇到**同步任务** `start` 的打印，执行打印
2. 遇到**异步任务** `setTimeout`，将 setTimeout 放到 **Event Table** 开始计时（注：setTimeout 回调函数被调用的前提是时间到了，所以是在 **Event Table** 中等待计时结束，如果是其他的回调，例如 on 绑定的事件，则是在 Table 中等待事件被触发）
3. 遇到**同步任务** `end` 的打印，执行打印，同步任务执行完毕，**monitoring process** 进程检测到主线程为空，开始去 **Event Queue** 那里检查是否有等待被调用的函数
4. setTimeout 的计时结束，将其放到 **Event Queue** 中（**monitoring process** 进程检测到 **Event Queue** 存在等待被调用的函数，就将 setTimeout 的回调函数放进主线程执行）

是不是觉得还是很简单的，那么我们现在开始上难度，将异步任务细分为**宏任务**和**微任务**，进一步认识事件循环

## 宏任务，微任务

**异步任务**可以细分为**宏任务**，**微任务**

宏任务（_task_）大概包括：

- script(整体代码)
- setTimeout
- setInterval
- setImmediate
- I/O
- UI render

微任务（_jobs_）大概包括：

- process.nextTick
- Promise.then( )
- Async/Await(实际就是 promise)
- MutationObserver(html5 新特性)

## 简述事件循环：

执行**宏任务**，然后执行该**宏任务产生的微任务**，若**微任务在执行过程中产生了新的微任务**，则**继续执行微任务**，微任务执行完毕后，**再回到宏任务**中进行下一轮循环。

![事件循环](https://i.328888.xyz/2023/02/16/mKIJw.png)

光这样大家可能不会很理解，那么我们上例子，结合例子来分析讲解

![代码实列](https://i.328888.xyz/2023/02/16/mK7Xy.png)

第一轮事件循环分析如下：

- 整体`script`代码作为第一个宏任务进入主线程，遇到同步任务`console.log`，输出 1。
- 遇到宏任务`setTimeout`，其回调函数被分发到**宏任务 Event Queue**中（注：setTimeout 是先被放到**Event Table**中进行计时的，等到时间到了，其回调函数才放到宏任务**Event Queue**中，并不是直接就放到宏任务**Event Queue**）。我们暂且记为*setTimeout1*。
- 遇到微任务`process.nextTick()`，其回调函数被分发到**微任务 Event Queue**中。我们记为*process1*。
- 遇到 Promise，new Promise 里面的参数函数作为**同步任务**直接执行，输出 7。then 里面的回调函数作为微任务被分发到**微任务 Event Queue 中**。我们记为*then1*。
- 又遇到了宏任务`setTimeout`，其回调函数被分发到**宏任务 Event Queue**中，我们记为*setTimeout2*。

| 宏任务 Event Queue | 微任务 Event Queue |
| ------------------ | ------------------ |
| setTimeout1        | process1           |
| setTimeout2        | then1              |

- 上表是第一轮事件循环宏任务结束时各 Event Queue 的情况，此时已经输出了 1 和 7。

**第一轮宏任务结束**后，开始处理产生的**微任务**

- 执行 process1,输出 6。
- 执行 then1，输出 8。

---

**第一轮事件循环正式结束**，接着开始**第二轮**，从**宏任务 Event Queue**中取出**宏任务 setTimeout1**开始处理：

- 首先执行同步任务输出 2。接下来遇到了微任务`process.nextTick()`，同样将其分发到**微任务 Event Queue**中，记为*process2*。
- `new Promise`参数里的参数函数作为**同步任务**立即执行输出 4，**微任务**`then`也分发到**微任务 Event Queue**中，记为*then2*

| 宏任务 Event Queue | 微任务 Event Queue |
| ------------------ | ------------------ |
| setTimeout2        | process2           |
|                    | then2              |

- 上表是第二轮事件循环宏任务结束时各 Event Queue 的情况，此时已经输出了 2 和 4

**第二轮宏任务结束**，开始处理产生的**微任务**：

- 执行 process2,输出 3。
- 执行 then2，输出 5。

---

**第二轮事件循环正式结束**，接着开始**第三轮**，从**宏任务 Event Queue**中取出宏任务`setTimeout2`开始处理：

- 首先执行同步任务输出 9。接下来遇到了微任务`process.nextTick()`，同样将其分发到**微任务 Event Queue**中，记为*process3*。
- `new Promise`参数里的参数函数作为同步任务立即执行输出 11，微任务`then`也分发到**微任务 Event Queue**中，记为*then3*

| 宏任务 Event Queue | 微任务 Event Queue |
| ------------------ | ------------------ |
|                    | process3           |
|                    | then3              |

- 上表是**第三轮事件循环宏任务结束**时各 Event Queue 的情况，此时已经输出了 9 和 11。

**第三轮宏任务结束**，开始处理产生的**微任务**：

- 执行 process3,输出 10。
- 执行 then3，输出 12。

第三次事件循环结束，整个事件循环结束，共经历了三次循环，完整的输出为 1，7，6，8，2，4，3，5，9，11，10，12

## async/await 执行顺序

首先我们来看一段代码：
![代码实列](https://i.328888.xyz/2023/02/16/mMF2Q.png)

输出结果为：script start => async2 end => Promise => script end => async1 end => promise1 => promise2 => setTimeout

如果 await 后面直接跟的为一个**变量**，比如：await 1；这种情况的话相当于**直接把 await 后面的代码注册为一个微任务**，可以简单理解为 **promise.then(await 下面的代码)**。然后跳出 async1 函数，执行其他代码，当遇到 promise 函数的时候，会注册 promise.then()函数到微任务队列，注意此时微任务队列里面已经存在 await 后面的微任务。所以这种情况会先执行 await 后面的代码（async1 end），再执行 async1 函数后面注册的微任务代码(promise1,promise2)。

我们再来看另外一段代码：

![代码实列](https://i.328888.xyz/2023/02/16/mMSpX.png)

输出结果为： script start => async2 end => Promise => script end => async2 end1 => promise1 => promise2 => async1 end => setTimeout

如果 await 后面跟的为**一个异步函数的调用**，此时执行完 await**并不先把 await 后面的代码放到微任务队列中去，而是执行完 await 之后，直接跳出 async1 函数，执行其他代码**。然后遇到 promise 的时候，把 promise.then 注册为微任务。**其他代码执行完毕后，需要回到 async1 函数去执行剩下的代码，然后把 await 后面的代码注册到微任务队列当中**，注意此时微任务队列中是有之前注册的微任务的。所以这种情况会先执行 async1 函数之外的微任务(promise1,promise2)，然后才执行 async1 内注册的微任务(async1 end). **可以理解为，这种情况下，await 后面的代码会在本轮循环的最后被执行。**

总的来说：如果 await 后面是一个变量，则直接把 await 后面的代码放到微任务队列里面。如果后面为一个异步函数的调用，则等到本轮循环中宏任务执行完毕后再把 await 后面的代码放到微任务队列，也就是说这个时候 await 后面的代码时是本轮循环中的微任务队列中的最后一个微任务，会在本轮循环的最后被执行。
