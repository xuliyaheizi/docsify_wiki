# Hadoop

## 一、概述

### 1.1、是什么

Hadoop是一个由Apache基金会所开发的分布式系统基础架构，主要解决海量数据的存储和海量数据的分析计算问题。广义上说，Hadoop是指Hadoop生态圈。

### 1.2、优势

- 高可靠性：Hadoop底层维护多个数据副本，所以即使某个计算元素或存储出现故障，也不会导致数据的丢失。
- 高扩展性：在集群间分配任务数据，可方便的扩展数以千计的节点。
- 高效性：在Map Reduce的思想下，Hadoop是并行工作的，以加快任务处理速度。
- 高容错性：能够自动将失败的任务重新分配。

### 1.3、组成

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220608200350845.png" alt="image-20220608200350845" width="50%;" />

### 1.4、HDFS架构概述

Hadoop Distributed File System，简称HDFS，是一个分布式文件系统。

1. `NameNode（nn）`：存储文件的元数据，如文件名，文件目录结构，文件属性（生成时间、副本数、文件权限），以及每个文件的块列表和块所在的DataNode等。
2. `DataNode(dn)`：在本地文件系统存储文件块数据，以及块数据的校验和。
3. `Secondary NameNode(2nn)`：每隔一段时间对NameNode元数据备份。

### 1.5、YARN架构概述

Yet Another Resource Negotiator 简称YARN ，另一种资源协调者，是`Hadoop`的资源管理器。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220608200758870.png" alt="image-20220608200758870" width="50%;" />

1. `ResourceManager(RM)`：整个集群资源（内存、CPU等）的管理者
2. `NodeManager(NM)`：单个节点服务器资源的管理者。
3. `ApplicationMaster(AM)`：单个任务运行的管理者。
4. `Container`：容器，相当于一台独立的服务器，里面封装了任务运行所需要的资源，如内存、CPU、磁盘、网络等。

### 1.6、MapReduce架构概述

MapReduce将计算过程分为两个阶段：Map和Reduce

- Map阶段并行处理输入数据
- Reduce阶段对Map结果进行汇总

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220608201047671.png" alt="image-20220608201047671" width="67%;" />

### 1.7、大数据生态体系

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220608201138312.png" alt="image-20220608201138312" width="67%;" />

1. `Sqoop`：Sqoop 是一款开源的工具，主要用于在Hadoop、Hive 与传统的数据库（MySQL）间进行数据的传递，可以将一个关系型数据库（例如 ：MySQL，Oracle 等）中的数据导进到Hadoop 的HDFS 中，也可以将HDFS 的数据导进到关系型数据库中。
2. `Flume`：Flume 是一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统，Flume 支持在日志系统中定制各类数据发送方，用于收集数据。
3. `Kafka`：Kafka 是一种高吞吐量的分布式发布订阅消息系统。
4. `Spark`：Spark 是当前最流行的开源大数据内存计算框架。可以基于Hadoop 上存储的大数据进行计算。
5. `Flink`：Flink 是当前最流行的开源大数据内存计算框架。用于实时计算的场景较多。
6. `Oozie`：Oozie 是一个管理Hadoop 作业（job）的工作流程调度管理系统。
7. `Hbase`：HBase 是一个分布式的、面向列的开源数据库。HBase 不同于一般的关系数据库，它是一个适合于非结构化数据存储的数据库。
8. `Hive`：Hive 是基于Hadoop 的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供简单的SQL 查询功能，可以将SQL 语句转换为MapReduce 任务进行运行。其优点是学习成本低，可以通过类SQL 语句快速实现简单的MapReduce 统计，不必开发专门的MapReduce 应用，十分适合数据仓库的统计分析。
9. `ZooKeeper`：它是一个针对大型分布式系统的可靠协调系统，提供的功能包括：配置维护、名字服务、分布式同步、组服务等。

## 二、HDFS分布式文件系统

HDFS是一个分布式文件系统，用于存储文件，通过目录树来定位文件，基于流数据模式访问和处理超大文件的需求而开发的，具有高容错性、高可靠性、高扩展性和高吞吐率等特征，适合`一次写入，多次读取`的场景。

`流式数据`：将数据序列化为字节流来存储，这样不会破坏文件的结构和内容。当超大规模的文件本身就已经超越了单台服务器的存储，需要多台服务器同时存储，需将文件序列化，然后按照字节的顺序进行切分后分布式的存储在各个服务器上。

若要将一个大的文件进行切分，该文件必须支持序列化。若要存储在文件系统中，该文件系统必须是流式数据访问模式。

HDFS将大规模的数据以分布式的方式均匀存储在集群中的各个服务器上，然后分布式并行计算框架mr就可以利用各个服务器的本地计算资源在本地服务器上对大规模数据集的一个子集数据进行计算。

### 2.1、优点

- 高容错性：数据自动保存多个副本，通过增加副本的形式，来提高容错性。某个副本丢失后，可以自动恢复。
- 适合处理大数据：数据规模能达到GB、TB、甚至PB级别。文件规模能够处理百万规模以上的文件数量。
- 可构建在廉价机器上：通过多副本机制，提高可靠性。
- 流式数据访问模式：一次写入，多次读取（离线、统计分析），写入后不允许修改，Hadoop适用于处理离线数据，不适合处理适时数据。HDFS的数据处理规模比较大，应用一次需要大量的数据，同时这些应用一般都是批量处理，而不是用户交互式处理。`主要的是数据的吞吐量，而不是访问速度`。

### 2.2、局限性

- 不适合低延时数据访问：无法访问毫秒级的存储数据。

- 无法高效的对大量小文件进行存储：存储大量的小文件会占用NameNode大量的内存存储文件目录和块信息，导致NameNode处理性能降低，且会制约HDFS的扩展性。小文件的寻址时间会超过读取时间，违背了HDFS的设计目标。大量的小型文件会给HDFS扩展性和访问处理性能带来严重问题，可通过`SequenceFile、MapFile`等方式归档小文件来解决。

  ```shell
  #hadoop archive -archiveName 指定归档文件的文件名 -p 需要归档的目录 归档文件的输出目录
  hadoop archive -archiveName input.har -p /input /output
  #查看归档文件
  hadoop fs -ls har:///output/input.har
  #解归档文件
  hadoop fs -cp har:///output/input.har/*  /wcinput
  ```

- 不支持并发写入、文件随机修改：一个文件只能一个写，不允许多个线程同时写，多线程同时写会涉及多线程安全问题，要解决多线程安全问题会造成文件系统处理性能上的损失。仅支持数据追加，不支持文件的随机修改。因为HDFS的数据冗余设计，当对任意一个位置进行修改，那么备份的数据也会进行修改，如此HDFS的开销会很大，不利于对文件数据的访问和处理。

### 2.3、组成架构

- NameNode：管理HDFS的名称空间，配置副本策略，管理数据块（BLOCK）映射信息，处理客户端读写请求。
- DataNode：NameNode下达命令后，DataNode执行实际的操作，存储实际的数据块，执行数据块的读/写操作。
- Client：客户端，文件切分，文件上传HDFS的时候，客户端将文件切分成一个一个的数据块，然后进行上传。与NameNode交互，获取文件的位置信息。与DataNode交互，读取或写入数据。命令管理或访问HDFS。
- Secondary NameNode：辅助NameNode，分担其工作量，定期合并Fsimage和Edits，并推送给NameNode。紧急情况下可辅助恢复NameNode。当NameNode挂掉的时候，并不能立刻替换NameNode并提供服务。

### 2.4、文件块大小

HDFS中的文件在物理上是分块存储的，块的大小可以通过配置参数来规定。

```xml
<!--配置hdfs-site.xml-->
<property>
	<name>dfs.block.size</name>
	<value>134217728</value>   
</property>

<!-- Java编程指定-->
conf.set("dfs.block.size", args[0]);
```

#### 为什么HDFS中的块大小设置为128M？

HDFS中平均寻址时间大概为10ms，最佳状态是寻址时间为传输时间的1%，即最佳传输时间为10ms/0.01=1s。

#### 为什么Block块不能设置太大，也不能设置太小？

Block块设置太大，使得从磁盘传输数据的时间会明显大于寻址时间，导致程序在处理这个块数据时，变得非常慢。另一方面，MapReduce中的Map任务通常一次只处理一个块中的数据，若块过大运行速度也会非常慢。

设置太小，存放大量小文件会占用NameNode中大量的内存来存储文件目录和块信息，另一方面文件快过小会导致寻址时间增大，导致程序一直在找block的开始位置。

#### 数据块抽象设计带来的好处

在企业生成实际环境中，因文件数据以数据块形式存储，一个文件的大小可以大于网络集群中任意一个磁盘的容量。使用块抽象而不是文件，简化了存储子系统，在集群的网络环境中，只需考虑文被切分后的数据块存储就可以了，对集群中任意一个节点上文件数据的存储就更易操作。数据块非常适用于数据的备份，从而提高数据的容错能力，当数据丢失时，可以以块为单位找回，而不涉及文件整体。当要使用一个文件时，只需要将这个文件对应的块进行临时的拼接。

### 2.5、常用命令

**上传**

```bash
# -moveFromLcoal：从本地剪切粘贴到HDFS
hadoop fs -moveFromLocal ./shugo.txt /sanguo

# -copyFromLocal：从本地文件系统中拷贝文件到HDFS路径中去
hadoop fs -copyFromLocal ./weiguo.txt /sanguo

# -put：效果与copyFromLocal相同
hadoop fs -put ./wuguo /sanguo

# -appendToFile：追加一个文件到已存在的文件末尾
hadoop fs -appendToFile liubei.txt /sanguo/shugo.txt
```

**下载**

```bash
# -copyToLocal：从HDFS拷贝到本地
hadoop fs -copyToLocal /sanguo/shuguo.txt ./

# -get：等同于copyToLocal
hadoop fs -get /sanguo/shuguo.txt ./
```

**基本命令**

```bash
# -ls：显示目录信息
hadoop fs -ls /sanguo

# -cat：显示文件内容
hadoop fs -cat /sanguo/shuguo.txt

# -chgrp、-chmod、-chown：与Linux文件系统中的用法一样，修改文件所属权限
hadoop fs -chmod 777 /sanguo/shuguo.txt
hadoop fs -chown zhulin:zhulin /sanguo/shuguo.txt

# -mkdir：创建路径
hadoop fs -mkdir /jinguo

# -cp：从HDFS的一个路径拷贝到HDFS的另一个路径
hadoop fs -cp /sanguo/shuguo.txt /jinguo

# -mv：在HDFS目录中移动文件
hadoop fs -mv /sanguo/shuguo.txt /jinguo

# -rm：删除文件或文件夹
hadoop fs -rm /sanguo/shuguo.txt

# -rm -r：递归删除目录及目录里面内容
hadoop fs -rm -r /sanguo

# -setrep：设置HDFS中文件的副本数量
hadoop fs -setrep 10 /jinguo/shuguo.txt
```

### 2.6、API操作

**创建目录**

```java
public void testMkdirs() throws URISyntaxException, IOException {
    //1.获取文件系统
    Configuration configuration = new Configuration();
    FileSystem fs = FileSystem.get(new URI("hdfs://zhulinz.top:8090"), configuration);
    //2.创建目录
    fs.mkdirs(new Path("/home/zhulin"));
    //3.关闭资源
    fs.close();
}
```

**上传文件**

```java
public void testCopyFromLocal() throws URISyntaxException, IOException {
    Configuration configuration = new Configuration();
    configuration.set("dfs.replication", "2");
    FileSystem fs = FileSystem.get(new URI("hdfs://zhulinz.top:8090"), configuration);
    //上传文件
    fs.copyFromLocalFile(new Path("D:\\IDEA工作区\\Hadoop\\HdfsClientDemo\\test1.txt"), new Path("/home/zhulin"));
    //关闭资源
    fs.close();
}
```

**下载文件**

```java
public void testCopyToLocal() throws URISyntaxException, IOException {
    //获取文件系统
    Configuration configuration = new Configuration();
    FileSystem fs = FileSystem.get(new URI("hdfs://zhulinz.top:8090"), configuration);

    //下载文件  delSrc 是否删除原文件  usrRawLocalFileSystem  是否开启文件校验
    fs.copyToLocalFile(false, new Path("/home/zhulin/test1.txt"), new Path("D:\\IDEA工作区\\Hadoop\\HdfsClientDemo" +"\\test2.txt"), true);
    //资源关闭
    fs.close();
}
```

```java
public void testDelete() throws URISyntaxException, IOException {
    Configuration configuration = new Configuration();
    FileSystem fs = FileSystem.get(new URI("hdfs://zhulinz.top:8090"), configuration);

    fs.delete(new Path("/home/zhulin/"), true);

    fs.close();
}
```

### 2.7、读写流程

#### 写数据流程

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220609164200158.png" alt="image-20220609164200158" width="67%;" />

1. 客户端通过`Distributed FileSystem`模块向`Name Node`请求上传文件，`Name Node`检查目标文件是否存在，父目录是否存在。
2. `Name Node`向客户端响应是否可以上传文件。
3. 客户端请求上传第一个数据块到哪几个`Data Node`服务器上。请求返回`Data Node`节点。
4. `Name Node`响应请求，返回dn1、dn2、dn3节点，表示采用这三个节点存储数据。
5. 客户端通过`FSDataOutputStream`模块请求dn1上传数据，然后依次调用dn2、dn3，请求建立`Block`传输通道。
6. `Data Node`节点依次应答客户端。
7. 客户端开始往dn1上传第一个`Block`（先从磁盘读取数据放到一个本地内存缓存），以`Packet`为单位，dn1收到一个Packet就会传给dn2，然后dn2传给dn3。dn1每传一个packet会放入一个应答队列等待应答。
8. 当一个`Block`传输完成后，客户端再次请求`Name Node`上传第二个Block的服务器。（依次重复3~7步）

### 2.8、机架感知

整个数据块副本存放的过程称为机架感知，默认是关闭的。

```shell
#查看机架感知
hdfs  dfsadmin  -printTopology
```

**编写脚本 rackaware.sh**

```shell
#!/bin/bash
HADOOP_CONF=/usr/hadoop/etc/hadoop/rackaware
while [ $# -gt 0 ] ; do
  nodeArg=$1
  exec<${HADOOP_CONF}/topology.data
  result=""
  while read line ; do
    ar=( $line )
    if [ "${ar[0]}" = "$nodeArg" ]||[ "${ar[1]}" = "$nodeArg" ]; then
      result="${ar[2]}"
    fi
  done
  shift
  if [ -z "$result" ] ; then
    echo -n "/default-rack"
  else
    echo -n "$result"
  fi
done
```

**topoloy.data**

```shell
192.168.10.200 master /dc1/rack1
192.168.10.201 node1 /dc1/rack2
192.168.10.202 node2 /dc1/rack2
192.168.10.203 node3 /dc1/rack3
```

**启用机架感知**

```xml
<!--在core-site.xml中加入-->
<property>
	<name>net.topology.script.file.name</name>
	<value>/home/zhulin/bin/rackaware.sh</value>
</property>
```

```shell
Rack: /dc1/rack1
   192.168.10.200:9866 (master)

Rack: /dc1/rack2
   192.168.10.201:9866 (node1)
   192.168.10.202:9866 (node2)

Rack: /dc1/rack3
   192.168.10.203:9866 (node3)
```

### 2.9、数据块副本的存放策略

以副本数为3为例，第一个副本放置在客户端所在的DataNode节点（如客户端不在集群内，则第一个DataNode随机选，但原则上仍是选取距离客户端近的DataNode），第二个副本放置在与第一个节点不同机架的DataNode中（随机选），第三个副本放置在与第一个副本所在节点同一机架的另一个节点上，若还有更多副本，就随机放。

可优先保证本机架下对该数据块所属文件的访问，即使机架发生故障，也可在另外的的机架上找到该数据块的副本。

### 2.10、数据块的备份数

```shell
#修改hdfs-site.xml
<property>
	<name>dfs.replication</name>
	<value>3</value>
</property>

#通过命令修改已经上传的文件的副本数
hadoop fs -setrep -R 3 /test

#上传文件的同时指定创建的副本数
hdfs dfs -Ddfs.replication=1 -put  core-site.xml /

#查看当前hdfs的副本数
hdfs fsck -locations
```

### 2.11、安全模式

```shell
#退出安全模式
hdfs dfsadmin -safemode leave
#进入安全模式
hdfs dfsadmin -safemode enter
#查看安全模式状态
hdfs dfsadmin -safemode get
#等待，直到安全模式结束
hdfs dfsadmin -safemode wait
#对hdfs文件系统进行检查
hdfs fsck /
		-move: 移动损坏的文件到/lost+found目录下
		-delete: 删除损坏的文件
		-files: 输出正在被检测的文件
		-openforwrite: 输出检测中的正在被写的文件
		-includeSnapshots: 检测的文件包括系统snapShot快照目录下的
		-list-corruptfileblocks: 输出损坏的块及其所属的文件
		-blocks: 输出block的详细报告
		-locations: 输出block的位置信息
		-racks: 输出block的网络拓扑结构信息
		-storagepolicies: 输出block的存储策略信息
		-blockId: 输出指定blockId所属块的状况,位置等信息
```

### 2.12、负载均衡

Hadoop的HDFS集群非常容易出现机器与机器之间磁盘利用率不平衡的情况，例如：当集群内新增、删除节点，或者某个节点机器内硬盘存储达到饱和值。当数据不平衡时，Map任务可能会分配到没有存储数据的机器，这将导致网络带宽的消耗，也无法很好的进行本地计算。

当HDFS负载不均衡时，需要对HDFS进行数据的负载均衡调整，即对各节点机器上数据的存储分布进行调整。从而使数据均匀的分布在各个DataNode上，均衡IO性能，防止热点的发生。进行数据的负载均衡调整，必须满足以下原则：

- 数据平衡不能导致数据块减少，数据块备份丢失。
- 管理员可以中止数据平衡进程。
- 每次移动的数据量以及占用的网络资源，必须是可控的。
- 数据均衡过程，不能影响NameNode的工作。

#### 负载均衡算法

数据均衡过程的核心是一个数据均衡算法，该数据均衡算法将不断迭代数据均衡逻辑，直至集群内数据均衡为止。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220615161022417.png" alt="image-20220615161022417" width="50%;" />

- 数据均衡服务（Rebalancing Server）首先要求NameNode生成DataNode数据分布分析报告，获取每个DataNode磁盘使用情况。
- Rebalancing Server汇总需要移动的数据分布情况，计算具体数据块迁移路线图。数据块迁移路线图，确保网络内最短路径。
  - 把当前的DataNode节点根据阈值的设定情况分到Over、Above、Below、Under四个组中。且在移动的过程中Over、Above组中的块向Below、Under组移动。
  - Over组，此组的DataNode均满足：DataNode_usedSpace_percent > Cluster_usedSpace_percent + threshold
  - Above组：Cluster_usedSpace_percent + threshold > DataNode_ usedSpace _percent > Cluster_usedSpace_percent
  - Below组：Cluster_usedSpace_percent > DataNode_ usedSpace_percent > Cluster_ usedSpace_percent – threshold
  - Under组：Cluster_usedSpace_percent – threshold > DataNode_usedSpace_percent
- 开始数据块迁移任务，Proxy Source Data Node复制一块需要移动迁移的数据块。
- 将复制的数据块复制到目标DataNode上。
- 删除原始数据块。
- 目标DataNode向Proxy Source Data Node确认该数据块迁移完成。
- Proxy Source Data Node向Rebalancing Server确认本次数据块迁移完成。然后继续执行这个过程，直至集群达到数据均衡标准。

#### 数据均衡命令

```shell
#数据自动平衡脚本
start-balancer.sh –threshold
		-threshold：默认设置：10，参数取值范围：0-100
		#参数含义：判断集群是否平衡的阈值。理论上，该参数设置的越小，整个集群就越平衡
		dfs.balance.bandwidthPerSec：默认设置：1048576（1M/S）
		#参数含义：Balancer运行时允许占用的带宽
		
#在hdfs-site.xml中设置数据均衡占用的网络带宽限制
<property>
	<name>dfs.balance.bandwidthPerSec</name>
	<value>1048576</value>
</property>

#设置定时任务来实现定时的负载均衡
00 22 * * 5 hdfs balancer -Threshold 5 >>/home/logs/balancer_`date +"\%Y\%m\%d"`.log 2>&1
```

### 2.13、心跳机制

主节点和从节点之间的通信是通过心跳机制（心跳实际上是一个RPC函数）实现的，master启动的时候，会开启一个RPC Server，slave启动时进行连接master，并每隔3秒钟主动向master发送一个“心跳”，将自己的状态信息告诉master，然后master通过这个心跳的返回值，向slave节点发送指令。

- HDFS：DataNode默认向NameNode`每隔3秒汇报一次`，包括`DataNode的状态信息以及所持有的数据块的信息`。若NameNode连续`10次`没有收到汇报，则认为可能存在宕机的可能。在DataNode启动后，会专门启动一个`负责心跳数据包的线程`，如果整个DataNode没有任何问题，只是负责发送心跳数据包的线程挂了。NameNode会发送命令向DataNode确认，查看心跳数据包的服务是否正常，为保险起见，一般会确认2次，每5分钟一次，当两次都没有返回结果，则认为DataNode节点挂了。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220615163638693.png" alt="image-20220615163638693" width="50%;" />

## 三、YARN资源管理

YARN是一个通用的资源管理平台，为各类计算框架提供资源的管理和调度。可将多种计算框架（离线处理MapReduce、在线处理的Storm、内存计算框架Spark等）部署到一个公共集群中，共享集群的资源。

- 资源的同一管理和调度：集群中所有节点的资源（内存、CPU、磁盘、网络等）抽象为Container（容器）。在资源进行运算任务时，计算框架需要向YARN申请Container，YARN按照策略对资源进行调度，进行Container的分配。
- 资源隔离：YARN使用了轻量级资源隔离机制Cgroup进行资源隔离，以避免相互打扰，一旦Contariner使用的资源量超过事先定义的上限值，就将其杀死。

### 体系结构

- **ResourceManager（RM）**：负责对各`NM`上的资源进行统一管理和调度。给`AM`分配空闲的Container并监控其运行状态。对AM申请的资源请求分配相应的空闲Container。其主要由两个组件构成：调度器和应用程序管理器。
  - `调度器（Scheduler）`：调度器根据容量、队列等限制条件，将系统中的资源分配给各个正在运行的应用程序。调度器仅根据各个应用程序的资源需求进行资源分配，而资源分配单位是Container，从而限定每个任务使用的资源量。
  - `应用程序管理器（Applications Manager）`：应用程序管理器负责管理整个系统中所有的应用程序，包括应用程序提交，与调度器协商资源以启动`AM`，监控`AM`运行状态并在失败时重新启动等。
- **NodeManager（NM）**：`NM`是每个节点上的资源和任务管理器。它会定时地向`RM`汇报本节点上的资源使用情况和各个Container的运行状态；同时会接收并处理来自`AM`的Container启动/停止等请求。
- **ApplicationMaster（AM）**：用户提交的应用程序均包含一个`AM`，负责应用的监控，跟踪应用执行状态，重启失败任务等。
- **Container**：Container封装了某个节点上的多维度资源，如内存、CPU、磁盘、网络等，是YARN对资源的抽象。当AM向RM申请资源时，RM为AM返回的资源便是用Container表示的。YARN会为每个任务分配一个Container且该任务只能使用该Container中描述的资源。

<img src="C:/Users/zhulin/AppData/Roaming/Typora/typora-user-images/image-20220618195053564.png" alt="image-20220618195053564" width="50%;" />

### 调度模型

采用了双层资源调度模型，RM中的资源调度器将资源分配给各个AM，资源分配过程是异步的。资源调度器将资源分配给一个应用程序后，不会立刻push给对应的AM，而是暂时放到一个缓冲区中，等待AM通过周期性的心跳主动来取。

YARN目前采用的资源调度算法：

- `先来先调度FIFO`：先按照优先级高低调度，如优先级相同则按照提交时间先后顺序调度，若提交时间相同则按照队列名或Application ID比较顺序调度。
- `公平调度算法FAIR`：尽可能公平的调度，即已分配资源类少的优先级高。
- `主资源公平调度DRF`：算法扩展了最大最小公平算法，使之能够支持多维资源，算法是配置资源百分比小的优先级高。

### 优缺点和使用场景

- YARN使用了轻量级资源隔离机制Cgroups进行资源隔离以避免资源之间相互干扰，实现对CPU和内存两种资源的隔离。
- YARN上可以运行各种应用类型的计算框架，包括离线计算MapReduce、DAG计算框架Tez、基于内存的计算框架SPARK、实时计算框架Storm等。
- 支持先进先出FIFO、公平调度FAIR、主资源公平调度DRF等分配算法。
- 支持多租户资源调度，包括支持资源按比例分配、支持层级队列划分和支持资源抢占。

##四、集群高可用性

Hadoop1.X中每个集群只有一个NaemNode，使得HDFS中存在单点故障，难以应用在线上场景。NameNode压力过大，内存受限，影响扩展性。

- 针对单点故障的修复方案：HDFS HA高可用，通过主备2个Namenode（3.X上最多5个备，官方推荐3个备。NN太多导致很多数据发送，造成网络压力），主提供服务，备不提供，但是运行着。如果主Namenode发生故障，切换到备Namenode上。
- 解决内存受限问题：HDFS Federation（联邦），水平扩展，**支持多个Namenode**；同时对外提供服务，分治；
  每个Namenode分管一部分目录，所有的Namenode共享所有DataNode存储资源。

### 3.1、实现手动HA

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220613210414833.png" alt="image-20220613210414833" width="50%;" />

NameNode存了两类元数据：客户端产生的动态数据，生成的目录；DataNode汇报到block位置信息。Standby（备用NameNode）通过以下两种方式同步获取Active（主NameNode）上的元数据。

- 阻塞（为了保持数据一致性，丧失可用性）：客户端要求NN Active创建目录，NN Active向NN Standby发送指令创建目录，成功之后Standby返回ok给Active，Active再发送ok给客户端。如果Satndby中途挂掉，后续操作就阻塞了。
- 异步（为了可用性，丧失了一致性）：客户端要求Active创建目录，Activite向Standby传达相同指令。此时，activite不管standby，只要activite它自己创建完成，里面给客户端返回ok。但是standby创建目录的过程中，有可能挂掉。

不能为解决一个问题，从而引入另一个问题。只需实现最终数据一致性。借助中间的组件JN（JournalNodes），往Active中写数据，相当于写到了NFS里，读也是从里面读。客户端向Active写入数据，Active同时要写入到JN中（2个NN只能Active往JN写，JN放的是edits文件，JN可以做到可靠性存储数据，能保证最终一致性。和NFS干的活一样，不过实现技术不一样。）然后Standby从JN中读取数据，即使两者之间的Socket连接有网络波动，一旦网络恢复，Standby继续从JN中读取数据，最终实现数据一致性。JN中有一个过半机制，在Active往JN群写数据时，只要过半的JN写入成功，Standby从过半JN的任意一个读取到了修改的数据，Standby就可以顺利同步全部数据。

> NFS（Network File System）：是一种基于TCP/IP传输的网络文件系统协议。通过该协议，客户机可以像访问本地目录一样访问远程服务器中的共享资源。依赖于RPC来实现网络文件系统共享。

> RPC（Remote Procedure Call Protocol）：远程过程调用协议，是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。RPC协议假定某些传输协议的存在，如TCP/UDP，为通信程序之间携带信息数据。目的是调用远程方法像调用本地方法一样。
>
> 采用C/S模式，客户机请求程序调用进程发送一个有进程参数的调用信息到服务进程，然后等待应答信息。在服务器端，进程保持睡眠状态直到调用信息到达为止。当一个调用信息到达，服务器获得进程参数，计算结果，发送答复信息，然后等待下一个调用信息。

> JN(JournalNode)：为了让备用节点保持与活动节点的状态同步，两个节点都与一组名为“JournalNodes”（JN）的独立守护进程通信。当主动节点执行任何命名空间修改时，它会将修改记录持久地记录到这些 JN 中的大多数。Standby 节点能够从 JN 中读取编辑，并不断地观察它们以了解对编辑日志的更改。当备用节点看到编辑时，它将它们应用到自己的命名空间。在发生故障转移的情况下，备用节点将确保它已从 JournalNodes 读取所有编辑，然后再将其提升为 Active 状态。这可确保在发生故障转移之前完全同步命名空间状态。
>
> 为防止数据在两个NameNode之间产生分歧，以及所谓的“脑裂场景”，JN永远只允许一个NameNode一次称为写入者。
>
> （过半机制）必须至少有 3 个 JournalNode 守护进程，因为编辑日志修改必须写入大多数 JN。这将允许系统容忍单台机器的故障。您也可以运行 3 个以上的 JournalNode，但为了实际增加系统可以容忍的故障数量，您应该运行奇数个 JN（即 3、5、7 等）。请注意，当使用 N 个 JournalNode 运行时，系统最多可以容忍 (N - 1) / 2 次故障并继续正常运行。

### 3.2、实现自动HA

基于Zookeeper实现自动化集群高可用，Hadoop实现高可用主要有两种方式，一种是使用共享日志编辑系统(QJM)，另一种是基于网络文件系统(NFS)的高可用方案。基于NFS的高可用方案需要额外安装NFS服务器，而QJM的高可用方案不需要安装额外的服务器。

1. 两台NN启动后都会去zk（zookeeper）进行注册，优先注册的为主节点（Active），另外一个为备节点（Standby），
2. 主NN对外提供服务，备NN同步主NN元数据，以待切换，通过集群JN(JournalNode)，备用NN也会帮助主NN合并editsLog文件和fsimage产生新的fsimage，并推送ActiveNN。

3. ZKFailover Controller(ZKFC，与NN在同一机器上)的作用是监控NameNode健康状态，当主NN挂掉之后，备用NN的ZKFC会得到消息，然后会将备用NN状态改为（Active），并是原来的主NN改为备用NN。

4. DN（datenode)会同时把信息报告给主从NN。

## 五、MapReduce

MapReduce是一个软件框架，用于轻松编写应用程序，这些程序以可靠、容错的方式在大型商用硬件集群（数千个节点）上并行处理大量数据。

可以分成Map和Reduce。

- Map：映射过程，把一组数据按照某种Map函数映射成新的数据。一条数据进入map会被处理成多条数据，就是1进N出。
- Reduce：归纳过程，把若干组映射结果进行汇总进行输出。一组数据进入Reduce会被归纳为一组数据（或者多组数据），也就是一组进N出。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220618201951389.png" alt="image-20220618201951389" width="50%;" />

### 工作流程

**分片、格式化数据源**

输入 Map 阶段的数据源，必须经过分片和格式化操作。

- 分片操作：指的是将源文件划分为大小相等的小数据块( Hadoop 2.x 中默认 128MB )，也就是分片( split )，Hadoop 会为每一个分片构建一个 Map 任务，并由该任务运行自定义的 map() 函数，从而处理分片里的每一条记录。
- 格式化操作：将划分好的分片( split )格式化为键值对<key,value>形式的数据，其中， key 代表偏移量， value 代表每一行内容。

**执行 MapTask**

每个 Map 任务都有一个内存缓冲区(缓冲区大小 100MB )，输入的分片( split )数据经过 Map 任务处理后的中间结果会写入内存缓冲区中。如果写人的数据达到内存缓冲的阈值( 80MB )，会启动一个线程将内存中的溢出数据写入磁盘，同时不影响 Map 中间结果继续写入缓冲区。在溢写过程中， MapReduce 框架会对 key 进行排序，如果中间结果比较大，会形成多个溢写文件，最后的缓冲区数据也会全部溢写入磁盘形成一个溢写文件，如果是多个溢写文件，则最后合并所有的溢写文件为一个文件。

**执行 Shuffle 过程**

MapReduce 工作过程中， Map 阶段处理的数据如何传递给 Reduce 阶段，这是 MapReduce 框架中关键的一个过程，这个过程叫作 Shuffle 。Shuffle 会将 MapTask 输出的处理结果数据分发给 ReduceTask ，并在分发的过程中，对数据按 key 进行分区和排序。

**执行 ReduceTask**

输入 ReduceTask 的数据流是<key, {value list}>形式，用户可以自定义 reduce()方法进行逻辑处理，最终以<key, value>的形式输出。

**写入文件**

MapReduce 框架会自动把 ReduceTask 生成的<key, value>传入 OutputFormat 的 write 方法，实现文件的写入操作。
