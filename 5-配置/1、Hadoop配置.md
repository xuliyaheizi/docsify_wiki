# Hadoop配置

## 一、安装CentOS系统

[CSND教程](https://blog.csdn.net/m0_46413065/article/details/114667174?spm=1001.2014.3001.5501)

## 二、命令配置

**图形界面与命令行界面切换**

```bash
//输入“systemctl get-default” 获得当前系统启动模式（mult-user.target 为命令行）
systemctl set-default multi-user.target     //将图形改为命令行
systemctl set-default graphical.target      //将命令行改成图形
```

**CentOs静态Ip设置**

```bash
//静态IP设置
vim /etc/sysconfig/network-scripts/ifcfg-ens33
//在该文件输入
BOOTPROTO=dhcp修改为static
IPADDR=192.168.10.200(子网IP)
GATEWAY=192.168.10.2(网关)
DNS1=192.168.10.2
//重启网络服务
service network restart

//主机名设置
vim /etc/hostname

systemctl stop NetworkManager 临时关闭
systemctl disable NetworkManager 永久关闭网络管理命令
systemctl start network.service 开启网络服务
```

**配置SFTP**

```bash
//添加用户组
groupadd sftp
//添加用户
useradd -g sftp -s /sbin/nologin -M sftp
//修改sftp用户的密码
passwd sftp
//创建sftp用户的根目录和属主.属组，修改权限（755）
cd /usr
mkdir sftp
chown root:sftp sftp
chmod 755 sftp
//在sftp的目录中创建可写入的目录
cd sftp/
mkdir module
chown sftp:sftp module
//修改/etc/ssh/sshd_config的配置文件
vim /etc/ssh/sshd_config
// 把原来的sshd_config配置文件里的subsystem行注释掉
//再在文件的最后加入
Match Group sftp
ChrootDirectory /usr/sftp
ForceCommand internal-sftp
AllowTcpForwarding no
XllForwarding no
```

**关闭防火墙**

```bash
//取消开机自启动防火墙
systemctl stop firewalld
systemctl disable firewalld.service

systemctl status firewalld
```

**centos7设置时间和时区**

```bash
1、安装ntp服务软件包：yum install ntp

2、将ntp设置为缺省启动：systemctl enable ntpd

3、启动ntp服务：service ntpd restart

4.  将系统时区改为上海时间（即CST时区）：ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

5. 输入date命令查看时间是否正确

```

**安装 epel-release**

```bash
//注：Extra Packages for Enterprise Linux 是为“红帽系”的操作系统提供额外的软件包，适用于 RHEL、CentOS 和 Scientific Linux。相当于是一个软件仓库，大多数 rpm 包在官方 repository 中是找不到的）

yum install -y epel-release
```

**配置JDK**

```bash
//卸载系统jdk
rpm -qa | grep -i java | xargs -n1 rpm -e --nodeps
➢ rpm -qa：查询所安装的所有 rpm 软件包
➢ grep -i：忽略大小写
➢ xargs -n1：表示每次只传递一个参数
➢ rpm -e –nodeps：强制卸载软件

//重新安装jdk
//配置jdk环境变量
sudo vim /etc/profile.d/my_env.sh
//添加以下内容
#JAVA_HOME 
export JAVA_HOME=/opt/module/jdk1.8.0_212 
export PATH=$PATH:$JAVA_HOME/bin 
//让新的环境变量 PATH 生效
source /etc/profile

//出现该内容Permission denied 对jdk内的bin/java加权限
chmod 777 java/jdk/bin/java
```

**配置Hadoop**

```bash
//利用传输工具将Hadoop文件传上去
//将Hadoop添加到环境变量
//打开配置文件
sudo vim /etc/profile.d/my_env.sh
//添加内容
#HADOOP_HOME 
export HADOOP_HOME=/opt/module/hadoop-3.1.3 
export PATH=$PATH:$HADOOP_HOME/bin 
export PATH=$PATH:$HADOOP_HOME/sbin 
//使配置文件生效
source /etc/profile
//测试环境
hadoop version
```

**集群之间免密登录**

```bash
//生成密钥
ssh-keygen -t rsa
//将公钥拷贝到其他集群机器上
ssh-copy-id Node1
ssh-copy-id Node2
ssh-copy-id Node3
...........
//
```

## 三、集群配置

**配置 core-site.xml**

```bash
//配置 core-site.xml
cd $HADOOP_HOME/etc/hadoop
vim core-site.xml
//文件内 输入以下内容
<configuration> 
    <!-- 指定 NameNode 的地址 --> 
    <property> 
        <name>fs.defaultFS</name> 
        <value>hdfs://master:9000</value> 
    </property> 
 
    <!-- 指定 hadoop 数据的存储目录 --> 
    <property> 
        <name>hadoop.tmp.dir</name> 
        <value>/opt/hadoopdata</value> 
    </property> 
 
    <!-- 配置 HDFS 网页登录使用的静态用户为 atguigu --> 
    <property> 
        <name>hadoop.http.staticuser.user</name> 
        <value>atguigu</value> 
    </property> 
</configuration> 
```

**HDFS 配置文件**

```bash
//配置 hdfs-site.xml
vim hdfs-site.xml
//以下内容
<configuration> 
  <!-- NameNode web 端访问地址--> 
  <property> 
        <name>dfs.namenode.http-address</name> 
        <value>master:9870</value> 
    </property> 
  <!-- SecondaryNameNode web 端访问地址--> 
    <property> 
        <name>dfs.namenode.secondary.http-address</name> 
        <value>node4:9871</value> 
    </property> 
</configuration> 
```

**YARN 配置文件**

```bash
//配置 yarn-site.xml
vim yarn-site.xml
//内容如下
<configuration> 
    <!-- 指定 MR 走 shuffle --> 
    <property> 
        <name>yarn.nodemanager.aux-services</name> 
        <value>mapreduce_shuffle</value> 
    </property> 
 
    <!-- 指定 ResourceManager 的地址--> 
    <property> 
        <name>yarn.resourcemanager.hostname</name> 
        <value>node3</value> 
    </property> 
 
    <!-- 环境变量的继承 --> 
    <property> 
        <name>yarn.nodemanager.env-whitelist</name> 
        
<value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value> 
    </property> 
</configuration> 

```

**MapReduce 配置文件**

```bash
//配置 mapred-site.xml
vim mapred-site.xml
//内容
<configuration> 
  <!-- 指定 MapReduce 程序运行在 Yarn 上 --> 
    <property> 
        <name>mapreduce.framework.name</name> 
        <value>yarn</value> 
    </property> 
</configuration> 
```

**启动**

```bash
//初始化（注意：只有第一次的时候才需要）
hdfs namenode -format

//主节点启动 HDFS
sbin/start-dfs.sh

//关闭
stop-all.sh

// 在配置了 ResourceManager 的节点启动yarn
start-yarn.sh

//查看
master:9870  //namenode
node3:8088   //resourceManager
```

**集群测试**

```bash
//上传文件到集群
hadoop fs -mkdir /input
//上传大文件
hadoop fs -put /opt/software/jdk-8u212-linux-x64.tar.gz /
//执行 wordcount 程序
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount /input /output
//删除文件
hadoop fs -rm -r /output
```

**配置历史服务器**

```bash
//配置 mapred-site.xml
vim mapred-site.xml
//输入以下内容
<!-- 历史服务器端地址 --> 
<property> 
    <name>mapreduce.jobhistory.address</name> 
    <value>node2:10020</value> 
</property> 
 
<!-- 历史服务器 web 端地址 --> 
<property> 
    <name>mapreduce.jobhistory.webapp.address</name> 
    <value>node2:19888</value> 
</property> 

//启动历史服务器
mapred --daemon start historyserver
```

**配置日志聚集**

```bash
//配置 yarn-site.xml
vim yarn-site.xml
//输入以下内容
<!-- 开启日志聚集功能 --> 
<property> 
    <name>yarn.log-aggregation-enable</name> 
    <value>true</value> 
</property> 
<!-- 设置日志聚集服务器地址 --> 
<property>   
    <name>yarn.log.server.url</name>   
    <value>http://node2:19888/jobhistory/logs</value> 
</property> 
<!-- 设置日志保留时间为 7 天 --> 
<property> 
    <name>yarn.log-aggregation.retain-seconds</name> 
    <value>604800</value> 
</property> 

//分发配置
xsync $HADOOP_HOME/etc/hadoop/yarn-site.xml
//重启
mapred --daemon stop historyserver
sbin/stop-yarn.sh

```

## 四、配置错误

```bash
//HDFS格式化后启动dfs出现以下错误：
[root@master sbin]# ./start-dfs.sh
Starting namenodes on [master]
ERROR: Attempting to operate on hdfs namenode as root
ERROR: but there is no HDFS_NAMENODE_USER defined. Aborting operation.
Starting datanodes
ERROR: Attempting to operate on hdfs datanode as root
ERROR: but there is no HDFS_DATANODE_USER defined. Aborting operation.
Starting secondary namenodes [slave1]
ERROR: Attempting to operate on hdfs secondarynamenode as root
ERROR: but there is no HDFS_SECONDARYNAMENODE_USER defined. Aborting operation.

//将start-dfs.sh，stop-dfs.sh两个文件顶部添加以下参数
#!/usr/bin/env bash
HDFS_DATANODE_USER=root
HADOOP_SECURE_DN_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root

//start-yarn.sh，stop-yarn.sh顶部也需添加以下：
#!/usr/bin/env bash
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
```

```bash
//出现错误
master: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
//问题原因：出现这问题是因为免密登录配置上的缺陷导致，所以只要解决免密问题即可。
ssh  localhost  -- 也会让你输入密码的，可以使用这个命令测试一下，如果还是让输入密码那就是本机的免密配置没有配置。
//处理方法
 cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys
//这时候再尝试下面的命令测试，发现不会再是密码，说明配置成功。
ssh localhost 

```

```bash
//出现 Permission denied
chmod u+x 文件路径
```

```
ssh hadoop1 jps
jps出错
把/etc/profile里面的环境变量追加到~/.bashrc目录
[root@cdh1 opt]# cat /etc/profile >> ~/.bashrc 
[root@cdh2 opt]# cat /etc/profile >> ~/.bashrc 
[root@cdh3 opt]# cat /etc/profile >> ~/.bashrc 
```



## 五、编写集群脚本

### Hadoop 集群启停脚本（包含 HDFS ，Yarn ，Historyserver ）

```bash
//进入用户目录
cd /home/zhulin/bin
vim myhadoop.sh
```

**输入脚本内容**

```bash
#!/bin/bash 
 
if [ $# -lt 1 ] 
then 
    echo "No Args Input..." 
    exit ; 
fi 
 
case $1 in 
"start") 
        echo " =================== 启动 hadoop 集群 ===================" 
 
        echo " --------------- 启动 hdfs ---------------" 
        ssh master "/usr/hadoop/sbin/start-dfs.sh"
        echo " --------------- 启动 yarn ---------------" 
        ssh node3 "/usr/hadoop/sbin/start-yarn.sh" 
        echo " --------------- 启动 historyserver ---------------" 
        ssh node2 "/usr/hadoop/bin/mapred --daemon start historyserver" 
;; 
"stop") 
        echo " =================== 关闭 hadoop 集群 ===================" 
 
        echo " --------------- 关闭 historyserver ---------------" 
        ssh node2 "/usr/hadoop/bin/mapred --daemon stop historyserver" 
        echo " --------------- 关闭 yarn ---------------" 
        ssh node3 "/usr/hadoop/sbin/stop-yarn.sh" 
        echo " --------------- 关闭 hdfs ---------------" 
        ssh master "/usr/hadoop/sbin/sbin/stop-dfs.sh"
        
;; 
*) 
    echo "Input Args Error..." 
;; 
esac 
```

```bash
//然后赋予脚本执行权限
chmod +x myhadoop.sh
//拷贝到系统目录的bin下
sudo cp myhadoop.sh /bin
//脚本关闭集群
myhadoop.sh top
//脚本启动集群
myhadoop.sh start
```

### 查看三台服务器 Java 进程脚本：jpsall

> 由于每次查看进程都得到每台服务器上输入jps查看，比较麻烦，且如果服务器较多，十分耗时，于是想到编写一个脚本，查看所有服务器的进程情况。

```bash
//用户目录 写入脚本文件
cd /home/zhulin/bin
vim jps
```

**脚本内容**

```bash
#!/bin/bash 
 
for host in master node1 node2 node3 node4 
do 
        echo =============== $host =============== 
        ssh $host jps  
done 
```

```bash
//赋予脚本权限
chmod +x jpsall
//拷贝至系统bin目录
sudo cp jpsall /bin
```

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220607174704672.png" alt="image-20220607174704672" width="67%;" />

## 六、使用Docker安装Hadoop

### 获取centos镜像

```
docker search centos    //查找centos镜像
docker pull centos      //拉取centos镜像
docker images           //查看镜像
```

### 安装SSH

以centos7镜像为基础，构建一个带有SSH功能的centos，基于Dockerfile

```
vim Dockerfile           # 以 centos 镜像为基础，安装SSH的相关包，设置了root用户的密码
```

```
//Dockerfile内容
FROM centos:7                       # 基于centos镜像
MAINTAINER  zhulin                  # 创建者信息

# 执行的命令
RUN  yum -y install openssh-server sudo  
RUN  sed -i 's/UsePAM yes/UsePAM no/g'  /etc/ssh/sshd_config
RUN  yum -y install openssh-clients
RUN  yum -y install net-tools
RUN  yum -y install initscripts
RUN  yum -y install ntp
RUN  yum -y install rsync
RUN  ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

RUN echo "root:a"  | chpasswd
RUN echo "root ALL=(ALL)  ALL"  >> /etc/sudoers
RUN ssh-keygen -t dsa  -f /etc/ssh/ssh_host_dsa_key
RUN ssh-keygen -t rsa  -f /etc/ssh/ssh_host_rsa_key

RUN mkdir /var/run/sshd
EXPOSE 22                    # 开放的端口
CMD ["/usr/sbin/sshd","-D"]      # 执行的命令，这里为启动的命令，在/lib/systemd/system/sshd.service 可以查看到相应的启动命令
```

```
//构建镜像
docker build -t mycentos .
```

### 构建Hadoop镜像

```
vim Dockerfile      # 编写Dockerfile
```

```
FROM mycentos
ADD jdk-8u333-linux-x64.tar.gz  /usr/local
RUN mv /usr/local/jdk1.8.0_333   /usr/local/jdk1.8
ENV JAVA_HOME  /usr/local/jdk1.8
ENV PATH $JAVA_HOME/bin:$PATH

ADD hadoop-3.1.3.tar.gz  /usr/local
RUN mv /usr/local/hadoop-3.1.3  /usr/local/hadoop
ENV HADOOP_HOME /usr/local/hadoop
ENV PATH $HADOOP_HOME/bin:$PATH

RUN yum -y install which sudo vim bash-completion
```

```shell
docker network create --subnet=172.168.0.0/24 docker-br0
//启动容器
docker run --restart always --name hdmaster --hostname master --net docker-br0 --ip 172.168.0.10 -d --add-host=node1:172.168.0.11 --add-host=node2:172.168.0.12 --add-host=node3:172.168.0.13 -p 8210:22 -p 8211:8020 -p 8212:8042 -p 8214:9864 -p 8215:50070 -p 8218:8088 -p 9866:9866 --privileged=true zhulins/myhadoop /usr/sbin/init

docker run --restart always --name hdnode1 --hostname node1 --net docker-br0 --ip 172.168.0.11 -d --add-host=master:172.168.0.10 --add-host=node2:172.168.0.12 --add-host=node3:172.168.0.13 -p 8310:22 -p 8311:8020 -p 8312:8042 -p 8314:9864 -p 8315:50070 -p 8318:8088 --privileged=true zhulins/myhadoop /usr/sbin/init

docker run --restart always --name hdnode2 --hostname node2 --net docker-br0 --ip 172.168.0.12 -d --add-host=node1:172.168.0.11 --add-host=master:172.168.0.10 --add-host=node3:172.168.0.13 -p 8410:22 -p 8411:8020 -p 8412:8042 -p 8413:9864 -p 8415:50070 -p 8418:8088 --privileged=true zhulins/myhadoop /usr/sbin/init

docker run --restart always --name hdnode3 --hostname node3 --net docker-br0 --ip 172.168.0.13 -d --add-host=node1:172.168.0.11 --add-host=node2:172.168.0.12 --add-host=master:172.168.0.10 -p 8510:22 -p 8511:8020 -p 8512:8042 -p 8513:9864 -p 8515:50070 -p 8518:8088 --privileged=true zhulins/myhadoop /usr/sbin/init

docker run --restart always --name dknode4 --hostname node4 --net docker-br0 --ip 172.172.0.14 -d --add-host=node1:172.172.0.11 --add-host=node2:172.172.0.12 --add-host=master:172.172.0.10 -p 8510:22 -p 8511:8020 -p 8512:8042 -p 8513:9864 -p 8515:50070 -p 8518:8088 --privileged=true zhulins/myhadoop /usr/sbin/init

```

| 节点名 |     IP     | NameNode:50070 | DataNode | JournalNodes(共享文件系统) |         ZK          |               ZKFC               | RM:8088 | historyserver |
| :----: | :--------: | :------------: | :------: | :------------------------: | :-----------------: | :------------------------------: | :-----: | :-----------: |
| master | 172.17.0.2 |       1        |          |                            |          1          |                1                 |         |               |
| node1  | 172.17.0.3 |       1        |    1     |             1              |          1          |                1                 |         |       1       |
| node2  | 172.17.0.4 |                |    1     |             1              |          1          |                                  |    1    |               |
| node3  | 172.17.0.5 |                |    1     |             1              |                     |                                  |    1    |               |
|        |            |    API:8020    |          |       轻量级,奇数个        |       奇数个        | zookeeper的 fail over controller |         |               |
|        |            |                |          |     用于存editLog日志      | 机器的状态,接收心跳 |            zk的客户端            |         |               |

### 配置环境变量

```
vim /etc/profile

#JAVA_HOME 
export JAVA_HOME=/usr/local/jdk1.8
export PATH=$PATH:$JAVA_HOME/bin 

#HADOOP_HOME 
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin 
export PATH=$PATH:$HADOOP_HOME/sbin 

//使配置文件生效
source /etc/profile

cat /etc/profile >> ~/.bashrc 
```

###修改hosts文件

```
vi /etc/hosts    # 在每个容器修改/etc/hosts配置文件
172.17.0.5 hadoop0
172.17.0.6 hadoop1
172.17.0.7 hadoop2
172.17.0.8 hadoop3
```

### 免密登录

```
ssh-keygen        # 在每台主机都执行该操作
for i in master node{1..3}; do ssh-copy-id root@$i; done    
# 将公钥传给包括自己的每台主机，三个容器都要做！！！确保最终每台主机都能免密访问其他主机包括自己
```

### 安装配置Hadoop

```
//修改 hadoop-env.sh
vim etc/hadoop/hadoop-env.sh
修改   export JAVA_HOME=/usr/local/jdk1.8

//将start-dfs.sh，stop-dfs.sh(在hadoop安装目录的sbin里)两个文件顶部添加以下参数
HDFS_DATANODE_USER=root
HADOOP_SECURE_DN_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root

//将start-yarn.sh，stop-yarn.sh(在hadoop安装目录的sbin里)两个文件顶部添加以下参数
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
```

### 配置文件

```
//修改配置文件core-site.xml
vim etc/hadoop/core-site.xml 
在 <configuration> 块儿中添加：
    <!-- 指定 NameNode 的地址 --> 
    <property> 
        <name>fs.defaultFS</name> 
        <value>hdfs://hadoop0:9000</value> 
    </property> 
 
    <!-- 指定 hadoop 数据的存储目录 --> 
    <property> 
        <name>hadoop.tmp.dir</name> 
        <value>/opt/hadoopdata</value> 
    </property> 
     
//修改配置文件hdfs-site.xml 
vim etc/hadoop/hdfs-site.xml 
在 <configuration> 块儿中添加：
<configuration> 
  <!-- NameNode web 端访问地址--> 
  <property> 
        <name>dfs.namenode.http-address</name> 
        <value>hadoop0:50070</value> 
    </property> 
  <!-- SecondaryNameNode web 端访问地址--> 
    <property> 
        <name>dfs.namenode.secondary.http-address</name> 
        <value>hadoop3:9871</value> 
    </property>
    <property>
      <name>dfs.webhdfs.enabled</name>
      <value>true</value>
    </property>
</configuration> 


//配置 yarn-site.xml
vim etc/hadoop/yarn-site.xml
//内容如下
<configuration> 
    <!-- 指定 MR 走 shuffle --> 
    <property> 
        <name>yarn.nodemanager.aux-services</name> 
        <value>mapreduce_shuffle</value> 
    </property> 
 
    <!-- 指定 ResourceManager 的地址--> 
    <property> 
        <name>yarn.resourcemanager.hostname</name> 
        <value>hadoop0</value> 
    </property> 
 
    <!-- 环境变量的继承 --> 
    <property> 
        <name>yarn.nodemanager.env-whitelist</name>
 <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value> 
    </property> 
</configuration> 

//配置 mapred-site.xml
vim etc/hadoop/mapred-site.xml
//内容
<configuration> 
  <!-- 指定 MapReduce 程序运行在 Yarn 上 --> 
    <property> 
        <name>mapreduce.framework.name</name> 
        <value>yarn</value> 
    </property> 
</configuration> 
```

### 配置历史服务器与日志聚集

```
//配置 mapred-site.xml  历史服务器
vim etc/hadoop/mapred-site.xml
//输入以下内容
<!-- 历史服务器端地址 --> 
<property> 
    <name>mapreduce.jobhistory.address</name> 
    <value>hadoop1:10020</value> 
</property> 
 
<!-- 历史服务器 web 端地址 --> 
<property> 
    <name>mapreduce.jobhistory.webapp.address</name> 
    <value>hadoop1:19888</value> 
</property> 

//配置 yarn-site.xml  日志聚集
vim etc/hadoop/yarn-site.xml
//输入以下内容
<!-- 开启日志聚集功能 --> 
<property> 
    <name>yarn.log-aggregation-enable</name> 
    <value>true</value> 
</property> 
<!-- 设置日志聚集服务器地址 --> 
<property>   
    <name>yarn.log.server.url</name>   
    <value>http://zhulinz.top:8100/jobhistory/logs</value> 
</property> 
<!-- 设置日志保留时间为 7 天 --> 
<property> 
    <name>yarn.log-aggregation.retain-seconds</name> 
    <value>604800</value> 
</property> 
```



###编写脚本

**文件分发脚本**

```
//进入用户目录
cd /home/zhulin/bin
vim xsync

#!/bin/bash 
 
#1. 判断参数个数 
# 判断参数是否小于1
if [ $# -lt 1 ]   
then 
    echo Not Enough Arguement! 
    exit; 
fi 

#2. 遍历集群所有机器 
for host in hadoop0 hadoop1 hadoop2 hadoop3 
do 
   echo ====================  $host  ==================== 
   #3. 遍历所有目录，挨个发送 
   for file in $@ 
   do 

        #4. 判断文件是否存在 
        if [ -e $file ] 
            then 
                #5. 获取父目录 
                pdir=$(cd -P $(dirname $file); pwd) 
                
                #6. 获取当前文件的名称 
                fname=$(basename $file) 
                ssh $host "mkdir -p $pdir" 
                rsync -av $pdir/$fname $host:$pdir 
            # 如果不存在
            else 
                echo $file does not exists! 
        fi 
    done 
done

//然后赋予脚本执行权限
chmod +x xsync
//拷贝到系统目录的bin下
sudo cp xsync /bin
```

**启动脚本**

```
//进入用户目录
cd /home/zhulin/bin
vim myhadoop.sh

#!/bin/bash 
 
if [ $# -lt 1 ] 
then 
    echo "No Args Input..." 
    exit ; 
fi 
 
case $1 in 
"start") 
        echo " =================== 启动 hadoop 集群 ===================" 
 
        echo " --------------- 启动 hdfs ---------------" 
        ssh hadoop0 "/usr/local/hadoop/sbin/start-dfs.sh"
        echo " --------------- 启动 yarn ---------------" 
        ssh hadoop0 "/usr/local/hadoop/sbin/start-yarn.sh" 
        echo " --------------- 启动 historyserver ---------------" 
        ssh hadoop1 "/usr/local/hadoop/bin/mapred --daemon start historyserver" 
;; 
"stop") 
        echo " =================== 关闭 hadoop 集群 ===================" 
 
        echo " --------------- 关闭 historyserver ---------------" 
        ssh hadoop1 "/usr/local/hadoop/bin/mapred --daemon stop historyserver" 
        echo " --------------- 关闭 yarn ---------------" 
        ssh hadoop0 "/usr/local/hadoop/sbin/stop-yarn.sh" 
        echo " --------------- 关闭 hdfs ---------------" 
        ssh hadoop0 "/usr/local/hadoop/sbin/stop-dfs.sh"
        
;; 
*) 
    echo "Input Args Error..." 
;; 
esac 

//然后赋予脚本执行权限
chmod +x myhadoop.sh
//拷贝到系统目录的bin下
sudo cp myhadoop.sh /bin
//脚本关闭集群
myhadoop.sh top
//脚本启动集群
myhadoop.sh start
```

**查看进程脚本**

```
//用户目录 写入脚本文件
cd /home/zhulin/bin
vim jpsall

#!/bin/bash  

for host in hadoop0 hadoop1 hadoop2 hadoop3 
do         
echo =============== $host ===============         
ssh $host jps  
done 

//赋予脚本权限
chmod +x jpsall
//拷贝至系统bin目录
sudo cp jpsall /bin
```

### 集群启动

```
//初始化（注意：只有第一次的时候才需要）
hdfs namenode -format

//脚本启动
myhadoop.sh start
/usr/local/hadoop/sbin/hadoop-daemon.sh start namenode
/usr/local/hadoop/sbin/hadoop-daemon.sh start datanode
/usr/local/hadoop/sbin/yarn-daemon.sh start resourcemanager
/usr/local/hadoop/sbin/yarn-daemon.sh start nodemanager

/usr/local/hadoop/etc/hadoop/sbin/yarn-daemon.sh start resourcemanager
```

