# 使用systemd管理elasticsearch

# 前言
为了将elasticsearch作为系统服务，统一管理，我们决定使用systemd来实现我们的一些需求。

# 需求
将elasticsearch作为系统服务管理，我们要实现哪些需求呢
1. 开机自启
2. 统一的关停重启命令
3.  服务挂掉可以自动重启
幸运的是，这些需求都可以通过systemd来实现。

#systemd 是什么
以前的linux系统由init进程来启动，缺点是需要串行执行，速度慢，无法解决脚本之间的复杂关系。systemd就是为系统的启动和管理提供一套完整的解决方案而诞生的。

# 初识
systemd 由一组命令组成
1. systemctl 主命令，管理服务
2. hostnamectl 主机信息
3. localectl 本地化设置
4. timedatectl 时区相关
5. loginctl 当前用户相关

# Unit
unit 是由systemd管理的资源或服务，将elasticsearch作为服务，我们就要将其加入unit
## 如何配置一个unit
/usr/lib/systemd/system/ 这个目录下存放着系统服务的unit配置文件,我们将自己的一个应用的配置加入到这个目录即可

```
# /usr/lib/systemd/system/elasticsearch.service
[Unit]
Description=elasticsearch  #描述
After=network.target #启动顺序，在network之后启动
Wants=network.target  #依赖关系，wants 表示若依赖，requires 表示强依赖，即network.target如果启动失败，elasticsearch也会退出

[Service]
Type=notify #启动类型 notify启动结束后会发出通知信号，然后 Systemd 再启动其他服务，其他类型参考官网
EnvironmentFile=/etc/sysconfig/elasticsearch#变量参数文件，kv格式，可以被下面读取，如$OPTIONS
ExecStart=/app/es/bin/elasticsearch -d  #启动时执行的命令
ExecReload=/bin/kill -HUP $MAINPID #重载时执行的命令
KillMode=mixed #主进程将收到 SIGTERM 信号，子进程收到 SIGKILL 信号
Restart=on-failure #重启的方式 ，elasticsearch任何意外的失败，都会重启
RestartSec=42s #重启等待的时间

[Install]
WantedBy=multi-user.target #表示该服务所在的 Target。
```
# 使用elasticsearch服务
## 重新加载systemd配置
```
sudo systemctl daemon-reload
```
## 开启elasticsearch服务
```
 sudo systemctl enable elasticsearch.service
# 会在 /etc/systemd/system/multi-user.target.wants 创建/usr/lib/systemd/system/elasticsearch.service的链接
# linux 在启动时会启动multi-user.target的服务，这就实现了开机自启的功能
```
##elasticsearch服务管理

```
#启动
systemctl start elasticsearch.service
#停止
systemctl stop elasticsearch.service
#重启
systemctl restart elasticsearch.service
```
##服务失败重启
```
Restart=on-failure #重启的方式 ，elasticsearch任何意外的失败，都会重启
RestartSec=42s #重启等待的时间

我们在unit里配置了以上的两个属性，也就实现了elasticsearch意外失败的重启
```



# 参考资料
[http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)
[https://linux.cn/article-6888-1.html](https://linux.cn/article-6888-1.html)
[https://wiki.archlinux.org/index.php/Systemd_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)](https://wiki.archlinux.org/index.php/Systemd_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
[https://www.freedesktop.org/software/systemd/man/systemd.unit.html](https://www.freedesktop.org/software/systemd/man/systemd.unit.html)
[http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html)
