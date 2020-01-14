---
title: JS获取调用栈信息
date: 2018-01-22 10:18:30
tags: 
- js
---

由于JS的动态类型，相比Java等强类型开发工具难以支持方法在哪调用的查找。所以为了更清晰地看到代码调用流程有必要可以输出其调用栈日志。以下是获取第三层调用栈方法。
```js
var myObj = {};
Error.captureStackTrace(myObj);
var str = String(myObj.stack);
var arr = str.split("\n");//以换行为分割
console.log(arr[3]);//第三层调用调用栈存入帮助排查问题,[0]是"Error"从1开始
```
以上只是简单显示，可以做一个Log工具方法进行封装。一般显示两层调用栈，同时输出的日志可配合IDE点击跳转到对应位置。