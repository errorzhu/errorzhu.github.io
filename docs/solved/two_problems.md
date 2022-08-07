# 记两个问题(kibana 和 spark)

# 前言
最近工作中遇到了两个问题，一并在此记录一下。
1 使用nginx 反代kibana，跳转不过去的问题
2 spark yarn-cluster模式，Kerberos认证找不到kdc的问题

# nginx 访问kibana
## 背景
由于环境的要求，只有一台接口机可以开放端口，想通过同一端口不同的目录映射到后端不同服务。在配赠访问kibana时，跳不过去。
## 解决
1 将kibana.yml 中 
```
server.basePath: "/kibana
```
2 nginx 配置
```
server 
    listen 8888;
    location /kibana {
      rewrite ^/kibana/(.*)$ /$1 break;
      proxy_pass http://ip:5601;
    }
  }



```
## 原理
server.basePath 这个参数会使url的请求不是以根路径开始的，而是有你设置的开始的(现在需要请求http://ip:5601/kibana才能访问)
nginx添加rewrite规则，因为它明确表明代理需要在请求到达Kibana服务器之前从请求中删除基本路径。
通过以上两条，原先登录跳转的http://localhost:5601/app/kibana#/home?_g=()，会变成http://localhost:5601/kibana/app/kibana#/home?_g=(),而成功的被nginx拦截到，跳转到http://real_server_ip:5601/kibana/app/kibana#/home?_g=()

# spark yarn-cluster模式，Kerberos认证找不到kdc的问题
## 背景
1 使用spark 写了一个elasticsearch的数据导入程序，将hdfs文件导入es。在使用yarn-client 模式下，kerberos认证没问题，但切换成cluster模式就报 cannot locate KDC。
2 环境使用etl服务器安装spark-client 来执行spark 提交作业，etl服务器可以根据hadoop_conf 连接不同的集群。

##解决
在spark-submit 提交任务的脚本中，加入如下变量
```
    export SPARK_SUBMIT_OPTS="-Djava.security.krb5.conf=/var/krb5.conf 
    #这里的krb5.conf 路径是用于提交spark作业节点的本地krb5.conf路径
```
```

export SPARK_SUBMIT_OPTS="-Djava.security.krb5.conf=/var/krb5.conf 
spark-submit --master yarn \ 
--deploy-mode cluster \
--conf spark.driver.extraJavaOptions=" -Djava.security.krb5.conf=/etc/krb5.conf" \
--class xxxx  \
 xxxx.jar



```
##原理
1 spark client 和cluster模式最大的区别就是driver启动的位置不同，client 模式是在提交节点启动（可以不是集群中的节点，而是接口机），而cluster模式是yarn 所启动的application master中启动（必然为集群中的一个节点）。
2 设置的spark.driver.extraJavaOptions ,spark.executor.extraJavaOptions 这两个参数 Djava.security.krb5.conf=/var/krb5.conf，其实是传递给driver进程和executor进程的jvm参数，和本地提交是无关的
3 以client 模式spark.driver.extraJavaOptions="-Djava.security.krb5.conf=/var/krb5.conf",指定的krb5.conf 是本地的路径，因为driver启动在本地。而与此相反，cluster模式的该参数应该设置成集群节点中的krb5的位置。
4 cluster模式即使设置了集群节点的位置还是报找不到kdc，经查阅源码
```
private Tuple4<Seq<String>, Seq<String>, SparkConf, String> doPrepareSubmitEnvironment(SparkSubmitArguments args, Option<Configuration> conf)
...
...
  .MODULE$.require((new File(args.keytab())).exists(), new NamelessClass_8(args));
  sparkConf.set(org.apache.spark.internal.config.package..MODULE$.KEYTAB(), args.keytab());
  sparkConf.set(org.apache.spark.internal.config.package..MODULE$.PRINCIPAL(), args.principal());
  UserGroupInformation.loginUserFromKeytab(args.principal(), args.keytab());

```
在通过spark-submit提交作业的过程中，在执行doPrepareSubmitEnvironment方法时，已经通过
 UserGroupInformation.loginUserFromKeytab 进行了Kerberos认证。而默认读取的是/etc/krb5.conf。由于上文所说，该服务器作为etl服务器链接不同集群，查看/etc/krb5.conf，果然没有所需要链接集群的realm，因此找不到kdc。
5 由于无权修改此文件，只能想到将jvm参数传递到提交作业进程之中，在ibm的知识库中，找到SPARK_SUBMIT_OPTS该变量，设置成本地的krb5，在提交进程中做正确的认证。

## 疑问
从源码上来看，在启动driver之前就已经做了Kerberos认证，那为什么client模式，没有设置这个参数就可以正常运行呢？后续再研究一下




# 参考资料
[https://www.ibm.com/support/knowledgecenter/SSZU2E_2.3.0/managing_cluster/kerberos_submit_applications.html](https://www.ibm.com/support/knowledgecenter/SSZU2E_2.3.0/managing_cluster/kerberos_submit_applications.html)
[https://discuss.elastic.co/t/nginx-reverse-proxy-setup-for-kibana/167327/5](https://discuss.elastic.co/t/nginx-reverse-proxy-setup-for-kibana/167327/5)
[https://discuss.elastic.co/t/kibana-5-4-behind-nginx/98114/5](https://discuss.elastic.co/t/kibana-5-4-behind-nginx/98114/5)
[https://discuss.elastic.co/t/kibana-and-nginx-in-subpath/90280](https://discuss.elastic.co/t/kibana-and-nginx-in-subpath/90280)
