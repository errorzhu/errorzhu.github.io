# spark on k8s

# 前言
最近在做k8s相关工作，发现使用spark提交k8s集群，搜到的一些资料都是在k8s集群之内单独起pod来提交作业的，但这样和我们生产应用有些出入。一般情况，我们肯定是部署spark client 于跳板机，从跳板机提交任务，而不会直接操作k8s集群。调试了一下，跑通了，特此记录。

# 流程
## 1 创建 serviceaccount
``` 
kubectl create serviceaccount spark
kubectl create clusterrolebinding spark-role --clusterrole=edit --serviceaccount=default:spark --namespace=default
```
## 2 把spark镜像推至registry或在工作节点本地load

## 3 准备ca.crt 和 serviceaccount 的token
```
#ca.crt 在主节点
/etc/kubernetes/pki/ca.crt
#获取serviceaccount token
kubectl  get secrets -o jsonpath="{.items[?@.metadata.annotations['kubernetes\.io/service-account\.name']=='spark'}].data.token}"
```
## 4 在k8s集群外安装一台spark client

## 5 访问集群apiserver的两种方式

### 5.1 开启proxy，访问proxy来屏蔽对apiserver的直接访问
```
kubectl proxy --address=ip --disable-filter=true
#指定master为代理的地址
spark-submit --master k8s://http://ip:8001
```
### 5.2 在提交任务时传入ca.crt 和token
```
spark-submit \
--master k8s://https://<k8s-apiserver-host>:<k8s-apiserver-port> \
--conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
--conf spark.kubernetes.authenticate.submission.caCertFile=ca.crt \
--conf spark.kubernetes.authenticate.submission.oauthTokenFile=token \
```

## 6 两种提交模式

### 6.1 cluster
```
spark-submit \
--master k8s://https://<k8s-apiserver-host>:<k8s-apiserver-port> \
--deploy-mode cluster \
--name spark-pi \
--class org.apache.spark.examples.SparkPi \
--conf spark.executor.instances=5 \
--conf spark.kubernetes.container.image=<spark-image> \
--conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
--conf spark.kubernetes.authenticate.submission.caCertFile=ca.crt \
--conf spark.kubernetes.authenticate.submission.oauthTokenFile=token \
local:///path/to/examples.jar
```

### 6.2 client
(因为driver在本地，必须保证和k8s创建的executor通信，需要显示声明 driver port，driver host 及 blockManager port 从而使k8s集群内executor可以找到driver)

```
#注意 cafile 和 token的参数和cluster模式下不一样
spark-submit \
--master k8s://https://<k8s-apiserver-host>:<k8s-apiserver-port> \
--deploy-mode client\
--name spark-pi \
--class org.apache.spark.examples.SparkPi \
--conf spark.executor.instances=5 \
--conf spark.kubernetes.container.image=<spark-image> \
--conf spark.driver.bindAddress=0.0.0.0 \
--conf spark.driver.host=提交的客户端ip \
--conf spark.driver.port=19987  \
--conf spark.blockManager.port=19988 \
--conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
--conf spark.kubernetes.authenticate.caCertFile=ca.crt \
--conf spark.kubernetes.authenticate.oauthTokenFile=token \
local:///path/to/examples.jar

```

# 也记录一下在k8s集群内起pod提交

## 以serviceaccount 启动个pod
```
kubectl run sparkclient -it  -n default --serviceaccount='spark'  --image=<spark-image> -- /bin/bash
```
## 两种提交方式

### cluster
```
spark-submit \
--master k8s://https://<k8s-apiserver-host>:<k8s-apiserver-port> \
--deploy-mode cluster \
--name spark-pi \
--class org.apache.spark.examples.SparkPi \
--conf spark.executor.instances=5 \
--conf spark.kubernetes.container.image=<spark-image> \
local:///path/to/examples.jar

```

### client

```
#启动无头服务
kubectl expose deployment sparkclient --port=19987 --type=ClusterIP --cluster-ip=None

#driver.host 为dns 对应的域名 （服务名+namespace+svc+cluster.local）

spark-submit \
--master k8s://https://<k8s-apiserver-host>:<k8s-apiserver-port> \
--deploy-mode client\
--name spark-pi \
--class org.apache.spark.examples.SparkPi \
--conf spark.executor.instances=5 \
--conf spark.kubernetes.container.image=<spark-image> \
--conf spark.driver.host=sparkclient.default.svc.cluster.local \
--conf spark.driver.port=19987  \
local:///path/to/examples.jar

```



# 参考资料
[http://spark.apache.org/docs/latest/running-on-kubernetes.html](http://spark.apache.org/docs/latest/running-on-kubernetes.html)
