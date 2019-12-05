---
title: pomelo对服务进程内存中变量读写
date: 2018-01-05 18:10:03
tags: pomelo 内存读写
---
# 背景
有时候我们需要对内存中的数据进行读写。比如以下场景:
* 游戏逻辑变得复杂出现bug，单纯由log无法看出内存中用户数据的变化，需要直接读取内存数据分析。
* 不影响线上玩家游戏体验，进行临时性的热更新。

# 实现
需要对内存数据读写具体方法参照：
[pomelo-cli-exec命令使用](https://github.com/NetEase/pomelo/wiki/pomelo-cli-exec%E5%91%BD%E4%BB%A4%E4%BD%BF%E7%94%A8)
可以同过app的set与get方法对游戏对象设置引用，比如我的用法：

```js
app.get("gameManager").playerEnterGame = function () {//这里app是服务进程的app对象，gameManager是在服务启动后设置下去的
return JSON.stringify(this.players);
};
result = app.get("gameManager").playerEnterGame();//这里的result 是对结果返回
```

以上是对gameManager对象的playerEnterGame方法进行更改。