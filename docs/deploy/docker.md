# docker 部署

```
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
				  
				  
				  
-------------------------------------------------------
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
  
  
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo  
	
sudo yum install docker-ce docker-ce-cli containerd.io

sudo systemctl start docker

vi /etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://bytkgxyr.mirror.aliyuncs.com"
  ]
}
-------------------------------------------------------
{
"registry-mirrors": [
 "https://mirror.ccs.tencentyun.com"
 ,"http://hub-mirror.c.163.com"
]
}
-------------------------------------------------------

sudo systemctl daemon-reload
sudo systemctl restart docker

-------------------------------------------------------
配置代理
sudo mkdir -p /etc/systemd/system/docker.service.d
vi /etc/systemd/system/docker.service.d/http-proxy.conf
[Service]
Environment="HTTPS_PROXY=http://192.168.21.23:1080/" "NO_PROXY=localhost,127.0.0.1,registry.docker-cn.com,hub-mirror.c.163.com"

sudo systemctl daemon-reload
sudo systemctl restart docker
systemctl show --property=Environment docker
```


