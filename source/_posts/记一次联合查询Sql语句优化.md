---
title: 记一次联合查询Sql语句优化
date: 2019-07-02 09:58:39
tags:
  - sql
  - 优化
---

## 优化前

```sql
SELECT a.id, a.user_name, a.user_code, a.head_url, ifnull(b.red_packet_received_amount, 0) AS red_packet_received_amount,
    a.create_time
    FROM (select * from t_player_data WHERE openinstall = '24735' )a
    LEFT JOIN (SELECT user_id, SUM(amount) AS red_packet_received_amount
    FROM t_red_packet_order
    WHERE is_received = 1
    GROUP BY user_id
) b ON b.user_id = a.id;
```

耗时间 4s

## 优化后

```sql
SELECT a.id, a.user_name, a.user_code, a.head_url,SUM(IF(b.is_received = 1, b.amount,0)) AS red_packet_received_amount,
        a.create_time
        FROM (select * from t_player_data  WHERE openinstall = '24735' )a
            LEFT JOIN t_red_packet_order b ON b.user_id = a.id  GROUP BY id;
```

耗时 0.11s

## 分析

可以看到虽然查出的结果一样的但时间差距不是一个数量级的.原因就是第一条 sql 中

```sql
SELECT user_id, SUM(amount) AS red_packet_received_amount FROM t_red_packet_order WHERE is_received = 1 GROUP BY user_id
```

语句会产生很大的临时表,把预计单独执行就耗时 3s 多,所以通过”select \* from t_player_data WHERE openinstall = ‘24735’”产生的 id 去筛选结果省去了临时表的生成从而提高了效率.
