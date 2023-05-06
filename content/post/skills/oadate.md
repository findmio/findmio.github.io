---
title: "oa date 是什么"
date: 2023-05-06T10:31:14+08:00
draft: true
---

在使用 js 的库 `xlsx` 解析 Excel 表格数据的时候，其中一个日期字段，解析之后是 `45040.3964930556` 这样子，在 `Excel` 中展示的是 `2023/4/24 9:30`



这又是什么奇怪的格式，看起来也不像时间戳啊



## oa date (OLE Automation date) 



`oa date` 使用浮点数来保存时间，分为两个部分，整数部分从 **1899年12月30日** 以来的天数，小数部分是这一天的时间除以24

`js` 中的 `Date` 对象则是从 自 **1970年1月1日** 起经过的毫秒数



## 如何转换

### 1. 使用 `moment.js` 库

`moment` 提供了一个插件 `moment-msdate`，[参考文档](https://momentjs.com/docs/#/plugins/msdate/)

把 `moment` 对象转换为 `oa date`

```typescript
moment().toOADate();  // 一个浮点数
```



把 `oa date` 转换为 `moment` 对象

```typescript
moment.fromOADate(41493); // Wed Aug 07 2013 00:00:00 GMT-0600 (MDT)
```



### 2. 自定义函数

把 `Date` 对象转换为 `oa date`

```typescript
function JSDateToOADate(date: Date) {
    return (date.getTime() / (86400 * 1000)) + 25569;
}
```



把 `oa date` 转换为 `Date`

```typescript
function OADateToJSDate(date: number) {
    return new Date((date - 25569) * 86400 * 1000);
}
```

其中的 1000 是秒到毫秒之间的换算倍数

86400 是 一天之中的总秒数 24 * 60 * 60

25569 则是 1899年12月30日 到 1970年1月1日 间隔的天数