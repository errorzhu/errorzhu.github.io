# presto 优化

 # 操作系统
## 关闭swap 分区
       root用户执行 swapoff -a 
       注释掉/etc/fstab swap分区行的挂载
# presto
## 内存
		-xmx 80%总内存
		xms xmx 同时配，避免内存分配分配带来的时间
		query.max-memory 最大总内存
		query.max-memory-per-node  xmx 的 20%
## 并发
		query.max-concurrent-queries 20
		task.max-worker-threads Node CPUs * 4
		
## hive
		增加presto访问hive metastore 超时时间
			修改 /opt/apps/presto/etc/catalog/hive.properties
			增加参数hive.metastore-timeout=30s
		配置presto访问hive metastore 高可用
			修改 /opt/apps/presto/etc/catalog/hive.properties
			修改配置hive.metastore.uri=thrift://host1:9083,thrift://host2:9083,thrift://host3:9083
		允许presto删除表
			修改 /opt/apps/presto/etc/catalog/hive.properties
			hive.allow-drop-table=true