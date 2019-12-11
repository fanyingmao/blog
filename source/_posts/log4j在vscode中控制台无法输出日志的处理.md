---
title: log4j在vscode中控制台无法输出日志的处理
date: 2019-12-11 16:59:10
tags: log4j vscode
---
# 问题

项目中使用log4j来做日志输出管理，配置如下：
```ts
        this.log4js = require('log4js');
        this.log4js.configure({
            appenders: {
                out: { type: 'stdout' },//设置是否在控制台打印日志
                ruleConsole: { type: 'console' },
                ruleFile: {
                    type: 'dateFile',
                    filename: LogManager.instance.log_filename,
                    pattern: 'yyyy-MM-dd.log',
                    maxLogSize: 1024 * 1024 * 1024,
                    numBackups: 3,
                    alwaysIncludePattern: true
                }
            },
            categories: {
                default: {
                    appenders: ['out', 'ruleFile'],
                    level: "info"
                }
            }
        });
```
然后出现了用webstrom可以正常输入日志，但vscode无法输出日志的情况。

# 解决方案

在.vscode/launch.json的启动配置加上   "outputCapture": "std", 就可以了。
```json
   {
            "type": "node",
            "request": "launch",
            "name": "api_server",
            "program": "${workspaceFolder}/bin/www",
            "preLaunchTask": "tsc: build - tsconfig.json",
            "outputCapture": "std",
            "outFiles": [
                "${workspaceFolder}/bin/**/*.js"
            ]
    }
```
[参考的issues](https://github.com/microsoft/vscode/issues/19750)