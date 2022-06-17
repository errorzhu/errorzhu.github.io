# arm64 centos7 yum 安装非常慢的问题

## 背景
想在arm架构的服务器上做编译适配一些软件,启动arm架构的centos隔离编译环境,但是yum安装包的时候,安装过程非常慢

## 分析
通过docker stats查看,该容器cpu使用率为100%,首先想到,扩展docker 容器使用的cpu来补充资源,增加 --cpus=N,后cpu使用率依然为100%,没有再上升
经过查询yum是单进程操作,至多消耗一个cpu资源，因此对容器扩充cpu无效。

又怀疑是rpm安装过程中，编译耗费的时间，经查询资料，rpm是预先编译的，安装过程只是将包安装到对应目录，此猜测也被否定。

最后偶然发现linux论坛的问题，在容器内使用ulimit -n 查看,数值非常大，将此数值调小，再次使用yum安装,正常！

## 猜测
在宿主机执行ulimit -n得到的文件数量是小于，在容器内执行得到的数量的，怀疑和此有关，后续继续研究



## 参考 

https://www.linuxquestions.org/questions/linux-software-2/docker-fedora-31-installing-rpms-for-redhat-variants-slow-at-scriptlet-start-4175667314/