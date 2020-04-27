---
title: mysql在加了索引两个字段在or的情况下全表扫描的问题处理
date: 2020-04-26 10:48:07
tags: mysql 索引
---

## 背景

根据设备号imei和oaid来与渠道对接引流用户或激活，很容易想到imei与oaid加上索引来查找用户。然后很容易想到用这样的sql查询

```sql
SELECT count(1) FROM t_player_data WHERE  imei=<imei> OR oaid=<oaid>;
```
但是在阿里云的mysql后台中看到这个语句在一些情况下会是慢查询(最慢的在正式服上大约要12秒)，需要查找下原因。

## 分析

表字结构（已过滤其他非关键字段）,可以看到iemi，oaid都加上索引了，iemi是utf8mb4编码这个是历史遗留问题但和现在性能问题关系不大:
```sql
CREATE TABLE `t_player_data` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '用户真实id',
  `imei` varchar(255) CHARACTER SET utf8mb4 DEFAULT '' COMMENT '设备码  --网站用',
  `oaid` varchar(255) NOT NULL DEFAULT '' COMMENT 'android10无法获取imei用oaid代替',
  PRIMARY KEY (`id`) USING BTREE,
  KEY `idx_imei` (`imei`(191)) USING BTREE,
  KEY `idx_oaid` (`oaid`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1010638 DEFAULT CHARSET=utf8 ROW_FORMAT=COMPACT;
```
需要分析sql性能一般是通过执行计划来看的，分析结果中比较关键的是type和row，这里对EXPLAIN的type做下说明：

链接类型 | 	说明  
-|-
system | 表只有一行，MyISAM引擎。
const | 常量连接，表最多只有一行匹配，通用用于主键或者唯一索引比较时
eq_ref | 每次与之前的表合并行都只在该表读取一行，这是除了system，const之外最好的一种，特点是使用=，而且索引的所有部分都参与join且索引是主键或非空唯一键的索引
ref | 如果每次只匹配少数行，那就是比较好的一种，使用=或<=>，可以是左覆盖索引或非主键或非唯一键
fulltext | 全文搜索
ref_or_null | 与ref类似，但包括NULL
index_merge | 表示出现了索引合并优化(包括交集，并集以及交集之间的并集)，但不包括跨表和全文索引。这个比较复杂，目前的理解是合并单表的范围索引扫描（如果成本估算比普通的range要更优的话）
unique_subquery | 在in子查询中，就是value in (select…)把形如select unique_key_column的子查询替换。PS：所以不一定in子句中使用子查询就是低效的
index_subquery | 同上，但把形如”select non_unique_key_column“的子查询替换
range | 常数值的范围
index | 索引树扫描。a.当查询是索引覆盖的，即所有数据均可从索引树获取的时候（Extra中有Using Index）；b.以索引顺序从索引中查找数据行的全表扫描（无 Using Index）；c.如果Extra中Using Index与Using Where同时出现的话，则是利用索引查找键值的意思；d.如单独出现，则是用读索引来代替读行，但不用于查找
all | 	全表扫描(full table scan).

上表至上而下性能越来越慢。


在正式服测试：

```sql
EXPLAIN SELECT *shao FROM t_player_data WHERE imei='0' OR oaid='1'; --type ALL 
EXPLAIN SELECT * FROM t_player_data WHERE imei='1' OR oaid='0'; --type ALL
EXPLAIN SELECT * FROM t_player_data WHERE imei='0' OR oaid='1'; --type ALL
EXPLAIN SELECT * FROM t_player_data WHERE imei='1' OR oaid='1'; --type index_merge
```

可以看到执行计划在等于0的时候,就不走索引进行全表扫描了，同时row的条数也等于表条数了。

但在测试服测试中这4条语句的type就都是 index_merge，怀疑是imei和oaid为0的条数差异导致同样的语句链接类型的不同。
通过不断修改条数发现，row在1000条左右时链接类型会由index_merge 变为ALL 从而效率大幅降低。

## 解决

iemi与oaid在客户端获取不到的情况下，客户端都会给服务端传0，导致数据库中由大量这样的数据，然后在激活时又传0的设备码来查询就会出现全表扫描的慢查询。方案就是对传0的特殊处理，不进入查询。
