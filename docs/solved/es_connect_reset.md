# 记一次elasticsearch client的conncection reset 异常

# 背景 

这一段时间在测试elasticsearch的查询性能。我们有一个springboot开发的服务，集成了elasticsearch high level rest client 实现对es集群的查询。发现总是在测试闲置一段时间，再进行查询，就抛出若干connection reset。
![](..\images\wiki\es.jpg)


#分析
在客户端所在服务器执行
```
netstat -anpo | grep 9200
```
发现很多tcp连接，依然保持着和es集群的连接，因为es highlevel reset api 维护了一个连接池，并与集群保持长连接，节省查询开销。

但在es集群内查看，发现并无反向的连接存在。这就解释了会报出connection reset异常的原因。显然是因为客户端和服务端并没有走正常的4次挥手关闭tcp连接。这里只是服务端单方面的关闭了连接，而客户端并不知情，因此再次发送请求时，被服务端reset。

但服务端的连接为何会关闭，猜测是由于防火墙发现一定时长idle的连接，并只阻塞了单向的消息传递。

#模拟验证
在验证之前，说一下tcp keepalive 和 http keepalive

tcp keepalive 是tcp协议的一种保活策略，在连接空闲一定时长后，服务端会向客户端发送一个探测包，如果客户端响应，则继续保持连接。否则在一定间隔时间再发送若干探测包，如果均无响应，则关闭连接。
具体配置可以查看
```
sysctl -a | grep keepalive
/etc/sysctl.conf
```
http keepalive 是复用http 连接的一种策略，是实现长连接的方法。通过在请求头添加Connection: Keep-Alive，以表示连接可以复用。

## 模拟
1. 方便验证，首先在服务端服务器，将tcp keepalive参数调小，这样，在连接空闲较短时间，服务端便会向客户端发送探测包
2. 使用客户端进行请求，并通过netstat查看，已经建立起一条连接。
3. 开启防火墙，阻止服务端探测包发送到客户端
4.  使用netstat查看，客户端的连接还在，但服务端的连接已经不在了
5. 关闭防火墙，再次通过客户端发送请求，报connection reset
```
iptables -I OUTPUT -d 客户端ip -j DROP
```

#解决
1. 有条件可以修改操作系统参数的可以将tcp keepalive 时间调的比防火墙检测时间小
2. 在setKeepAliveStrategy中，使用自定义的实现
```
HttpAsyncClientBuilder.setKeepAliveStrategy()
```



# 参考资料
[https://juejin.im/entry/5bcfecf36fb9a05ce95c9cce](https://juejin.im/entry/5bcfecf36fb9a05ce95c9cce)
[https://stackoverflow.com/questions/52997697/how-to-get-around-connection-reset-by-peer-when-using-elasticsearchs-restclie](https://stackoverflow.com/questions/52997697/how-to-get-around-connection-reset-by-peer-when-using-elasticsearchs-restclie)