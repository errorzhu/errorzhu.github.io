# mysql并发写入cpu过高

## 背景

我们有个程序并发（30个进程用 ）读取mysql数据，通过传入参数读取不同时间范围的数据，发现mysql cpu在读取二天的数据时，比读取一天时 高了非常多

读取一天

![](../images\wiki\mysql_cpu1.png)

读取两天

![](../images\wiki\mysql_cpu2.png)

## 分析与解释

分别查看执行计划

```
explain select * from c_ac_data_powerby15min where day >= 20240924 and cons_no = 'aaaa' and day <= 20240924
```

![](../images\wiki\msql_explain_2.png)

```
explain select * from c_ac_data_powerby15min where day >= 20240924 and cons_no = 'aaaa' and day <= 20240926
```

![](../images\wiki\mysql_explain_1.png)

c_ac_data_powerby15min 的主键是 (day, cons_no, id, data_date)，并且使用了 BTREE 索引。

​​主键索引的顺序​​是 day、cons_no、id、data_date。
查询时，MySQL 会优先使用主键索引的最左前缀匹配原则。

1. 查询一天
   
   主键索引的最左前缀是 day，因此 MySQL 可以使用主键索引来快速定位 day = 20240924 的数据。
   由于 cons_no 是主键索引的第二列，MySQL 在定位到 day = 20240924 后，需要进一步过滤 cons_no = 'aaaa' 的数据
   
   ​执行计划​​：
   type = range：表示 MySQL 使用了索引的范围扫描。
   rows = 1：表示 MySQL 估计需要扫描 1 行数据。
   Using where：表示 MySQL 在索引扫描后，还需要通过 WHERE 条件进一步过滤数据。

2. 查询多天
   
   主键索引的最左前缀是 day，因此 MySQL 可以使用主键索引来扫描 day 在 [20240924, 20240926] 范围内的数据。
   由于 cons_no = '0006728436' 是主键索引的第二列，MySQL 在扫描到 day 范围内的数据后，需要进一步过滤 cons_no = 'aaaa' 的数据。
   ​​执行计划​​：
   type = range：表示 MySQL 使用了索引的范围扫描。
   rows = 711227：表示 MySQL 估计需要扫描 711227 行数据。
   Using where：表示 MySQL 在索引扫描后，还需要通过 WHERE 条件进一步过滤数据。

## 总结

1. mysql 查询需要根据索引顺序，构建适合的查询条件.

2. 通过查看执行计划，分析查询语句的执行情况
