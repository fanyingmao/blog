---
title: node在Typescrip中实现热更新的思路
date: 2021-04-20 19:20:46
tags: 热更新 Typescrip
---

# 背景

其实 node 热更基本原理都是清除 require 的缓存后，在重新 require。之前也见过别人在 Typescrip 下的热更实现，但是有个缺点：重新 require 后缺失了引用对象的代码提示，造成了不好的开发体验。

# 实现记录

下面是我的实现

```ts
import * as fs from "fs";
import * as Card_role from "../data/Card_role.json";
import { LogUtils } from "./LogUtils";
import path = require("path");

const mLogUtils = new LogUtils(__filename);
/**
 * 这里会将data目录下所有的文件绑定到HotReload上，同时会修改后热更新，但为了代码编写有提示所有还需要import文件，同时typeof到成员变量上
 */
export class HotReload {
  // 为了开发效率这里就用一样的名字，还有这是为了代码中有提示才typeof
  static Card_role: typeof Card_role;

  public static init() {
    HotReload.watchCleanCache("../data/", (fileName: string, moudle: any) => {
      const name = fileName.split(".")[0];
      HotReload[name] = moudle;
    });
  }

  private static cleanCache(modulePath: string) {
    delete require.cache[modulePath];
  }

  private static watchCleanCache(dir: string, cb: Function) {
    const dirPath = path.join(__dirname, dir);
    fs.readdir(dirPath, (err, files) => {
      files.forEach((item) => {
        const moudle = require(dirPath + item);
        cb(item, moudle);
        HotReload.watchFile(dirPath, item, cb);
      });
    });
  }

  private static watchFile(dirPath: string, fileName: string, cb: Function) {
    let filePath = dirPath + fileName;
    fs.watchFile(filePath, function () {
      mLogUtils.info("HotReload :" + fileName);
      HotReload.cleanCache(require.resolve(filePath));
      const moudle = require(filePath);
      cb(fileName, moudle);
    });
  }
}
```

这里顺便说下为什么用 watchFile 而不是 watch 方法，因为时间测试中修改文件一次出现 watch 方法回调了 3 次，虽然 3 次没什么影响，但是有节约强迫症改为了 watchFile，watchFile 有个小问题估计是轮询机制导致的就是有点延迟，但是可以接受。
