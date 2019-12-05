---
title: JS实现对emoji表情替换
date: 2017-12-18 12:23:03
tags: js emoji
---
由于客户端对微信昵称包含emoji表情json解析出错要求服务端去除，以下是去除方法：
```js
var a =",,🐶🐅";

var ranges = [
    '\ud83c[\udf00-\udfff]',
    '\ud83d[\udc00-\ude4f]',
    '\ud83d[\ude80-\udeff]'
];
a = a.replace(new RegExp(ranges.join('|'), 'g'), '');
console.log(a);
```