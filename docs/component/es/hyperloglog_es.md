# hyperloglog 在 elasticsearch 中 的应用

# 概述
hyperloglog是用来近似统计不重复数据次数（cardinality count）的一种算法。主要应用于在一定误差允许范围内，进行去重计数。比如，近似统计网站日活用户数（非计算精确指标，只是为了监控流量趋势，提供背压标准）
# 为什么是hyperloglog
去重计数可以通过其他方案来实现，比如hashmap，b树，bitmap，但上述方法都有一个缺点，内存消耗会随数据规模线性增长，以bitmap为例，如果要统计1亿个数据的基数值，大约需要内存： 100000000/8/1024/1024 ≈ 12M
而redis中实现的HyperLogLog，只需要12K内存，在标准误差0.81%的前提下，能够统计2^64 个数据。
# 原理
## 伯努利实验
连续抛一枚硬币，直到出现正面，我们称为完成了一次伯努利实验。将该次实验中出现正面的次数记为k。
重复若干次实验，所有次中最大的k，记为kmax。
根据最大似然估计，进行的次数n = 2^ kmax，即通过分布模型推测出模型参数，这里次数越少，误差就越大。
为了降低误差，我们进行m组实验，估算出每组的n值nm，总的次数 = nm*m
最后得到公式
n = 修正因子 *组数m*2^(m组kmax的调和平均数)

## hyperloglog与伯努利实验的关系
将数据进行hash运算得到bit串，每一个bit只有0,1两种情况，和硬币只有正反是同样的。所以我们取得比特串从低位到高位的方向上，第一次出现1的位数，这个过程可以类比为取到伯努利实验中第一次正面的次数k。
伯努利实验是根据kmax，估算出实验样本次数n
hyperloglog是根据kmax，估算出实验样本数据量n
**ps:hash算法使相同的数据的bit串是一样的，这就意味估算的实验样本数据是去重的**

## hyperloglog的结构
hyperloglog是长度为L的bitmap，分成m个桶，桶长为p ，即L = m * p
## hyperloglog的步骤
1. hash（id） 为比特串 ，如 hash(10)= 1010
2. 分桶，上面我们说到伯努利实验为减少误差，采取了分组，只进行一组实验，偶然性很大，但进行100组，误差就小很多了。hyperloglog也是同样的思想，将数据划入不同的桶中，计算出每桶的值，将每桶的值求平均，再乘以总桶数，即得到数据总量。
如1010去前两位，10进制为2，落入2号桶中。
3. 计算k值，1010，去掉前两位桶号，剩下的10，从低位到高位，第二位为1，将位数2转为二进制10，落入桶内
4 重复上述步骤，如果计算的值大于桶内现有值，则更新桶值kmax
5 通过公式 n = 修正因子 *组数m*2^(m组kmax的调和平均数)，估算出去重数据量n
**ps:修正因子是为了将结果修正成无偏估计**，数学分析过程参考[http://blog.codinglabs.org/articles/algorithms-for-cardinality-estimation-part-iv.html](http://blog.codinglabs.org/articles/algorithms-for-cardinality-estimation-part-iv.html)

# 应用
elasticsearch的聚合方法中，提到了其基数聚合（cardinality aggregation）就是采用的hyperloglog算法

# 参考资料
[https://stackoverflow.com/questions/12327004/how-does-the-hyperloglog-algorithm-work](https://stackoverflow.com/questions/12327004/how-does-the-hyperloglog-algorithm-work)
[http://www.rainybowe.com/blog/2017/07/13/%E7%A5%9E%E5%A5%87%E7%9A%84HyperLogLog%E7%AE%97%E6%B3%95/index.html](http://www.rainybowe.com/blog/2017/07/13/%E7%A5%9E%E5%A5%87%E7%9A%84HyperLogLog%E7%AE%97%E6%B3%95/index.html)
[https://juejin.im/post/5c7900bf518825407c7eafd0](https://juejin.im/post/5c7900bf518825407c7eafd0)
