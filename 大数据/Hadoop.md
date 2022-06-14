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

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220608200350845.png" alt="image-20220608200350845" width="67%;" />

### 1.4、HDFS架构概述

Hadoop Distributed File System，简称HDFS，是一个分布式文件系统。

1. `NameNode（nn）`：存储文件的元数据，如文件名，文件目录结构，文件属性（生成时间、副本数、文件权限），以及每个文件的块列表和块所在的DataNode等。
2. `DataNode(dn)`：在本地文件系统存储文件块数据，以及块数据的校验和。
3. `Secondary NameNode(2nn)`：每隔一段时间对NameNode元数据备份。

### 1.5、YARN架构概述

Yet Another Resource Negotiator 简称YARN ，另一种资源协调者，是Hadoop 的资源管理器。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220608200758870.png" alt="image-20220608200758870" width="67%;" />

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

HDFS是一个分布式文件系统，用于存储文件，通过目录树来定位文件，

### 2.1、优点

- 高容错性：数据自动保存多个副本，通过增加副本的形式，来提高容错性。某个副本丢失后，可以自动恢复。
- 适合处理大数据：数据规模能达到GB、TB、甚至PB级别。文件规模能够处理百万规模以上的文件数量。
- 可构建在廉价机器上：通过多副本机制，提高可靠性。

### 2.2、缺点

- 不适合低延时数据访问：无法访问毫秒级的存储数据。
- 无法高效的对大量小文件进行存储：存储大量的小文件会占用NameNode大量的内存存储文件目录和块信息。小文件的寻址时间会超过读取时间，违背了HDFS的设计目标。
- 不支持并发写入、文件随机修改：一个文件只能一个写，不允许多个线程同时写。仅支持数据追加，不支持文件的随机修改。

### 2.3、组成架构

- NameNode：管理HDFS的名称空间，配置副本策略，管理数据块（BLOCK）映射信息，处理客户端读写请求。
- DataNode：NameNode下达命令后，DataNode执行实际的操作，存储实际的数据块，执行数据块的读/写操作。
- Client：客户端，文件切分，文件上传HDFS的时候，客户端将文件切分成一个一个的数据块，然后进行上传。与NameNode交互，获取文件的位置信息。与DataNode交互，读取或写入数据。命令管理或访问HDFS。
- Secondary NameNode：辅助NameNode，分担其工作量，定期合并Fsimage和Edits，并推送给NameNode。紧急情况下可辅助恢复NameNode。当NameNode挂掉的时候，并不能立刻替换NameNode并提供服务。

### 2.4、文件块大小

HDFS中的文件在物理上是分块存储的，块的大小可以通过配置参数来规定。

如果寻址时间约为10ms，即查找到目标block的时间为10ms。

寻址时间为传输时间的1%时，则为最佳状态。因此传输时间=10ms/0.01=1000ms=1s

HDFS的块设置太小，会增加寻址时间，若太大，从磁盘传输数据的时间会明显大于定位这个块开始位置所需的时间。

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

## 三、集群高可用性

Hadoop1.X中每个集群只有一个NaemNode，使得HDFS中存在单点故障，难以应用在线上场景。NameNode压力过大，内存受限，影响扩展性。

- 针对单点故障的修复方案：HDFS HA高可用，通过主备2个Namenode（3.X上最多5个备，官方推荐3个备。NN太多导致很多数据发送，造成网络压力），主提供服务，备不提供，但是运行着。如果主Namenode发生故障，切换到备Namenode上。
- 解决内存受限问题：HDFS Federation（联邦），水平扩展，**支持多个Namenode**；同时对外提供服务，分治；
  每个Namenode分管一部分目录，所有的Namenode共享所有DataNode存储资源。

### 实现手动HA

![image-20220613210414833](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220613210414833.png)

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

| 节点名 |       IP       | NameNode | DataNode | JournalNodes (共享文件系统) |         ZK          |               ZKFC               |
| :----: | :------------: | :------: | :------: | :-------------------------: | :-----------------: | :------------------------------: |
| master | 192.168.10.200 |    1     |          |                             |          1          |                1                 |
| node1  | 192.168.10.201 |    1     |    1     |              1              |          1          |                1                 |
| node2  | 192.168.10.202 |          |    1     |              1              |          1          |                                  |
| node3  | 192.168.76.203 |          |    1     |              1              |                     |                                  |
|        |                |          |          |        轻量级,奇数个        |       奇数个        | zookeeper的 fail over controller |
|        |                |          |          |      用于存editLog日志      | 机器的状态,接收心跳 |            zk的客户端            |

```shell
#启动
myzk.sh start	#启动zookeeper集群
myjn.sh start	#启动jn
HDFS_JOURNALNODE_USER
HDFS_ZKFC_USER
```

