---
title: '编程题：实现一个 LazyMan'
date: 2022-05-30T22:16:59+08:00
draft: false
tags: ['编程题']
---

## 题目

实现一个 `LazyMan`，可以按照以下方式调用:

```js
LazyMan('Tony');
// Hi I am Tony

LazyMan('Tony').sleep(10).eat('lunch');
// Hi I am Tony
// 等待了10秒...
// I am eating lunch

LazyMan('Tony').eat('lunch').sleep(10).eat('dinner');
// Hi I am Tony
// I am eating lunch
// 等待了10秒...
// I am eating diner

LazyMan('Tony')
  .eat('lunch')
  .eat('dinner')
  .sleepFirst(5)
  .sleep(10)
  .eat('junk food');
// Hi I am Tony
// 等待了5秒...
// I am eating lunch
// I am eating dinner
// 等待了10秒...
// I am eating junk food
```

## 思路

总共需要实现三个函数：一个同步的 `eat`，两个异步的 `sleep`、`sleepFirst`。

目前可公开情报：

- 这些函数需要能够 **链式调用**，所以在执行完毕，要返回这个对象
- 异步函数会延迟之后的函数执行
- `sleepFirst` 需要先执行

为了实现异步函数阻塞之后的函数执行，我们先用一个队列来储存函数的调用，然后依序执行。

![队列](https://findmio.oss-cn-hangzhou.aliyuncs.com/blog/dc130be655510a7afa70235cf7d505bf.png)

```javascript
class Man {
  constructor(name) {
    this.queue = [];
    console.log(`Hi I am ${name}`);
    // 创建一个微任务，在同步代码执行完后开始处理任务队列
    queueMicrotask(() => {
      this.next();
    })
  }
  eat(food) {
    this.queue.push(() => {
      console.log(`I am eating ${food}`);
      this.next();
    });
    // 返回 this，实现链式调用
    return this;
  }
  sleep(time) {
    this.queue.push(() => {
      setTimeout(() => {
        console.log(`等待了${time}秒...`);
        // 在定时结束后开始执行下一个任务
        this.next();
      }, time * 1000);
    });
    return this;
  }
  sleepFirst(time) {
    // 插入到队列的最前面
    this.queue.unshift(() => {
      setTimeout(() => {
        console.log(`等待了${time}秒...`);
        this.next();
      }, time * 1000);
    });
    return this;
  }
  next() {
    this.queue.shift()?.();
  }
}

function LazyMan(name) {
  return new Man(name);
}
```