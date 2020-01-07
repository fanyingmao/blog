---
title: pomelo通过高阶函数实现非侵入式的自定义异常捕获
date: 2018-09-16 16:57:47
tags: 高阶函数 异常捕获 pomelo
---
## 背景
使用的是pomelo框架开发，项目中需要对接口自定义异常捕获处理。
## 方案分析 
方案有三个：
1、首先想到的是在函数上层调用try catch。但是上层调用是第三方框架中，需要侵入node module改写第三方代码。
2、使用装饰器改写函数，但装饰器现在还只是es7提案，现在node还不支持这个语法。
3、使用高阶函数在构造函数中对接口方法重新定义函数。

## 实现
综上，我使用了第三种方案，例子如下：

```js
class Test {
  constructor(){
    let funs = Object.getOwnPropertyNames(Test.prototype);
    function fluent(fn){//高阶函数
      return function(...args){
        try {
          fn.apply(this,args);
        }
        catch (e) {
          console.log(e);
        }
        return this;
      };
    }
    funs.forEach(item =&gt; {
      if(item !== 'constructor'){
        this[item] = fluent( this[item]);//重新定义函数
      }
    });
  }

  log1(){
    throw 'log1';
  }

  log2(){
    console.log('log2')
  }
}
let test = new Test();
test.log1();
test.log2();
```

输出结果：

```sh
log1
log2
``` 

可以看到成功对类中的函数捕获异常。