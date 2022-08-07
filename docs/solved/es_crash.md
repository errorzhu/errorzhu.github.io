# 记一次 Elasticsearch 的查询崩溃分析 

# 前言
项目中使用es聚合查询时，发生集群崩溃的现象。经查阅资料，整理一下如何系统定位es异常的使用

# 背景
虚拟机三台（4core，8g内存，es堆分配了4g）
数据量（5000w条数据，100g左右）

# 现象
应用在执行一些自定义查询时timeout了，通过kibana监测到cpu和heap都飙满，导致一些节点下线，无法响应请求。

# 分析流程
ps（公司集群无法截图；下面的截图，均取自自己的阿里云测试服务器，只为说明情况，帮助理解）
## 1 系统资源查看
### top
![](https://images.xiaozhuanlan.com/photo/2020/f89a3b9f28794cf26c36c01e6e5ab04b.png)
通过top命令，我们可以粗略的看下服务器的资源使用情况。看看是es相关进程耗费的资源比较多，还是因为其他服务抢占了es的资源。图中我们可以看出es的进程消耗了很多的内存资源。

### sar
![](https://images.xiaozhuanlan.com/photo/2020/72c342046550724fe9fdb73eed74a1eb.png)

通过sar 命令，可以查看硬盘的读写情况，es统计需要加载索引到内存中计算，如果读写很高，说明查询没有命中内存，需要回盘去加载数据，可能因此导致响应延迟

## 2 查看日志
es的日志中记录gc及异常的发生，仔细查看日志，我们可以发现端倪。
![](https://images.xiaozhuanlan.com/photo/2020/0ef43db2b98243a3a7ab78350d048c29.png)
这里抛出了oom的异常，以及打印了gc花费的时间。再仔细查看相关的栈信息，我们发现是在InternalHistogram.addEmptyBucket时抛出的。我们可以初步判定由于InternalHistogram类型的查询，导致oom了。

## 3 进程信息
es是一个java程序，这里我们可以很方便的通过jdk 提供的一些工具，来分析进程级别的信息
### jps
![](https://images.xiaozhuanlan.com/photo/2020/6793c793cf288777942aa2bb30e1877c.png)
通过jps快速定位到java 进程的pid

### jstat
通过jstat 可以查看jvm的使用情况，由于上述日志中发现了gc相关的信息，我们主要关注一下gc的细节
![](https://images.xiaozhuanlan.com/photo/2020/cf5a45dcacc70aff874804421534451d.png)
这里重点关注一下fgc(full gc)，fgc会引起stw（stop the world），会暂停所有线程。如果一次fgc花费了很多时间，要考虑是否创建了太多的大对象，或者堆内存太小了。

### jmap
jmap 可以dump 堆信息
![](https://images.xiaozhuanlan.com/photo/2020/c959aaf735c96af18a0c5a9775631175.png)
jmap -heap 来查看一下堆的使用情况

![](https://images.xiaozhuanlan.com/photo/2020/99f43c90c10c62c8c120270d88632ad0.png)
jmap -histo 查看一下堆中具体哪些对象占用的较多。
这里我们发现InterHistogramBucket，占用的最多，基本验证了我们通过分析日志得出的结论。histogram 查询创建了太多的bucket，导致oom了。

## 结合业务分析
那这个histogram 是啥呢？最后我们再结合业务分析一下。我们通过查询应用的日志，知道了，用户配置了一个根据金额这个字段，按5w一个区间进行统计的一个查询。具体实现对应的就是es的histogram查询（直方图统计）。由于用户没有配置查询的上限，而金额这个字段中存储的最大值将有1000亿这么大，从0到1000亿,每5w一个区间，将会产生上千万个bucket,这也就是为什么我们jmap查看到有很多bucket对象的原因了。由于需要聚合，这些bucket无法回收，最终导致了oom。

## 通过es的api再分分析一下
### _nodes/hot_threads
可以查看cpu占用最高的线程
![](https://images.xiaozhuanlan.com/photo/2020/db01ed253011c60da2ebc3b4de4cbbe3.png)
我们可以查看到线程的栈信息,据此我们也可以结合业务来分析异常的原因

# 参考资料
[https://www.ucloud.cn/yun/34298.html](https://www.ucloud.cn/yun/34298.html)
[https://blog.csdn.net/zhaozheng7758/article/details/8623549](https://blog.csdn.net/zhaozheng7758/article/details/8623549)