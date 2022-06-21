# 记一次flink导数据异常

# 背景

使用flink 1.11.1 做数据迁移时，读取mysql数据导入clickhouse，发现卡住，查询到相关jira [https://issues.apache.org/jira/browse/FLINK-19435](https://issues.apache.org/jira/browse/FLINK-19435)，特此记录这个问题。

# 原理

1.  Class.forName时，会对类进行初始化，会执行加载类的静态代码块
1. 在mysql jdbc driver和clickhouse jdbc driver初始化过程中，会调用driverManager的静态代码块。
1. 类初始化时在jvm上会加锁
1. driverManager 会加载所有的驱动
1. 两个线程持锁互相竞争

![](../images\wiki\flink.png)

#  参考资料
[https://medium.com/@priyaaggarwal24/avoiding-deadlock-when-using-multiple-jdbc-drivers-in-an-application-ce0b9464ecdf](https://medium.com/@priyaaggarwal24/avoiding-deadlock-when-using-multiple-jdbc-drivers-in-an-application-ce0b9464ecdf)
[http://lovestblog.cn/blog/2014/07/08/jdk-sql-deadlock/?spm=a2c6h.12873639.0.0.4bae31fdKbeSWf](http://lovestblog.cn/blog/2014/07/08/jdk-sql-deadlock/?spm=a2c6h.12873639.0.0.4bae31fdKbeSWf)
[https://developer.aliyun.com/article/115107](https://developer.aliyun.com/article/115107)