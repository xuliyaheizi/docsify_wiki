# Flume日志收集系统

## 一、概述

Flume是Cloudera提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统，Flume支持在日志系统中定制各类数据发送方，用于收集数据；同时，Flume提供对数据进行简单处理，并写到各种数据接受方（可定制）的能力。

Flume 事件被定义为具有字节有效负载和一组可选字符串属性的数据流单元。Flume 代理是一个（JVM）进程，它托管事件从外部源流到下一个目标（跃点）的组件。Flume主要的作业是实时读取服务器本地磁盘的数据，将数据写入HDFS。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220623152049503.png" alt="image-20220623152049503" width="50%;" />

### 工作流程

Source采集数据并包装成Event，并将Event缓存在Channel中，Sink不断从Channel获取Event，并解决成数据，最终将数据写入存储或索引系统。

- `Agent`：Agent是一个JVM进程，以事件的形式将数据从源头送至目的。主要有三部分：Source、Channel、Sink
- `Source`：Source是负责接收数据到Flume Agent的组件，采集数据并包装成Event。Source组件可以处理各种类型、各种格式的日志数据，包括avro、thrift、exec、jms、spooling directory、netcat、sequence generator、syslog、http、legacy。
- `Sink`：Sink不断地轮询Channel中的事件且批量移除他们，并将这些事件批量写入到存储或索引系统、或者被发送到另一个Flume Agent。Sink组件目的地包括hdfs、logger、avro、thrift、ipc、file、HBase、solr、自定义。
- `Channel`：Channel是位于Source和Sink之间的缓冲区。因此，Channel允许Source和Sink运作在不同速率上。Channel是线程安全的，可以同时处理几个Source的写入操作和几个Sink的读取操作。Flume自带两种Channel：Memory Channel和File Channel
  - Memory Channel是内存中的队列，在不需要关系数据丢失的情景下适用。若需要关心数据丢失，那么Memory Channel就不适合使用，因为程序死亡、机器宕机或者重启都会导致数据丢失。
  - File Channel将所有事件写到磁盘，因此在程序关闭或者机器宕机的情况下不会丢失数据。
- `Event`：传输单元，Flume数据传输的基本单元，以Event的形式将数据从源头送至目的地。Event由Header和Body两部分组成，Header用来存放该event的一些属性，为K-V结构，Body用来存放该条数据，形式为字节数组。

## 二、多数据源合并案列

**案例**：node1、node2、node3节点运行tomcat项目，flume采集tomcat运行日志，发送给master节点，master将最终数据存入hdfs

node1、node2、node3上Flume配置（tomcatlogs-hdfs.conf） t1

```shell
t1.sources=r1
t1.sinks=k1
t1.channels=c1

#配置Sources
t1.sources.r1.type=exec
t1.sources.r1.command=tail -F /usr/local/tomcat/logs/localhost_access_log.txt
t1.sources.r1.shell=/bin/bash -c
t1.sources.r1.batchSize=10
t1.sources.r1.batchTimeout=2000

#配置Sink
t1.sinks.k1.type=avro
t1.sinks.k1.hostname=master
t1.sinks.k1.port=4141

#配置Channel
t1.channels.c1.type=memory
t1.channels.c1.capacity=1000
t1.channels.c1.transactionCapacity=1000

#连接
t1.sources.r1.channels=c1
t1.sinks.k1.channel=c1
```

master上配置

```shell
t1.sources=r1
t1.sinks=k1
t1.channels=c1

t1.sources.r1.type=avro
t1.sources.r1.bind=0.0.0.0
t1.sources.r1.port=4141
t1.sources.r1.threads=10
t1.sources.r1.batchSize=1000

#配置Sink
t1.sinks.k1.type=hdfs
t1.sinks.k1.hdfs.path=hdfs://master:8020/flume/tomcat/%Y-%m-%d
t1.sinks.k1.hdfs.filePrefix=log-%Y-%m-%d
t1.sinks.k1.hdfs.round=true
t1.sinks.k1.hdfs.roundValue=1
#指定每个HDFS块的最小配置数。如果没有指定，则来着类路径中的默认  Hadoop配置
t1.sinks.k1.hdfs.minBlockReplicas=1
t1.sinks.k1.hdfs.fileType=DataStream
#序列文件格式
t1.sinks.k1.hdfs.writeFormat=Text
#重新定义时间单位
t1.sinks.k1.hdfs.roundUnit=hour
#是否使用本地时间戳
t1.sinks.k1.hdfs.useLocalTimeStamp=true
#积攒多少个Event才flush到hdfs一次
t1.sinks.k1.hdfs.batchSize=1000
#设置文件类型，可支持压缩
1.sinks.k1.hdfs.codeC=gzip
t1.sinks.k1.hdfs.fileType=CompressedStream
#多久生成一个新的文件
t1.sinks.k1.hdfs.rollInterval=600
#设置每个文件的滚动大小
t1.sinks.k1.hdfs.rollSize=134217700
#文件的滚动与Event数量无关
t1.sinks.k1.hdfs.rollCount=0

#配置Channel
t1.channels.c1.type=memory
t1.channels.c1.capacity=1000
t1.channels.c1.transactionCapacity=100

#连接
t1.sources.r1.channels=c1
t1.sinks.k1.channel=c1
```

## 三、Fluem采集日志上传到HDFS

```shell
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
## exec表示flume调用给的命令，然后从给的命令的结果中去拿数据
a1.sources.r1.type = exec
## 使用tail这个命令来读数据
a1.sources.r1.command = tail -F /home/hadoop/testdir/flumedata/test.log
a1.sources.r1.channels = c1

# Describe the sink
## 表示下沉到hdfs，类型决定了下面的参数
a1.sinks.k1.type = hdfs
## sinks.k1只能连接一个channel，source可以配置多个
a1.sinks.k1.channel = c1
## 下面的配置告诉用hdfs去写文件的时候写到什么位置，下面的表示不是写死的，而是可以动态的变化的。表示输出的目录名称是可变的
a1.sinks.k1.hdfs.path = hdfs://master:9000/flume/tailout/%y-%m-%d/%H%M/
##表示最后的文件的前缀
a1.sinks.k1.hdfs.filePrefix = events-
## 表示到了需要触发的时间时，是否要更新文件夹，true:表示要
a1.sinks.k1.hdfs.round = true
## 表示每隔1分钟改变一次
a1.sinks.k1.hdfs.roundValue = 1
## 切换文件的时候的时间单位是分钟
a1.sinks.k1.hdfs.roundUnit = minute
## 表示只要过了3秒钟，就切换生成一个新的文件
a1.sinks.k1.hdfs.rollInterval = 3
## 如果记录的文件大于20字节时切换一次
a1.sinks.k1.hdfs.rollSize = 20
## 当写了5个事件时触发
a1.sinks.k1.hdfs.rollCount = 5
## 收到了多少条消息往dfs中追加内容
a1.sinks.k1.hdfs.batchSize = 10
## 使用本地时间戳
a1.sinks.k1.hdfs.useLocalTimeStamp = true
#生成的文件类型，默认是Sequencefile，可用DataStream：为普通文本
a1.sinks.k1.hdfs.fileType = DataStream

# Use a channel which buffers events in memory
##使用内存的方式
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

