---
title: 查看SQL执行计划(explain)
date: 2023-03-03 14:23:59
tags: 数据库
categories: SQL优化
---

## 查看SQL执行计划

**explain：**explain select * from xxx

当使用`explain sql`后会看到执行计划

![image-20230303142552512](F:\GitDown\hexo\source\_posts\查看SQL执行计划-explain.assets\image-20230303142552512.png)

| 字段          | 解释                                                         |
| ------------- | ------------------------------------------------------------ |
| id            | 每个被独立执行的操作标识，标识对象被操作的顺序，id值越大，先被执行，如果相同，执行顺序从上到下 |
| select_type   | 查询中每个select 字句的类型                                  |
| table         | 被操作的对象名称，通常是表名，但有其他格式                   |
| partitions    | 匹配的分区信息(对于非分区表值为NULL)                         |
| type          | 连接操作的类型                                               |
| possible_keys | 可能用到的索引                                               |
| key           | 优化器实际使用的索引(**最重要的列**) 从最好到最差的连接类型为`const`、`eq_reg`、`ref`、`range`、`index`和`ALL`。当出现`ALL`时表示当前SQL出现了“坏味道” |
| key_len       | 被优化器选定的索引键长度，单位是字节                         |
| ref           | 表示本行被操作对象的参照对象，无参照对象为NULL               |
| rows          | 查询执行所扫描的元组个数（对于innodb，此值为估计值）         |
| filtered      | 条件表上数据被过滤的元组个数百分比                           |
| extra         | 执行计划的重要补充信息，当此列出现`Using filesort` , `Using temporary` 字样时就要小心了，很可能SQL语句需要优化 |



### SQL优化小结

这里给大家总结一下优化SQL的套路

1. 查看执行计划 explain
2. 如果有告警信息，查看告警信息 show warnings;
3. 查看SQL涉及的表结构和索引信息
4. 根据执行计划，思考可能的优化点
5. 按照可能的优化点执行表结构变更、增加索引、SQL改写等操作
6. 查看优化后的执行时间和执行计划
7. 如果优化效果不明显，重复第四步操作

