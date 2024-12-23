# spark 调优

## 内存和cpu
内存和cpu的比例，单个executor 设置的vcores 数量太大，单个executor所处理的任务和vcore对应，同样需要读取对应数量的分区，内存设置的少，就会oom

## 并行度
每个task 处理一个分区的数据，同时能执行多少个task，意味着rdd多少个分区在同时被处理



## case

1. isEmpty 和 count
2. spark ui