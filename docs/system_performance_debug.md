# 系统运行状态分析通用方法论

## cpu
top
```
top - 01:53:51 up 184 days, 22:20, 12 users,  load average: 10.88, 10.09, 8.87
Tasks: 366 total,   1 running, 358 sleeping,   7 stopped,   0 zombie
%Cpu(s): 41.0 us, 15.0 sy,  0.0 ni, 41.4 id,  0.3 wa,  0.0 hi,  2.2 si,  0.2 st
KiB Mem : 32779428 total,   324692 free, 27374256 used,  5080480 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  3242740 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                               
 2149 root      20   0   10.7g   4.8g  15996 S 178.0 15.3 210944:39 java                                                                                                                                  
22379 polkitd   20   0 6294708   1.3g   3152 S 145.1  4.3   1702:23 mysqld 
```
通过top命令可以查看系统cpu/内存的使用情况
cpu主要看以下三项
us: 用户空间占用 CPU 的百分比
sy: 表示内核空间占用 CPU 的百分比,(可以排查是否系统调用很多,是否硬中断很多)
wa: 表示等待 I/O 的时间百分比(数值较高通常意味着系统中存在大量的 I/O 操作)

vmstat

```
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 7  0      0 312936      0 4693308    0    0     9   373    0    0 14 10 76  0  0
 5  0      0 311872      0 4694064    0    0     0  1047 38527 68439 32 15 53  0  0
10  0      0 250148      0 4698604    0    0    80  1283 28939 48661 36 14 49  0  0
14  0      0 242920      0 4693956    0    0    48 56600 38108 63757 38 20 41  0  0
 5  0      0 308664      0 4656800    0    0    96 38102 38307 67732 34 17 49  0  0

```
补充看一下上下文切换cs这个值,表示线程切换次数

## 内存

free 和 top
看下内存是否充足


## 磁盘

iostat -kdx 2 10

```
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.00    0.00    1.00     0.00    13.25    26.50     0.00    4.00    0.00    4.00   0.00   0.00
vdb               0.00     0.00    1.50  199.00    22.00 32823.25   327.63     1.59   11.27    1.00   11.35   0.36   7.25
scd0              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00


```
rkB/s: 每秒钟从磁盘读取的数据量（以KB为单位）。
wkB/s: 每秒钟写入到磁盘的数据量（以KB为单位）。
await: 平均请求等待时间（平均请求的等待时间，包括队列时间和服务时间）。
r_await: 平均读请求等待时间。
w_await: 平均写请求等待时间。
svctm: 平均请求服务时间（请求在设备上的服务时间）。
%util: 设备的利用率（设备的工作时间百分比）。

主要看%util这个值


iotop
```

Total DISK READ :      12.05 K/s | Total DISK WRITE :	    3.90 M/s
Actual DISK READ:       0.00 B/s | Actual DISK WRITE:    1281.33 K/s
  TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND                                                                                                                                     
 9725 be/4 root        0.00 B/s  542.04 K/s  0.00 %  3.93 % java -Xms64M -Xmx1G -Djava.util.logging.config.file=logging.properties ~activemq.data=/data/activemq -jar /opt/activemq//bin/activemq.jar start
```
直观看进程对应的磁盘读写


## 网络

nload
查看一下网络流量，有没有到网卡带宽瓶颈


sar -n DEV 1 查看各个网卡速率

sar -n TCP,ETCP 1 查看发起的tcp连接数量


## 基本原则

如果load超过了cpu核数，则负载过高
如果wa%过高，可初步判断I/O有问题
sy%,si%,hi%,st%，任何一个超过5%，都有问题
进程状态长时处于D、Z、T状态，需要关注

cpu负载高，cpu使用率正常
导致cpu负载高有很多原因。CPU 密集型进程，使用大量 CPU 会导致平均负载升高；大量等待 CPU 的进程调度也会导致平均负载很高，此时 CPU 使用率也会比较高。

cpu负载高，同时cpu id%高
通过top看到，cpu id%高，也就是空闲，比如90%。但cpu负载非常高，比如4核达到10。
cpu负载高，说明其任务已经排队，许多任务正在等待。出现此种情况，很可能是系统中存在大量进程处于D的状态，也就是不可中断的睡眠状态，这一般是由于硬件问题导致的。此时可以使用 iostat 或 iotop，它们将指示哪些进程正在执行更多的 I/O 操作，是否io到达瓶颈 

机器使用卡顿，cpu %wa高
top 查看机器CPU负载非常高，说明其任务已经排队，许多任务正在等待。且CPU wa%值很高，%wa指CPU等待磁盘写入完成的时间，怀疑磁盘性能负载过高导致

iostat或者查看磁盘监控进一步判断。await 响应时间应该低于5ms，如果大于10ms就比较大了。如果svctm的值与await很接近，表示几乎没有IO等待，磁盘性能很好。如果await的值远高于svctm的值，则表示IO队列等待太长，系统上运行的应用程序将变慢。





参考资料
https://cloud.tencent.com/developer/article/1553861

https://learn.microsoft.com/zh-cn/troubleshoot/azure/virtual-machines/troubleshoot-performance-bottlenecks-linux


