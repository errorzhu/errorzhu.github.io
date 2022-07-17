# 去hadoop大数据分析架构

# 前言
个人心目中的最佳大数据分析架构

- 简单
- 快速
- 可拓展
# 准备

- apache-hive-3.1.2-bin.tar.gz
- apache-hive-metastore-3.1.2-bin.tar.gz
- hadoop-3.2.3.tar.gz
- spark-3.2.1-bin-hadoop3.2.tgz

# 构建hive metastore镜像
```
mdkir build
# 准备二进制包
apache-hive-metastore-3.1.2-bin.tar.gz
hadoop-3.2.3.tar.gz
mysql-connector-java-8.0.19.jar
```
Dockerfile
```
FROM harbor.bdai.cosmo.com:8888/cosmo_bdai/openjdk:8-jdk-slim

WORKDIR /opt

ENV HADOOP_VERSION=3.2.3
ENV METASTORE_VERSION=3.1.2

ENV HADOOP_HOME=/opt/hadoop-${HADOOP_VERSION}
ENV HIVE_HOME=/opt/apache-hive-metastore-${METASTORE_VERSION}-bin

COPY apache-hive-metastore-${METASTORE_VERSION}-bin.tar.gz /opt
COPY hadoop-${HADOOP_VERSION}.tar.gz /opt
COPY mysql-connector-java-8.0.19.jar /opt


RUN tar zxf apache-hive-metastore-${METASTORE_VERSION}-bin.tar.gz  && \
    tar zxf hadoop-${HADOOP_VERSION}.tar.gz && \
    cp /opt/mysql-connector-java-8.0.19.jar ${HIVE_HOME}/lib/ && \
    rm -f ${HIVE_HOME}/lib/guava-19.0.jar && \
    cp ${HADOOP_HOME}/share/hadoop/common/lib/guava-27.0-jre.jar ${HIVE_HOME}/lib/ && \
    rm -rf /opt/apache-hive-metastore-${METASTORE_VERSION}-bin.tar.gz && \
    rm -rf /opt/hadoop-${HADOOP_VERSION}.tar.gz 

COPY conf/metastore-site.xml ${HIVE_HOME}/conf
COPY scripts/entrypoint.sh /entrypoint.sh

RUN chmod +x /entrypoint.sh

EXPOSE 9083

ENTRYPOINT ["sh", "-c", "/entrypoint.sh"]
```
其他文件
metastore-site.xml
```
<configuration>
    <property>
        <name>metastore.thrift.uris</name>
        <value>thrift://0.0.0.0:9083</value>
        <description>Thrift URI for the remote metastore. Used by metastore client to connect to remote metastore.</description>
    </property>
   
   <property>
    <name>metastore.task.threads.always</name>
    <value>org.apache.hadoop.hive.metastore.events.EventCleanerTask</value>
  </property>
  
  <property>
    <name>metastore.expression.proxy</name>
    <value>org.apache.hadoop.hive.metastore.DefaultPartitionExpressionProxy</value>
  </property>

    <property>
        <name>metastore.warehouse.dir</name>
        <value>s3a://spark/warehouse/</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.cj.jdbc.Driver</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://192.168.5.161:3306/metastore_db</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>admin</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>admin</value>
    </property>

    <property>
        <name>fs.s3a.access.key</name>
        <value>minioadmin</value>
    </property>
    <property>
        <name>fs.s3a.secret.key</name>
        <value>minioadmin</value>
    </property>
    <property>
        <name>fs.s3a.endpoint</name>
        <value>http://192.168.5.161:9000</value>
    </property>
    <property>
        <name>fs.s3a.path.style.access</name>
        <value>true</value>
    </property>

</configuration>

```
entrypoint.sh
```
#!/bin/sh

export HADOOP_HOME=/opt/hadoop-3.2.3
export HADOOP_CLASSPATH=${HADOOP_HOME}/share/hadoop/tools/lib/aws-java-sdk-bundle-1.11.901.jar:${HADOOP_HOME}/share/hadoop/tools/lib/hadoop-aws-3.2.3.jar
export JAVA_HOME=/usr/local/openjdk-8/

/opt/apache-hive-metastore-3.1.2-bin/bin/schematool -initSchema -dbType mysql
/opt/apache-hive-metastore-3.1.2-bin/bin/start-metastore

```
构建
```
docker build . -t hive-metastore:0.0.1
```
# 启动hive metastore standalone 和 minio
docker-compose.yml
```
version: "2"

services:
  mariadb:
    image: mariadb:10.7.3
    ports:
      - 3306:3306
    environment:
      MYSQL_ROOT_PASSWORD: admin
      MYSQL_USER: admin
      MYSQL_PASSWORD: admin
      MYSQL_DATABASE: metastore_db

  # make sure that you specify correct volume to be mounted
  minio:
    image: minio/minio:RELEASE.2020-02-20T22-51-23Z
    environment:
      - MINIO_ACCESS_KEY=minioadmin
      - MINIO_SECRET_KEY=minioadmin
    volumes:
      - /tmp/minio:/data
    ports:
      - 9000:9000
    command: server /data


```
```
docker-compose up -d
docker run -itd -p 9083:9083 --rm hive-metastore:0.0.1 /bin/bash
```
# 使用sparksql 查询s3
vim spark-defaults.conf
```
spark.sql.hive.metastore.version  3.1.2
spark.sql.hive.metastore.jars /opt/darch/apache-hive-metastore-3.1.2-bin/lib/*:/opt/darch/apache-hive-3.1.2-bin/lib/*
spark.hadoop.orc.overwrite.output.file true
spark.hadoop.fs.s3a.aws.credentials.provider org.apache.hadoop.fs.s3a.SimpleAWSCredentialsProvider
spark.hadoop.fs.s3a.endpoint http://ip:9000
spark.hadoop.fs.s3a.access.key minioadmin
spark.hadoop.fs.s3a.secret.key minioadmin
spark.hadoop.fs.s3a.impl org.apache.hadoop.fs.s3a.S3AFileSystem
```
vim hive-site.xml
```
<configuration>
    <property>
      <name>hive.metastore.uris</name>
      <value>thrift://ip:9083</value>
    </property>
</configuration>
```
```
/opt/darch/spark-3.2.1-bin-hadoop3.2/bin/spark-sql

CREATE EXTERNAL TABLE s3.hive_s3(  `id` string ) STORED AS TEXTFILE LOCATION  's3a://test/ss/'
select * from s3.hive_s3;
```
# 参考资料
[https://lrting.top/backend/2801/](https://lrting.top/backend/2801/)
[https://medium.com/@adamrempter/running-spark-3-with-standalone-hive-metastore-3-0-b7dfa733de91](https://medium.com/@adamrempter/running-spark-3-with-standalone-hive-metastore-3-0-b7dfa733de91)
