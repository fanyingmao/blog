---
title: 为js编写d.ts声明文件，js与ts导出兼容
date: 2019-06-03 11:53:39
tags:
  - ts
---

[ym-mongodb-sql](https://github.com/fanyingmao/ym-mongodb-sql)添加 ts 支持时发现：
ts 引用类的方法无法使用，查看变异后的代码发现，ts 引用类转为 js 中会加 default，但 js 引用类却没有，为了兼容二者。js 导出模块需要添加 default 导出。
代码处理：

```js
module.exports = MongodbSql;
module.exports.default = MongodbSql; //兼容ts写法
```
