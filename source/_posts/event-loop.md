---
title: JavaScript Event Loop 事件循环机制
date: 2021-1-6
---

JavaScript是一门单线程的语言，他的异步和多线程的实现是通过 Event Loop 事件循环机制来实现的。大体有三个部分组成：

* 调用栈(call stack)
* 消息队列(Message Queue)
* 微任务队列(Microtask Queue)
---

Event Loop 开始时会从全局代码开始，一行一行执行，遇到函数调用，会把他压入调用栈中，被压入的函数叫做`帧(Farme)`，当函数返回后会从调用栈中弹出。

```
// eg1:

function f1() {
  console.log(1);
}

function f2() {
  console.log(2);
  f1();
  console.log(3);
}

f2(); // 2 1 3
```

1. 首先把`f2()`压入调用栈中执行其中的代码；
2. 遇到`console.log(2);`，将其压入栈中并执行打印出`2`后弹出； // 2
3. `f1()`的调用被压入栈中，执行它里面的代码；
4. `console.log(1);`被压入栈执行打印出`1`后弹出； // 1
5. `f1()`执行完毕弹出；
6. `console.log(3);`被压入栈执行打印出`3`弹出； // 3
7. `f2()`执行完毕弹出，整个调用栈清空；
---

JavaScript中的(~~宏任务~~)异步操作比如`setTimeout`、`setInterval`中的回调函数会入队到消息队列中成为消息，消息会在调用栈清空的时候执行。

```
// eg2:

function f1() {
  console.log(1);
}

function f2() {
  setTimeout(() => {
    console.log(2);
  }, 0);
  f1();
  console.log(3);
}

f2(); // 1 3 2
```

1. 首先把`f2()`压入调用栈中执行其中的代码；
2. 遇到`setTimeout(() => { console.log(2); }, 0)`，将其压入栈时，他里面的回调函数`console.log(2);`入队到消息队列中，等到调用栈清空的时候再执行；
3. `f1()`的调用被压入栈中，执行它里面的代码；
4. `console.log(1);`被压入栈执行打印出`1`后弹出； // 1
5. `f1()`执行完毕弹出；
6. `console.log(3);`被压入栈执行打印出`3`弹出； // 3
7. `f2()`执行完毕弹出，此时调用栈为空，消息队列中的消息`console.log(2);`会被压入调用栈中并执行； // 2
---

使用`promise`、`async await`创建的(~~微任务~~)异步操作会被加入到微任务队列中，他会在调用栈被清空的时候立即执行，并且处理期间新加入的微任务也会一同执行。

```
var p = new Promise(resolve => {
  console.log(4);
  resolve(5);
});

function f1() {
  console.log(1);
}

function f2() {
  setTimeout(() => {
    console.log(2);
  }, 0);
  f1();
  console.log(3);
  p.then(resolved => {
    console.log(resolved);
  })
  .then(() => {
    console.log(6);
  });
}

f2(); // 4 1 3 5 6 2
```

1. 构造函数`new Promise`首先被压入调用栈，执行其中的代码；
2. `console.log(4);`被压入栈执行打印出`4`后弹出； // 4
3. `resolve(5);`被压入栈执行后弹出，构造函数`new Promise`执行完毕弹出；
4. `f2()`压入调用栈中执行其中的代码；
5. 遇到`setTimeout(() => { console.log(2); }, 0)`，将其压入栈时，他里面的匿名回调`console.log(2);`入队到消息队列中，等到调用栈清空的时候再执行；
6. `f1()`的调用被压入栈中，执行它里面的代码；
7. `console.log(1);`被压入栈执行打印出`1`后弹出； // 1
8. `f1()`执行完毕弹出；
9. `console.log(3);`被压入栈执行打印出`3`弹出； // 3
10. `p.then`的两个回调函数会入队到微任务队列中；
11. `f2()`执行完毕弹出，此时调用栈为空，立即执行微任务队列中的任务`console.log(resolved);`、`console.log(6);`，将其压入调用栈中并执行； // 5 6
12. 最后压入并执行消息队列中的消息`console.log(2);`； // 2