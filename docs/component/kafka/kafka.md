# kafka
目标是结合flink做实时处理,告警，iot数据处理
## 基本概念
		leader
		follower
		消费者组
	kafka的性能测试
		磁盘性能
			sync;time -p bash -c "(dd if=/dev/zero of=test.dd bs=1M count=20000)"
			dd if=/dev/zero of=/data/test.dd bs=1M count=6553 oflag=direct
		做一下kafka性能测试
				./kafka-producer-perf-test.sh  --topic benchmark --num-records 10000000 --throughput -1 --record-size 100 --producer-props bootstrap.servers=192.168.1.30:9092,192.168.1.23:9092,192.168.1.25:9092 acks=1 --print-metrics
					5000000 records sent, 162037.787212 records/sec (15.45 MB/sec), 622.09 ms avg latency, 2159.00 ms max latency, 273 ms 50th, 1951 ms 95th, 2071 ms 99th, 2097 ms 99.9th.
				./kafka-consumer-perf-test.sh  --topic benchmark --threads 1 --messages 5000000 --broker-list 192.168.1.30:9092,192.168.1.23:9092,192.168.1.25:9092
					start.time, end.time, data.consumed.in.MB, MB.sec, data.consumed.in.nMsg, nMsg.sec, rebalance.time.ms, fetch.time.ms, fetch.MB.sec, fetch.nMsg.sec
					2023-07-12 15:17:10:576, 2023-07-12 15:17:23:636, 476.8372, 36.5113, 5000000, 382848.3920, 1574, 11486, 41.5146, 435312.5544
## 部署

## 问题
    性能高的原因
		0拷贝
		随机I/O与顺序io的区别
	一次消费的保障
	结合flink的使用
	丢数和补数的方案
	如何配置可以不丢数
		kafka是异步发送
			producer send callback ack=all
			unclean.leader.election.enable = false
			replication >= 3
			min.insync.replicas > 1
			replication > min.insync.replicas
			enable.auto.commit = false
	kafka stream
	系统参数
		ulimit -n 1000000
		xfs和ext4
			xfs 对大文件有有时
		batch.size
## 实践
		export JAVA_HOME=/opt/jdk-11.0.18/
	问题
		kafka的事务,多少条提交一次，还是异步吗
## 优化
		禁用atime更新mount -o noatime
		文件系统xfs
		ulimit -n和vm.max_map_count
		将你的JVM堆大小设置成6～8GB。
		特别是关注Full GC之后堆上存活对象的总大小，然后把堆大小设置为该值的1.5～2倍。如果你发现Full GC没有被执行过，手动运行jmap -histo:live < pid >就能人为触发Full GC