# elasticsearch性能测试与瓶颈分析

# 前言
最近项目中要用到elasticsearch（以下简称es），遂对其进行一些基于业务的测试工作。本篇文章不会赘述es相关的基本概念，亦不会大段解释es底层的数据结构和原理，仅仅从使用者的角度，初步体验一下es及对遇到的问题和思考。对于理论原理方面的内容，会后续总结整理在其他的文章中。

# 背景
##1. 硬件
主机3台
cpu:  4 core
memory: 8g
机械硬盘sas: 100g

## 2. 软件
jdk: 1.8.132
elasticsearch:  6.8.3

## 3. 索引结构
![](https://images.xiaozhuanlan.com/photo/2019/97d1832d807aa53344ac79e6fd774190.png)
说明：35个字段，一半以上是double类型，剩余的为keyword类型

## 4. 索引数据量
5000w条数据，生成的索引文件占据46g硬盘

## 5. 查询DSL
![](https://images.xiaozhuanlan.com/photo/2019/e9fee15e861ad41b5a4993f448b46015.png)
说明: 业务上做对double类型字段的范围查询 + 对keyword 类型的码值字段的精确查询。

# 测试说明及结果
使用jmeter进行测试，查询语句中的具体字段和值，通过jmeter的csv config读取预先生成的1000组变量，基本上每条查询请求的具体参数都不一样。查询结果中没有计算es的query cache 和request cache引起的误差。


![](https://images.xiaozhuanlan.com/photo/2019/0a7a2a79bd64c35263d713b2a222daa2.png)

![](https://images.xiaozhuanlan.com/photo/2019/8fbd9debbba7070eecdabf83060602cb.png)

![](https://images.xiaozhuanlan.com/photo/2019/4eea2e7020ea7c1def447d8cc1c844f0.png)

说明: 
1.  测试中主要对并发数，查询线程数，查询条件，及副本数进行调整测试。
2. es中在elasticsearch.yml中可以设置不同工作线程，测试中分别设置了不同的值进行测试。default为es默认配置，根据（int((# of available_processors * 3) / 2) + 1）公式计算得出。具体线程可以通过 GET /_cat/thread_pool 请求查看。


# 疑问与分析

## 1. 3term查询比3term+3range组合查询要慢

- 组合查询需要对单个条件过滤得到的结果进行合并，但elasticsearch的查询并不是按照dsl里条件写的顺序依次执行的，对于每个过滤条件会进行调用cost函数计算损失值，然后在计算代价小的结果集上进行迭代。我这里因为term的条件都是两三个值的枚举，单个term只能滤出总数据的二，三分之一；而range查询，可能得到的结果只有总数据的100分之一。以range的结果集进行迭代的原点，大大降低了合并的速度。

## 2. 增加一副本，查询速度下降很多

- 测试以前，认为增加副本数，会提高查询速度的。因为目前情况是3台机器，每台2分片，固定的数据只能将请求转发到特定的机器上才能获取，增加副本后，理论上减少了请求转发的次数，应该会变快才对。
- 首先通过 sar -n DEV 查看到网络流量有一定减少，证明转发请求的猜测是正确的。
通过top命令查看，发现cpu的利用率没有被压满，考虑是硬盘或内存上出了瓶颈。
通过iostat -d ,发现磁盘读大幅增加。
- 这里进行如下推测，elasticsearch底层使用Lucene，Lucene非常依赖文件系统缓存。每台机器剩余6g内存(xms,xmx 都给了es 2g)，基本在执行查询后buffer都被读满（可以通过free -g确认），且无法清除（通过echo 3 > /proc/sys/vm/drop_caches），因为默认的elasticsearch通过mmap，做文件内存映射，这部分被读到的文件，将保持在buffer中。在0副本时，分片的数据分配是固定的，因查询中标的文件被缓存到buffer中，这使得命中同样数据的后续查询，不再需要从硬盘读数据。而增加了副本后，新的请求可能不会被发送到之前请求的同一个分片上。假设请求到同样数据的不同分片概率是一样的，这就使得后续查询需要从硬盘重新读数据的概率增大了一倍，所以发现磁盘读相对0副本时有了巨大的增加，导致查询变慢了很多。

# 总结

1. 适当调大search线程数，并发情况下，表现略好于默认配置。
2. 尽量使某一条件筛选出的结果集少，这样在多个条件下进行docid合并时，以短链为基准迭代，速度较快。
3. 堆外内存不足以cache巨大的Lucene段时，查询从硬盘读取数据，效果很差。

# 参考资料
1. [https://zhuanlan.zhihu.com/p/47951652](https://zhuanlan.zhihu.com/p/47951652)
2. [https://www.elastic.co/blog/elasticsearch-query-execution-order](https://www.elastic.co/blog/elasticsearch-query-execution-order)
3. [https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-threadpool.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-threadpool.html)




