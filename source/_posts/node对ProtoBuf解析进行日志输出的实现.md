---
title: node对ProtoBuf解析进行日志输出的实现
date: 2020-01-11 09:36:01
tags: ProtoBuf  日志 高阶函数
---

## 背景

项目开发的痛点之一没有输入输入日志，之前的框架设计没做好客户端输入输出的日志处理，导致服务端如果不断点，就无法知道客户端发了什么和收到什么。难以排查问题，降低了开发效率。现在需要在当前框架修改使其有客户端调用参数与回传的日志。

## 方案选择

为了解决这个问题考虑了4个方案来实现：

1. 在框架调用的公共入口和出口进行数据打印。
如果项目是json解析数据的是可以实现的，但当前项目采用了protobuff解析数据，在上层调用无法知道当前的解析对象，所以无法解析出数据。

1. 在之前的接口实现功能的地方进行日志打印。
实现接口功能时可以解析数据打印，但几百个接口工作量大而且代码复用性差，这是个可行但不好的方案。

1. 修改proto解析对象的生成模板代码，对其加入日志输出。
可以修改第三方代码来进行日志输出，但这种方案只能做为穷途末路的情况下的最后手段。首先修改第三方代码工作量大，容易出现问题，其次需要自己维护一个第三方的版本分支，以后无法升级第三方，最后增加后续维护接手的工作量。

1. 通过高阶函数对proto 解析方法进行重新定义。
这个方案修改了proto解析对象的内部方法，可以对所有proto解析进行进行日志输出，这个方案从代码复用性，工作量，可维护性，对第三方库的侵入性来综合考量是当前最好的解决方案。

综合上考虑，选择了第4个解决方案。

## 方案实现

具体实现如参考如下,代码量不多，只有些比较少用到的功能方法,代码为Typescript代码：

```ts
import * as pro from '../../../../protoFiles/protoCompiled';
import { LogManager } from '../log/LogManager';

//需要忽略日志的proto
const ignoreProto = ['heartbeadResult'];
/**
 * 对proto 的decode 重新定义输出日志
 * @param fn 原函数
 * @param cName proto name
 */
function funDecodeLog(fn: Function, cName: string): Function {
    return function (...args: any[]) {
        let res = fn.apply(this, args);
        if (!cheakIsItemProto()) {
            try {
                LogManager.instance.oninfo(`${cName} decode  proto message --->:${JSON.stringify(res)}`);
            }
            catch (e) {
                LogManager.instance.ondebug(`${cName} decode  proto Err :${e.message}`);
            }
        }
        return res;
    };
}

/**
 * 对proto 的encode 重新定义输出日志
 * @param fn 原函数
 * @param cName proto name
 */
function funEncodeLog(fn: Function, cName: string): Function {
    return function (...args: any[]) {
        if (!cheakIsItemProto()) {
            try {
                LogManager.instance.oninfo(`${cName} encode proto message <---:${JSON.stringify(args[0])}`);
            }
            catch (e) {
                LogManager.instance.onwarn(`${cName} encode  proto Err :${e.message}`);
            }
        }
        let res = fn.apply(this, args);
        return res;
    };
}

/**
 * 检查是否是子proto
 */
function cheakIsItemProto() {
    let myObj: any = {};
    Error.captureStackTrace(myObj);
    let str = String(myObj.stack);
    let arr = str.split("\n");
    return arr[3].indexOf('protoCompiled.js') > 0;//protoCompiled.js prot文件是生成的，根据对应的文件名设置
}

/**
 * 重定义encode、decode 函数初始化入口
 */
export function initProtoFun() {
    for (let cName in pro) {
        if (ignoreProto.includes(cName)) {
            continue;
        }
        if (typeof pro[cName]['decode'] === 'function') {
            pro[cName]['decode'] = funDecodeLog(pro[cName]['decode'], cName);
        }
        if (typeof pro[cName]['encode'] === 'function') {
            pro[cName]['encode'] = funEncodeLog(pro[cName]['encode'], cName);
        }
    }
}
```

实际上日志输出应该在框架层面封装好，但目前框架已经成形，这种实现从全局来看也并不完美。