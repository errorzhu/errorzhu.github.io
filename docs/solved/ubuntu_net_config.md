# ubuntu 网络ip设置的乌龙

## 背景
今天和同事一起设置服务器静态ip时,发现使用netplan并不生效。发现自己对这块的知识不是很熟悉,又重新整理了一下。

## 原理
ubuntu桌面版和服务器版的默认的网络管理工具并一样
	桌面版: networkmanager: 就是桌面任务栏中的图形化配置工具(也提供nmcli作为命令行工具)
	服务器版: netplan: 支持使用yaml格式的配置管理网络信息的工具
并且从Ubuntu Server 18.04开始，服务器版本已经全面采用 systemd-networkd 并使用 netplan网络配置 来替代NetworkManager配置网络。所以，不建议在服务器上使用NetworkManager。
我们正好是 18.04的桌面版本,但我们一般使用命令行来操作,因此我们实际选择的方式是使用netplan生成networkd的配置
在/etc/netplan/下建一个yml文件
```
network:
 version: 2
 renderer: networkd
 ethernets:
   eth0:
     dhcp4: no
     dhcp6: no
     addresses: [自己的ip/24]
     gateway4: 网关地址
     nameservers:
       addresses: [8.8.8.8,8.8.4.4]

```
然后使用netplan apply生成配置。
这样操作是隐式的使用了networkd配置了网络信息,与图像化界面看到的配置并不一样。乌龙就在这发生了。其实我已经修改了静态ip,但同事通过network manager查看到的并没有修改,于是他重新启动了network manager,这样导致network manager又覆盖了我刚才的配置,所以看起来没有生效.

## 总结

首先，当系统内没有第三方网络管理工具（比如nm）时，系统默认使用networkd文件内的参数进行网络配置。接着，当系统内安装了 nm之后，nm默认接管了系统的网络配置，使用nm 自己的网络配置参数来进行配置。但是，如果用户在安装nm之后(我们的Desktop版本默认安装了nm),自己手动修改了interfaces 文件，那nm 就自动停止对系统网络的管理，系统改使用interfaces 文件内的参数进行网络配置。此时，再去修改nm 内的参数，不影响系统实际的网络配置。若要让nm 内的配置生效，必须重新启用nm 接管系统的网络配置。

 
