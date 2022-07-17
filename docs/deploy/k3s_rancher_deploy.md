1. 准备

rancher 需要满足严格的版本兼容性，请参考[https://www.suse.com/suse-rancher/support-matrix/all-supported-versions/rancher-v2-5-9/](https://www.suse.com/suse-rancher/support-matrix/all-supported-versions/rancher-v2-5-9/)
安装docker

2. 安装rancher
```
docker pull rancher/rancher:v2.5.9
docker run --privileged -d --name rancher --restart=unless-stopped -p 8080:80 -p 8443:443 -v /opt/rancher:/var/lib/rancher rancher/rancher:v2.5.9
```

3. 安装k3s
```
mkdir -p /var/lib/rancher/k3s/agent/images/
从http://mirror.cnrancher.com/ 下载对应版本镜像包,拷贝至上述目录
cp k3s-airgap-images-arm64 /var/lib/rancher/k3s/agent/images/

curl -sfL http://rancher-mirror.cnrancher.com/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn INSTALL_K3S_VERSION=v1.19.10+k3s1 sh -
```

4. 确认k3s安装
```
kubectl get pods -A
```

5. 导入k3s

登录rancher
add cluster ---》 起个名字
在服务器上执行生成的命令，进行注册
```
curl --insecure -sfL https://192.168.5.161:8443/v3/import/rsm2pwbbwz9fkzg2tf5jjmz6kmg62pzhbrcffzzp9gflrqt6fcb8p5_c-kj2wv.yaml | kubectl apply -f -
```