---
title: 如何在typescript引用 .json 文件
date: 2018-11-15 12:07:29
tags: typescript json
---
如果查找typescript引用 .json 文件大部分给出的答案是：

1、require 使用时是 a['xxx'],这种方式没有提示和代码检查，所以不使用。

2、typings.d.ts 添加下面这个定义:
```ts
declare module "*.json" {
const value:any;
export defaultv alue;

}
```
但我转换出的js代码是

HeroAttribute2.default.xxx ，然后报找不到default 属性。
解决方案
typings.d.ts定义改为 :
```ts
declare module "*.json";
```
引用方式改为 “* as”就可以带提示使用了。

**最后以上都是瞎整，TS 2.9  新增加的属性设置就可以了，难怪没什么资料。**

```json
"resolveJsonModule": true,
```

引用:
```ts
import Setting = require('../shared/publicShared/config/Setting.json');
```