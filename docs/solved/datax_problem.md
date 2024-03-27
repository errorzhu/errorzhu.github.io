# datax 相关问题

## 支持parquet格式

1. 修改 https://github.com/alibaba/DataX/blob/master/hdfswriter/src/main/java/com/alibaba/datax/plugin/writer/hdfswriter/HdfsWriter.java#L131 增加如下代码块

```
            else if(fileType.equalsIgnoreCase("PARQUET")){
                this.unitizeParquetConfig(writerSliceConfig);
            }
```

2. 修改https://github.com/alibaba/DataX/blob/master/hdfsreader/src/main/java/com/alibaba/datax/plugin/reader/hdfsreader/DFSUtil.java#L937 修改成如下代码，打印具体栈信息

```
LOG.error("Inner error, getParquetSchema failed, message is {}", e);
```
    
3. 因maven中央仓库不再提供pentaho-aggdesigner-algorithm-5.1.5-jhyde.jar的下载，手动install到本地repository

4. 使用maven 3.5.4版本，执行mvn clean package -U -DskipTests assembly:assembly

5. 去除编译后的包目录/plugin/reader/hdfsreader/libs 的parquet-format-2.3.0.jar，否则执行会抛出no such method 的异常