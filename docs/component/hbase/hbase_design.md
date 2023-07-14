# 记一次hbase表设计

# 前言
这次的内容比较简单。之前hbasee接触的不多，难得有一次使用hbase的机会，趁机学习了一点。总之是一些比较基本的设计，记录总结一下。
# 业务背景
之前的文章基本是基于elasticsearch（以下简称es） 展开的。我们在项目中主要使用es对用户的一组属性做过滤，筛选出具备某些行为特征或者说标签的用户。这组筛选条件我们将保存下来，形成模型，以便复用。对于模型和用户的对应关系，也就是说用户是否存在于某个模型筛选结果之中，我们决定存入hbase中，以支持上层应用的查询。
其实目前主要满足两种业务场景
1. 给出用户id和模型id，判断是否用户是否在模型之中
2. 给出用户id和模型id，及业务属性，更新对应的业务属性

# 关于hbase rowkey的疑问
在之前就了解到hbase主要基于rowkey来做查询，那么是如何根据rowkey快速查询的，产生了下面几个疑问。
## 1 rowkey如何定位到整行数据的？
1. 根据meta表查到对应的regionserver
2. regionserver 根据列族构建StoreScanner，有多少个列族就有多少StoreScanner（这应该就是不提倡建多个列族的原因）
3. StoreScanner 对每个hfile创建一个StoreFileScanner
4. 根据rowkey的范围，时间的范围过滤去掉肯定不存在的hfile的StoreFileScanner
5. StoreFileScanner 读取hfile的块索引树，通过二分查找，定位到具体的rowkey对应的data block

## 2 数据是如何组织存储的？
hbase使用LSM树来存储数据，这个数据结构牺牲了部分读性能，但提高了写性能。主要是通过将内存中的数据憋到一定阈值再向硬盘flush，提高了顺序写，相对于随机写提高了性能

# rowkey的设计原则

## 长度
肯定是越短越好，越长就意味着占用越多的存储空间

##唯一
没什么好说的，主键，唯一性

##均匀
主要是为了在写入的过程中，可以均匀分布到不同region,相同前缀的rowkey会造成regionserver的热写

# 实践
1. rowkey = 用户id+模型id （用户id前缀是均匀的数值，模型id是等长的uuid，通过两者的组合，可以快速定位记录，满足上述的业务需求）
2. 开启bloom filter（bloom 过滤器可以快速跳过肯定不在的hfile，以提高读性能，缺点是需要额外的存储）
3. 预分区（避免region写满分裂而造成的性能消耗，提前分配region；由于我们的rowkey前缀是数值，正好利用这个特性，均匀散列到不同的region）
4. 综上建表语句为
create ‘test’,{NAME => ‘business’，BLOOMFILTER =>'ROW }, { NUMREGIONS => 10, SPLITALGO => ‘DecimalStringSplit’ }


# 参考资料
[https://www.cnblogs.com/qingyunzong/p/8696962.html](https://www.cnblogs.com/qingyunzong/p/8696962.html)
[http://hbasefly.com/2016/12/21/hbase-getorscan/](http://hbasefly.com/2016/12/21/hbase-getorscan/)
[http://hbasefly.com/2016/04/03/hbase_hfile_index/](http://hbasefly.com/2016/04/03/hbase_hfile_index/)
