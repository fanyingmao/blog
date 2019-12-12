---
title: js实现通过二分插入排序实现对用户排行榜性能的优化
date: 2017-12-08 15:58:00
tags: node 算法
---
## 背景
在开发中排序实现排序最简单的方法就是直接用sort方法进行排序。但当在后面用户量达到万级后就发现调用排行榜排序运算耗时太长导致连接超时。
## 优化思路：
1. 耗时运算放入另一个进程，同时隔一间隔更新排行榜。
1. 采用二分插入排序在原排序基础上进行部分排序。
显然直接用sort方法无法满足上述要求，需要自己手写排序算法了，下面是实现二分插入排序的插入的代码：
```js
rankManager.prototype.insertSort = function (targetArr, compareFn, player, isInit, gameType, type, isResource) {
    var delIndex;
    if (!isInit) {
        for (var i in targetArr) {
            if (player.id === targetArr[i].id) {
                delIndex = i;
            }
        }
        if (delIndex) {
            targetArr.splice(delIndex, 1);
        }
    }
    var left = 0; //最左边的数，从str[0]开始
    var right = targetArr.length; //最右边位，所要插入那个数的前一位
    var mid = 0;
    var index;
    while (left & lt; right) {
        if (compareFn(gameType, type, targetArr[left], player, isResource) & lt; 0) { index = left; break; } if (compareFn(gameType, type, targetArr[right - 1], player, isResource) & gt; 0) {
            index = right;
            break;
        }
        mid = parseInt((left + right) / 2);
        if (mid === left || mid === right) {
            index = mid;
            break;
        }
        if (compareFn(gameType, type, targetArr[mid], player, isResource) & lt; 0) { right = mid; } else { left = mid; }
    } targetArr.splice(index, 0, player); if (targetArr.length & gt; this.rankNum) {
        targetArr.splice(targetArr.length - 1, 1);
    }
};
```
这里需要维护targetArr这个在内存中的排行榜数组。服务器启动时对targetArr初始化排行，同时在每次用户数据更新都需要调用这个方法更新单个用户数据，这样就保证了降低运算量同时数据实时。

最后为什么不用大名鼎鼎的快速排序，先看下快速排序的思想：

1. 先从数列中取出一个数作为基准数
1. 分区过程，将比这个数大的数全放到它的右边，小于或等于它的数全放到它的左边
1. 再对左右区间重复第二步，直到各区间只有一个数

最后:用Redis的有序集合更容易实现不需要写排序算法。

2018.9.4 更新
**部分排序的方法还是不行的，对于排名掉出的情况会出现最后一名补充运算复杂度是n*n的情况。为了运算量的稳定考虑还是全排名比较合理。100万数据量最多对比次数也只是20次。**