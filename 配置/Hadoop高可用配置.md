# HA配置

## 一、HDFS高可用配置

### 1.1、安装Zookeeper

```shell
#为master、node1、node2节点安装Zookeeper
#上传zookeeper并配置环境变量
vim /etc/profile
#zookeeper
export ZK_HOME=/usr/zk336
export PATH=$PATH:$ZK_HOME/bin
#在zookeeper配置文件下配置zoo.cfg  注：zoo.cfg不应该有中文注释，实际请删除中文注释
vim /usr/zk336/conf/zoo.cfg
    tickTime=2000   #zk工作的基本单位时间  ms
    dataDir=/opt/zookeeper
    clientPort=2181
    initLimit=5    
    #配置初始化时间(leader被选举出来的时候)连接时，leader和follower最小的心跳时间(连接时间).如果超过这个时间，有一半以上的follower与leader已连接，此时leader被选出来了。 5*2000ms
    syncLimit=2   #配置leader与follower之间请求和应答的最长时间 2*2000
    server.1=node1:2888:3888    #server.X=A:B:C    X代表server序号   A代表主机名    B:代表follower与leader通信端口    c:代表选举leader端口
    server.2=node2:2888:3888
    server.3=node3:2888:3888
#在配置文件中的dataDir文件中创建myid文件写入编号
vim /opt/zookeeper/myid

#启动zookeeper
zkServer.sh start
#测试zookeeper客户端
zkCli.sh -server node1:2181
```

### 1.2、修改配置文件

**修改core-site.xml配置**

```xml
<!--yc为服务名，因为HA机制，这个名字既不能是node1，也不能是node2-->
<property>
  <name>fs.defaultFS</name>
  <value>hdfs://yc</value>
</property>
<!--保证每台服务器/opt/hadoopdata目录为空,这个目录将来是namenode, datanode, journalnode等存数据的公共目录-->
<property>
   <name>hadoop.tmp.dir</name>
   <value>/opt/hadoopdata</value>
</property>

<property>
   <name>ha.zookeeper.quorum</name>
   <value>master:2181,node1:2181,node2:2181</value>
</property>
```

**修改hdfs-site.xml配置**

```xml
<!--权限配置， 可以控制各用户之间的权限，这里先关掉--> 
<property>
   <name>dfs.permissions</name>
   <value>false</value> 
</property>
<property>
   <name>dfs.permissions.enabled</name>
   <value>false</value> 
</property>
<!--dfs.nameservices - the logical name for this new nameservice 服务名，与前面core-site.xml中一样-->
<property>
  <name>dfs.nameservices</name>
  <value>yc</value>
</property>
<!--dfs.ha.namenodes.[nameservice ID] - unique identifiers for each NameNode in the nameservice ,两个nn的逻辑名-->
<property>
  <name>dfs.ha.namenodes.yc</name>
  <value>nn1,nn2</value>
</property>
<!--注意: 这是client访问HDFS的RPC请求的端口（以前的是9000,注意修改eclipse插件配置)-->
<property>
  <name>dfs.namenode.rpc-address.yc.nn1</name>
  <value>master:8020</value>
</property>
<property>
  <name>dfs.namenode.rpc-address.yc.nn2</name>
  <value>node1:8020</value>
</property>
<!--浏览器端地址-->
<property>
  <name>dfs.namenode.http-address.yc.nn1</name>
  <value>master:50070</value>
</property>
<property>
  <name>dfs.namenode.http-address.yc.nn2</name>
  <value>node1:50070</value>
</property>
<!--指定jn,   它是hadoop自带的共享存储系统，主要用于两个namenode间数据的共享和同步，即指定 yc下的两个nn共享edits文件目录时，使用jn集群信息.--> 
<property>
  <name>dfs.namenode.shared.edits.dir</name>
  <value>qjournal://node2:8485;node3:8485;node4:8485/yc</value>
</property>
<!--自动故障迁移负责执行的类-->
<property>
  <name>dfs.client.failover.proxy.provider.yc</name>
  <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
</property>
<!--需要nn切换时，使用sshfence方式，  所以要配置免密钥-->
<property>
  <name>dfs.ha.fencing.methods</name>
  <value>sshfence</value>
</property>
<property>
  <name>dfs.ha.fencing.ssh.private-key-files</name>
  <value>/root/.ssh/id_rsa</value>     //配置私钥位置
</property>
<!--指定jn集群对nn的目录进行共享时，自己存储数据的磁盘路径。  它会生成一个journal目录到此位置-->
<property>
  <name>dfs.journalnode.edits.dir</name>
  <value>/opt/journal/node/local/data</value>
</property>

<property>
   <name>dfs.ha.automatic-failover.enabled</name>
   <value>true</value>
</property>
```

### 1.3、格式化集群

```shell
#格式化之前必须删除Hadoopdata（数据缓存）的文件夹
#启动jn，node1、node2、node3
hdfs --daemon start journalnode
#在一台namenode中执行格式化命令
hdfs namenode -format
#在另一台namenode同步数据之前要启动已经格式了的namenode
hdfs --daemon start namenode
#在没有格式的namenode上执行格式化
hdfs namenode -bootstrapStandby
#启动其他服务
start-dfs.sh
```

## 二、YARN高可用配置

**修改yarn-site.xml配置**

```xml
<property>
   <name>yarn.resourcemanager.ha.enabled</name>
   <value>true</value>
 </property>
 <property>
   <name>yarn.resourcemanager.cluster-id</name>
   <value>yc2yarn</value>
 </property>
 <property>
   <name>yarn.resourcemanager.ha.rm-ids</name>
   <value>rm1,rm2</value>
 </property>
 <property>
   <name>yarn.resourcemanager.hostname.rm1</name>
   <value>node2</value>
 </property>
 <property>
   <name>yarn.resourcemanager.hostname.rm2</name>
   <value>node3</value>
 </property>
 <property>
   <name>yarn.resourcemanager.zk-address</name>
   <value>master:2181,node1:2181,node2:2181</value>
 </property>
```



| 节点名 |       IP       | NameNode:50070 | DataNode | JournalNodes(共享文件系统) |         ZK          |               ZKFC               | RM:8088 | historyserver |
| :----: | :------------: | :------------: | :------: | :------------------------: | :-----------------: | :------------------------------: | :-----: | :-----------: |
| master | 192.168.10.200 |       1        |          |                            |          1          |                1                 |         |               |
| node1  | 192.168.10.201 |       1        |    1     |             1              |          1          |                1                 |         |       1       |
| node2  | 192.168.10.202 |                |    1     |             1              |          1          |                                  |    1    |               |
| node3  | 192.168.76.203 |                |    1     |             1              |                     |                                  |    1    |               |
|        |                |    API:8020    |          |       轻量级,奇数个        |       奇数个        | zookeeper的 fail over controller |         |               |
|        |                |                |          |     用于存editLog日志      | 机器的状态,接收心跳 |            zk的客户端            |         |               |