---
title: "在nodejs中深度打印对象"
date: 2022-11-25T10:30:15+08:00
draft: false
tags: ['nodejs', '小技巧']
---

```jsx
const obj = {
  foo: {
    bar: {
      foo: {
        bar: 'https://findmio.github.io/post/skills/在nodejs中深度打印对象/',
        foo: undefined
      }
    }
  }
};
```

如何在  `nodejs` 中打印这样一个对象？

直接使用 `console.log` 会有什么问题

```
{ foo: { bar: { foo: [Object] } } }
```

对象的嵌套层级超过三层，就会显示为 `Object` 

那么该如何完整的打印出这个对象呢？

## 方法一：`JSON.stringify()`

众所周知，此方法是将 `JavaScript` 对象或值转换为 `JSON` 字符串，直接调用的返回值可读性不是很好

```jsx
console.log(JSON.stringify(obj));
```

![https://findmio.oss-cn-hangzhou.aliyuncs.com/blog/97947077d72e3ae2e569b994f825f13b.png](https://findmio.oss-cn-hangzhou.aliyuncs.com/blog/97947077d72e3ae2e569b994f825f13b.png)

可以使用 第三个参数 `space` （缩进用的空白字符串，如果为数字，则代表有多少个空格），来进行美化

```jsx
console.log(JSON.stringify(obj, null, 2));
```

![https://findmio.oss-cn-hangzhou.aliyuncs.com/blog/45e6516dcd87fb0af1ba46e33f07edfe.png](https://findmio.oss-cn-hangzhou.aliyuncs.com/blog/45e6516dcd87fb0af1ba46e33f07edfe.png)

缺点：

- 没有对输出结果的值高亮
- [JSON.stringify()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify#%E6%8F%8F%E8%BF%B0) 会对一些属性进行处理，比如忽略掉 `undefined`
  、函数以及 `symbol` 值

## 方法二：`console.dir()` 和 `util.inspect`

这两个函数的参数是一样的，比较常用的有，[查看完整参数](https://nodejs.org/api/util.html#utilinspectobject-options)

- `showHidden` ：如果为 `true`，则格式化结果中会显示隐藏的不可枚举的符号和属性。默认为 `false`
- `depth`：格式化对象时进行递归的次数，如果要使其无限递归，可传入 `null` 或者 `Infinity`。默认为 `2`
- `colors`：如果为 `true`，输出带 ANSI 颜色的代码。默认为 `false`

```jsx
// 需要导入 nodejs 核心模块 util
console.log(util.inspect(obj, { depth: null, colors: true }));
console.dir(obj, { depth: null, colors: true });
```

![https://findmio.oss-cn-hangzhou.aliyuncs.com/blog/a1244dd02f921dc543255fc745ee16f0.png](https://findmio.oss-cn-hangzhou.aliyuncs.com/blog/a1244dd02f921dc543255fc745ee16f0.png)

## 参考链接：

[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)

[https://nodejs.org/api/util.html#utilinspectobject-options](https://nodejs.org/api/util.html#utilinspectobject-options)

