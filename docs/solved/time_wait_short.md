# 排查大量time_wait 连接的问题

## 背景
安装zookeeper,启动后有大量time_wait连接,但因连接时间短,无法用netstat/ss等命令捕获对应进程


## 分析

使用审计日志查看连接日志
```
auditctl -a exit,always -F arch=b64 -S connect -k MYCONNECT
tail -f /var/log/audit/audit.log
```
找到对应进程号，解决



## 参考 
https://blog.51cto.com/zxdlife/2117712