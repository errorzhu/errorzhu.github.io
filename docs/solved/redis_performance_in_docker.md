# 运行在docker中的redis的性能问题

## 背景

通过redis-benchmark对redis做性能测试的时候,发现吞吐只有2w左右，觉得很奇怪。

```
$# redis-benchmark -t set,get,incr -n 1000000 -q  
SET: 25440.76 requests per second, p50=1.007 msec                   
GET: 25738.70 requests per second, p50=0.999 msec                   
INCR: 25499.80 requests per second, p50=1.007 msec                   


```



## 查询与测试

1. 首先想到可能是通过docker网桥做了转发的原因，于是使用host模式启动容器发现还是不行

2. 发现了参考资料中的文章，关闭了seccomp，运行容器，性能如下

   ```
   SET: 61413.74 requests per second, p50=0.407 msec                   
   GET: 64288.01 requests per second, p50=0.399 msec                   
   INCR: 62640.94 requests per second, p50=0.399 msec                   
   
   ```

3. 又看到一篇文章，说是benchmark测试出的结果不一致，于是在容器外边使用benchmark测试了一下，发现果然性能能到6w左右





## 结论

应该是容器内的seccomp的限制了benchmark测试的结果，具体深入原因，后续再深入研究







## 参考资料

https://www.threatmark.com/dockerized-redis-performance-on-centos-7-5-2/