# activemq的使用和调优

## 基准性能

```
/data/mq-perf/maven/bin/mvn activemq-perf:producer  -Dfactory.brokerURL=tcp://ip:61616 -Dfactory.userName=admin  -Dfactory.password=密码  -Dproducer.destName=benchmark -DsysTest.numClients=1 -Dproducer.messageSize=120 -Dproducer.sendCount=1000000  -Dproducer.createNewMsg=true -Dproducer.sendType=count -Dfactory.asyncSession=false  -Dfactory.useAsyncSend=true

```

```
/data/mq-perf/maven/bin/mvn activemq-perf:consumer  -Dfactory.brokerURL=tcp://ip:61616?jms.prefetchPolicy.all=50000 -Dfactory.userName=admin  -Dfactory.password=密码 -Dconsumer.destName=benchmark -DsysTest.numClients=1 -Dconsumer.recvCount=1000000  -Dconsumer.recvType=count -Dfactory.asyncSession=false -Dfactory.prefetchTopic=50000
```

```
10个生产-----1个消费 
1w/s    -----5000/s

1个生产-----1个消费
1.8w/s -------8000/s
```

## 关于生产消费速度不一致验证影响性能的分析
1. 使用 useAsyncSend (异步发送),不需要得到broker的ack,虽然有可能丢数,但非常快
2. 使用 useAsyncSend (异步发送),默认不启用流控,如果启用需要(connctionFactory.setProducerWindowSize(1024000);),在不启用流控时,达到阈值<systemUsage>时,会写盘到temp_storage目录,同时阻塞生产连接
3. 所以当生产者速度远大于消费者时,会频繁触发写盘,写盘的速度跟不上生产的速度，就会引起长时间阻塞生产,导致生产消费变慢


## 优化

### 生产者
1. 异步发送 ?jms.useAsyncSend=true
2. <constantPendingMessageLimitStrategy limit="-1"/> 超出限制的数据会被丢弃,-1为不丢弃
3. 设置流控,对性能没有太大提升,主要用于保护broker,关键还是要平衡生产消费的速度 (((ActiveMQConnectionFactory) connectionFactory).setProducerWindowSize(10000);)(异步发送需要显式设置)

### 消费者
1. 预取策略,非持久化topic 默认32766,代码里写死,不能设置高于值。(((ActiveMQConnectionFactory) connectionFactory).getPrefetchPolicy().setTopicPrefetch();)
2. 直通发送,((ActiveMQConnectionFactory) connectionFactory).setAlwaysSessionAsync(false);session维护了一个内部队列供订阅的消费者调度数据,使用直通发送,可以不在session的队列中存储数据,直接发送到consumer的连接中
3. 批量确认 ((ActiveMQConnectionFactory) connectionFactory)).setOptimizeAcknowledge(true);消费者比生产者速度慢的原因是消费者每次消费数据需要向broker确认,但生产者如果异步发送可以不需要,所以可以是消费者采用批量确认的方式
4. 使用MessageListener 而不是receive的方式, 使用receive需要额外维护队列



## 参考
https://activemq.apache.org/slow-consumer-handling.html
https://activemq.apache.org/producer-flow-control.html
https://www.iteye.com/blog/shift-alt-ctrl-2035321