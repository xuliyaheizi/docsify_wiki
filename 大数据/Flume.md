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