# 使用vagrant快速创建虚拟机

# 背景
测试部署需要纯净的虚拟机环境，使用vmare 或 virtualbox 图形化创建很慢，这里使用vagrant通过几条命令就能部署一个虚拟机集群

# 准备及安装
下载安装 virtualbox https://www.virtualbox.org/
下载安装 vagrant https://www.vagrantup.com/
下载  基础镜像文件 http://www.vagrantbox.es/

# 怎么做
```
#添加镜像
vagrant box add test vagrant-centos-7.2.box
```
```
#初始化
vagrant init
#目录下生成一个Vagrantfile
```
修改Vagrantfile
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|

    update_yum = <<-SCRIPT
    sudo mkdir -p /etc/yum.repos.d/bak
    sudo mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/bak
    sudo wget http://mirrors.163.com/.help/CentOS7-Base-163.repo -O /etc/yum.repos.d/163.repo
    sudo wget http://mirrors.aliyun.com/repo/epel-7.repo -O /etc/yum.repos.d/epel-7.repo
    sudo yum clean all 
    sudo yum makecache
    SCRIPT


    common = <<-SCRIPT
    sudo yum -y upgrade parted
    sudo yum install -y cloud-utils-growpart
    sudo parted /dev/sda resizepart 2 100%
    sudo pvresize /dev/sda2
    sudo lvextend -l +100%FREE /dev/centos/root
    sudo xfs_growfs /dev/centos/root
    SCRIPT

   config.disksize.size = '50GB'
   (1..3).each do |i|
        config.vm.define "node#{i}" do |node|
		
			
            # 设置虚拟机的Box
            node.vm.box = "test"
			
			#node.vm.disk :disk, size: "50GB", primary: true
			#node.vm.disk :disk, size: "50GB", name: "extra_storage"


            # 设置虚拟机的主机名
            node.vm.hostname="hadoop#{i}"

            # 设置虚拟机的IP
            node.vm.network "public_network", ip: "192.168.9.#{236+i}" ,bridge: "Qualcomm Atheros QCA9377 Wireless Network Adapter"

            # 设置主机与虚拟机的共享目录
            #node.vm.synced_folder "./", "/opt"

            
            # VirtaulBox相关配置
            node.vm.provider "virtualbox" do |v|
                v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
                # 设置虚拟机的名称
                v.name = "hadoop#{i}"
                # 设置虚拟机的内存大小
                v.memory = 4096
                # 设置虚拟机的CPU个数
                v.cpus = 1
            end
            #增加默认路由，否则无法上互联网
            node.vm.provision "shell",run: "always",inline: "ip route add default via 192.168.9.1"
            #修改yum 源 update_yum
            node.vm.provision :shell, :inline => update_yum            
			      #增加硬盘
			      node.vm.provision :shell, :inline => common
            # 修改root密码
            node.vm.provision "shell", inline: "echo '1' | passwd root --stdin"
        end
   end
end
```
启动
```
vagrant up
```
连接
```
vagrant ssh
```
修改ssh 配置
```
vi /etc/ssh/sshd_config

#放开下面注释
PasswordAuthentication yes
LoginGraceTime 2m
PermitRootLogin yes
StrictModes yes
```
关闭虚拟机
```
vagrant halt
```
删除虚拟机
```
vagrant destroy
```