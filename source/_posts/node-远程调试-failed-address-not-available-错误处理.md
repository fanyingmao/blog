---
title: 'node 远程调试 failed: address not available 错误处理'
date: 2018-06-06 11:31:32
tags: 
- node 
- 调试
---

项目远程调试是十分必要的，可以对部署在外网的环境中断点查找问题,而不用在本机上测试问题，
但调试启动：

```sh
node --inspect=47.52.92.163:8025 testDebug.js
```

报错

```sh
Starting inspector on 47.52.92.163:8025 failed: address not available
```

测试发现虽然一般都用服务器的公网ip，但这里调试需要用云服务器的<strong>内网ip（私有ip）</strong>——linux下ifconfig命令显示的ip即内网ip，才可以成功运行的.修改后可以成功调试,用0.0.0.0这个ip就可以了。

```sh
node --inspect=0.0.0.0:8025 testDebug.js
```

测试代码：
```js
//testDebug.js
function test() {
console.log("test === " + Date.now());
}
setInterval(test.bind(this),1000);
```