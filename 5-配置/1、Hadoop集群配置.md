# Hadoop集群配置

**集群架构图**

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207041034924.png" alt="image-20220704103412317"/>

## 一、CentOs设置

### 1.1、界面切换命令

```shell
#输入“systemctl get-default” 获得当前系统启动模式（mult-user.target 为命令行）
$ systemctl set-default multi-user.target     #将图形改为命令行
$ systemctl set-default graphical.target      #将命令行改成图形
```

### 1.2、静态IP设置

```shell
#静态IP设置
$ vim /etc/sysconfig/network-scripts/ifcfg-ens33

#在该文件输入
    BOOTPROTO=dhcp修改为static
    IPADDR=192.168.10.200(子网IP)
    GATEWAY=192.168.10.2(网关)
    DNS1=192.168.10.2

#重启网络服务
$ service network restart

#主机名设置
$ vim /etc/hostname

$ systemctl stop NetworkManager     #临时关闭
$ systemctl disable NetworkManager  #永久关闭网络管理命令
$systemctl start network.service    #开启网络服务
```

### 1.3、配置SFTP

```shell
#添加用户组
$ groupadd sftp

#添加用户
$ useradd -g sftp -s /sbin/nologin -M sftp

#修改sftp用户的密码
$ passwd sftp

#创建sftp用户的根目录和属主.属组，修改权限（755）
$ cd /usr
$ mkdir sftp
$ chown root:sftp sftp
$ chmod 755 sftp

#在sftp的目录中创建可写入的目录
$ cd sftp/
$ mkdir module
$ chown sftp:sftp module

#修改/etc/ssh/sshd_config的配置文件
$ vim /etc/ssh/sshd_config

#把原来的sshd_config配置文件里的subsystem行注释掉，再在文件的最后加入
    Match Group sftp
    ChrootDirectory /usr/sftp
    ForceCommand internal-sftp
    AllowTcpForwarding no
    XllForwarding no
```

### 1.4、系统防火墙设置

```shell
#查看系统防火墙状态
$ systemctl status firewalld

#取消开机自启动防火墙
$ systemctl stop firewalld
$ systemctl disable firewalld.service

#启动防火墙，设置开机自启防火墙
$ systemctl start firewalld
$ systemctl enable firewalld.service
```

### 1.5、设置时间和时区

```shell
1、安装ntp服务软件包：yum install ntp
2、将ntp设置为缺省启动：systemctl enable ntpd
3、启动ntp服务：service ntpd restart
4.  将系统时区改为上海时间（即CST时区）：ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
5. 输入date命令查看时间是否正确
```

### 1.6、配置JDK

```shell
#注：Extra Packages for Enterprise Linux 是为“红帽系”的操作系统提供额外的软件包，适用于 RHEL、CentOS 和 Scientific Linux。相当于是一个软件仓库，大多数 rpm 包在官方 repository 中是找不到的）
$ yum install -y epel-release

#卸载系统jdk
$ rpm -qa | grep -i java | xargs -n1 rpm -e --nodeps
➢ rpm -qa：查询所安装的所有 rpm 软件包
➢ grep -i：忽略大小写
➢ xargs -n1：表示每次只传递一个参数
➢ rpm -e –nodeps：强制卸载软件

#重新安装jdk，配置jdk环境变量
$ sudo vim /etc/profile.d/my_env.sh

#添加以下内容
    #JAVA_HOME 
    export JAVA_HOME=/opt/module/jdk1.8.0_212 
    export PATH=$PATH:$JAVA_HOME/bin 
    
#让新的环境变量 PATH 生效
$ source /etc/profile

#出现该内容Permission denied 对jdk内的bin/java加权限
$ chmod 777 java/jdk/bin/java
```

### 1.7、集群免密配置

```shell
#在每台主机都执行该操作
$ ssh-keygen

#将公钥传给包括自己的每台主机，三个容器都要做！！！确保最终每台主机都能免密访问其他主机包括自己
$ for i in master node{1..3}; do ssh-copy-id root@$i; done    
```

## 二、Hadoop安装、配置

### 2.1、Hadoop安装

```shell
#Hadoop下载网站
https://archive.apache.org/dist/hadoop/common/
#解压上传至服务器目录 /usr/local/ 下

#配置环境变量
$ vim /etc/profile

#添加内容
    #HADOOP_HOME 
    export HADOOP_HOME=/usr/local/hadoop
    export PATH=$PATH:$HADOOP_HOME/bin
    export PATH=$PATH:$HADOOP_HOME/sbin
    
#使配置文件生效
$ source /etc/profile
#测试
$ hadoop version
    -Hadoop 3.1.3
    -Source code repository https://gitbox.apache.org/repos/asf/hadoop.git -r ba631c436b806728f8ec2f54ab1e289526c90579
    -Compiled by ztang on 2019-09-12T02:47Z
    -Compiled with protoc 2.5.0
    -From source with checksum ec785077c385118ac91aadde5ec9799
    -This command was run using /usr/local/hadoop/share/hadoop/common/hadoop-common-3.1.3.jar
```

### 2.2、单NN与单RM的配置

#### hadoop-env.sh

```shell
# Technically, the only required environment variable is JAVA_HOME.
# All others are optional.  However, the defaults are probably not
# preferred.  Many sites configure these options outside of Hadoop,
# such as in /etc/profile.d

# The java implementation to use. By default, this environment
# variable is REQUIRED on ALL platforms except OS X!
export JAVA_HOME=/usr/local/jdk1.8   #修改这一句

#文件底部添加
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
export HDFS_ZKFC_USER=root
export HDFS_JOURNALNODE_USER=root
```

#### core-site.xml

```shell
#配置 core-site.xml
$ cd $HADOOP_HOME/etc/hadoop
$ vim core-site.xml

#文件内 输入以下内容
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
    </configuration>
```

#### hdfs-site.xml

```shell
#配置 hdfs-site.xml
$ vim hdfs-site.xml

#以下内容
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

#### yarn-site.xml

```shell
#配置 yarn-site.xml
$ vim yarn-site.xml

#内容如下
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
 			<value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,
 			HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value> 
        </property> 
    </configuration> 
```

#### mapred-site.xml

```shell
#配置 mapred-site.xml
$ vim mapred-site.xml

    <configuration> 
      <!-- 指定 MapReduce 程序运行在 Yarn 上 --> 
        <property> 
            <name>mapreduce.framework.name</name> 
            <value>yarn</value> 
        </property> 
    </configuration> 
```

### 2.3、历史服务器与日志聚集的配置

```shell
#配置 mapred-site.xml
$ vim mapred-site.xml

#输入以下内容
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

#启动历史服务器
$ mapred --daemon start historyserver
```

```shell
#配置 yarn-site.xml
$ vim yarn-site.xml

#输入以下内容
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

#分发配置
$ xsync $HADOOP_HOME/etc/hadoop/yarn-site.xml

#重启
$ mapred --daemon stop historyserver
$ sbin/stop-yarn.sh
```

### 2.4、HA高可用配置

hadoop-env.sh文件与单节点配置相同。

#### core-site.xml

```xml
<configuration>
    <!-- 指定 NameNode 的地址 yc-服务名 配置高可用机制 不能指定主节点 --> 
    <property> 
        <name>fs.defaultFS</name> 
        <value>hdfs://yc</value> 
    </property> 
    <!-- 指定 hadoop 数据的存储目录 --> 
    <property> 
        <name>hadoop.tmp.dir</name>
        <value>/opt/hadoopdata</value> 
    </property> 
    <property>
        <name>ha.zookeeper.quorum</name>
        <value>master:2181,node1:2181,node2:2181</value>
    </property>
    <!--启用机架感知-->
    <property>
        <name>net.topology.script.file.name</name>
        <value>/home/zhulin/bin/rackaware.sh</value>
    </property>
</configuration>
```

#### hdfs-site.xml

```xml
<configuration>
    <property>
        <name>dfs.ha.automatic-failover.enabled</name>
        <value>true</value>
    </property>
    <!--权限配置， 可以控制各用户之间的权限-->
    <property>
        <name>dfs.permissions</name>
        <value>false</value> 
    </property>
    <property>
        <name>dfs.permissions.enabled</name>
        <value>false</value> 
    </property>
    <!--服务名，与前面core-site.xml中一样-->
    <property>
        <name>dfs.nameservices</name>
        <value>yc</value>
    </property>
    <!--两个nn主节点的逻辑名-->
    <property>
        <name>dfs.ha.namenodes.yc</name>
        <value>nn1,nn2</value>
    </property>
    <!--这是client访问HDFS的RPC请求的端口-->
    <property>
        <name>dfs.namenode.rpc-address.yc.nn1</name>
        <value>master:8020</value>
    </property>
    <property>
        <name>dfs.namenode.rpc-address.yc.nn2</name>
        <value>node1:8020</value>
    </property>
    <!--web访问端口-->
    <property>
        <name>dfs.namenode.http-address.yc.nn1</name>
        <value>master:50070</value>
    </property>
    <property>
        <name>dfs.namenode.http-address.yc.nn2</name>
        <value>node1:50070</value>
    </property>
    <!--指定jn,   它是hadoop自带的共享存储系统，主要用于两个namenode间数据的共享和同步，即指定 yc下的两个nn共享edits文件目录时，使用jn集群信息. -->
    <property>
        <name>dfs.namenode.shared.edits.dir</name>
        <value>qjournal://node1:8485;node2:8485;node3:8485/yc</value>
    </property>
    <!-- 自动故障迁移负责执行的类-->
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
        <value>/root/.ssh/id_rsa</value>
    </property>
    <!--指定jn集群对nn的目录进行共享时，自己存储数据的磁盘路径。  它会生成一个journal目录到此位置-->
    <property>
        <name>dfs.journalnode.edits.dir</name>
        <value>/opt/journal/node/local/data</value>
    </property>
    <!-- 设置block块大小为128MB-->
    <property>
        <name>dfs.block.size</name>
        <value>134217728</value>   
    </property>
</configuration>
```

#### yarn-site.xml

```xml
<configuration>
    <!-- Site specific YARN configuration properties -->
    <!-- 指定 MR 走 shuffle --> 
    <property> 
        <name>yarn.nodemanager.aux-services</name> 
        <value>mapreduce_shuffle</value> 
    </property> 
    <!--RS高可用配置-->
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
        <name>yarn.resourcemanager.webapp.address.rm1</name>
        <value>node2:8088</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address.rm2</name>
        <value>node3:8088</value>
    </property>
    <property>
        <name>yarn.resourcemanager.zk-address</name>
        <value>master:2181,node1:2181,node2:2181</value>
    </property>

    <!-- 环境变量的继承 --> 
    <property> 
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value> 
    </property>

    <!-- 开启日志聚集功能 --> 
    <property> 
        <name>yarn.log-aggregation-enable</name> 
        <value>true</value> 
    </property> 
    <!-- 设置日志聚集服务器地址 --> 
    <property>   
        <name>yarn.log.server.url</name>   
        <value>http://node1:19888/jobhistory/logs</value> 
    </property>
    <!-- 设置日志保留时间为 7 天 --> 
    <property> 
        <name>yarn.log-aggregation.retain-seconds</name> 
        <value>604800</value> 
    </property>

    <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
    </property>
</configuration>
```

#### mapred-site.xml

```xml
<configuration>
    <!-- 指定 MapReduce 程序运行在 Yarn 上 --> 
    <property> 
        <name>mapreduce.framework.name</name> 
        <value>yarn</value> 
    </property> 
    <!-- 历史服务器端地址 --> 
    <property> 
        <name>mapreduce.jobhistory.address</name> 
        <value>node1:10020</value> 
    </property> 
    <!-- 历史服务器 web 端地址 --> 
    <property> 
        <name>mapreduce.jobhistory.webapp.address</name> 
        <value>node1:19888</value> 
    </property> 
</configuration>
```

### 2.5、机架感知设置

```shell
172.172.0.10 master /dc1/rack1
172.172.0.11 node1 /dc1/rack2
172.172.0.12 node2 /dc1/rack2
172.172.0.13 node3 /dc1/rack3
```

## 三、Docker部署Hadoop集群

### 3.1、获取CentOs镜像

```shell
#查找centos镜像
$ docker search centos   
#拉取centos镜像
$ docker pull centos
#查看镜像
$ docker images        
```

### 3.2、构建具有SSH的镜像

```shell
#Dockerfile内容
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

#构建镜像
$ docker build -t mycentos .
```

### 3.3、构建Hadoop镜像

```shell
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

**容器启动命令**

```shell
#创建网桥
docker network create --subnet=172.168.0.0/24 docker-br0

#启动容器
docker run --restart always --name hdmaster --hostname master --net docker-br0 --ip 172.168.0.10 -d --add-host=node1:172.168.0.11 --add-host=node2:172.168.0.12 --add-host=node3:172.168.0.13 -p 8210:22 -p 8211:8020 -p 8212:8042 -p 8214:9864 -p 8215:50070 -p 8218:8088 -p 9866:9866 --privileged=true zhulins/myhadoop /usr/sbin/init

docker run --restart always --name hdnode1 --hostname node1 --net docker-br0 --ip 172.168.0.11 -d --add-host=master:172.168.0.10 --add-host=node2:172.168.0.12 --add-host=node3:172.168.0.13 -p 8310:22 -p 8311:8020 -p 8312:8042 -p 8314:9864 -p 8315:50070 -p 8318:8088 --privileged=true zhulins/myhadoop /usr/sbin/init

docker run --restart always --name hdnode2 --hostname node2 --net docker-br0 --ip 172.168.0.12 -d --add-host=node1:172.168.0.11 --add-host=master:172.168.0.10 --add-host=node3:172.168.0.13 -p 8410:22 -p 8411:8020 -p 8412:8042 -p 8413:9864 -p 8415:50070 -p 8418:8088 --privileged=true zhulins/myhadoop /usr/sbin/init

docker run --restart always --name hdnode3 --hostname node3 --net docker-br0 --ip 172.168.0.13 -d --add-host=node1:172.168.0.11 --add-host=node2:172.168.0.12 --add-host=master:172.168.0.10 -p 8510:22 -p 8511:8020 -p 8512:8042 -p 8513:9864 -p 8515:50070 -p 8518:8088 --privileged=true zhulins/myhadoop /usr/sbin/init

docker run --restart always --name dknode4 --hostname node4 --net docker-br0 --ip 172.172.0.14 -d --add-host=node1:172.172.0.11 --add-host=node2:172.172.0.12 --add-host=master:172.172.0.10 -p 8510:22 -p 8511:8020 -p 8512:8042 -p 8513:9864 -p 8515:50070 -p 8518:8088 --privileged=true zhulins/myhadoop /usr/sbin/init
```

## 四、Mysql安装

```shell
#下载安装包，将压缩包上传至 /opt/mysql，并解压
$ https://dev.mysql.com/downloads/file/?id=511379
$ tar -xf mysql-8.0.29-1.el7.x86_64.rpm-bundle.tar

#添加用户组
$ groupadd mysql
#添加用户
$ useradd -g mysql mysql
#查看用户信息。
$ id mysql

#卸载MariaDB
$ rpm -qa | grep mariadb #查询是否有该软件
#如果有，就卸载：rpm -e mariadb-libs-这里是你的版本，如果不能卸载（或报错）则采用强制卸载：rpm -e --nodeps mariadb-libs-这里是你的版本

#安装Mysql
$ rpm -ivh *.rpm
#安装过程要按照如下顺序（必须）进行：
    mysql-community-common-8.0.29-1.el7.x86_64.rpm
    mysql-community-client-plugins-8.0.29-1.el7.x86_64.rpm
    mysql-community-libs-8.0.29-1.el7.x86_64.rpm             --（依赖于common）
    mysql-community-client-8.0.29-1.el7.x86_64.rpm          --（依赖于libs）
    mysql-community-icu-data-files-8.0.29-1.el7.x86_64.rpm
    mysql-community-server-8.0.29-1.el7.x86_64.rpm         --（依赖于client、common）
    按照以上顺序进行一个个的安装，脚本如下：
    rpm -ivh mysql-community-server-5.7.16-1.el7.x86_64.rpm 
#期间缺少啥组件，安装啥
#libaio.so.1()(64bit) is needed by mysql-community-embedded-compat-8.0.29-1.el7.x86_64
$ yum -y install libaio
#libnuma.so.1()(64bit) is needed by mysql-community-embedded-compat-8.0.29-1.el7.x86_64
$ yum -y install numactl

#mysql安装软件在/usr/share/mysql目录下
#Mysql数据库创建在/var/lib/mysql目录下

#进入到mysql这个目录中，更改一下权限
$ cd /usr/share/mysql/
$ chown -R mysql:mysql .

#启动mysql
$ service mysqld restart
#登录
    -mysql
    -报错：ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)  需添加权限
    -在/ect/my.cnf 的最后面加上一行：skip-grant-tables
$ service mysqld restart

#初始化数据库
$ sudo mysqld --initialize --user=mysql
#查看临时生成的root用户密码
$ sudo cat /var/log/mysqld.log
#修改密码
$ mysql> ALTER USER USER() IDENTIFIED BY 'Admin2022!';
#查看当前密码规则
$ mysql> SHOW VARIABLES LIKE 'validate_password%';
#调整密码规则
$ mysql> set global validate_password.policy=0;
$ mysql>  set global validate_password.length=1;
#修改简单密码
$ mysql> ALTER USER USER() IDENTIFIED BY 'aaaa';
#登录
$ mysql> mysql -uroot -p

#修改配置，进行远程连接
#登录
$ mysql> mysql -uroot -p
#进入MySQL库
$ mysql> use mysql;
#查询user表
$ mysql> select user, host from user;
#修改user表，把Host表内容修改为%
$ mysql> update user set host="%" where user="root";
```

## 五、Nginx安装

```shell
#安装相关依赖包
$ sudo yum -y install openssl openssl-devel pcre pcre-devel zlib zlib-devel gcc gcc-c++

#将nginx文件上传到 /tmp/nginx
Ngnix下载地址：http://nginx.org/en/download.html

#进入nginx目录下，执行以下命令
$ ./configure   --prefix=/usr/local/nginx
$ make && make install

#启动nginx
#在/usr/local/nginx/sbin目录下执行  
$ ./nginx
#查看启动情况
$ ps -ef |grep nginx
```

## 六、Flume安装

```shell
#下载地址
http://archive.apache.org/dist/flume/
#下载文件解压，上传至服务器目录 /usr/local/ 下

#将flume/conf下的flume-env.sh.template文件修改为flume-env.sh，并配置flume-env.sh文件
$ mv flume-env.sh.template flume-env.sh
$ vim flume-env.sh
	export JAVA_HOME=/usr/java/jdk8
	
#Flume要想将数据输出到HDFS，必须持有Hadoop相关jar包
    commons-configuration-1.6.jar、
    hadoop-auth-2.7.2.jar、
    hadoop-common-2.7.2.jar、
    hadoop-hdfs-2.7.2.jar、
    commons-io-2.4.jar、
    htrace-core-3.1.0-incubating.jar
	#拷贝到/flume190/lib文件夹下。
```

## 七、Zookeeper安装

```shell
#为master、node1、node2节点安装Zookeeper
#上传zookeeper并配置环境变量
$ vim /etc/profile

#zookeeper
    export ZK_HOME=/usr/zk336
    export PATH=$PATH:$ZK_HOME/bin
    
#在zookeeper配置文件下配置zoo.cfg  注：zoo.cfg不应该有中文注释，实际请删除中文注释
$ vim /usr/zk336/conf/zoo.cfg
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
$ vim /opt/zookeeper/myid

#启动zookeeper
$ zkServer.sh start
#测试zookeeper客户端
$ zkCli.sh -server node1:2181
```



## 八、Azkaban安装

### 8.1、上传安装包

将Azkaban Web服务器、Azkaban执行服务器、Azkaban的sql执行脚本及MySQL安装包拷贝到`master`虚拟机/user/local/azkaban目录下

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207041025198.png" alt="image-20220630091249409" width="50%;" />

### 8.2、安装

```shell
#解压路径下的安装包
$ tar -xvf azkaban-executor-server-2.5.0.tar.gz
$ tar -xvf azkaban-sql-script-2.5.0.tar.gz
$ tar -xvf azkaban-web-server-2.5.0.tar.gz

#对解压后的文件改名
$ mv azkaban-web-2.5.0/ web
$ mv azkaban-executor-2.5.0/ executor

#azkaban脚本导入 进入mysql，创建azkaban数据库，并将解压的脚本导入到azkaban数据库
$ mysql -uroot -p
$ mysql> create database azkaban;
$ mysql> use azkaban;
$ mysql> source /usr/local/azkaban/azkaban-2.5.0/create-all-sql-2.5.0.sql
```

### 8.3、生成密钥库

- Keytool是java数据证书的管理工具，使用户能够管理自己的公/私钥对及相关证书。
- -keystore  指定密钥库的名称及位置(产生的各类信息将不在.keystore文件中)
- -genkey   在用户主目录中创建一个默认文件".keystore" 
- -alias  对我们生成的.keystore 进行指认别名；如果没有默认是mykey
- -keyalg 指定密钥的算法 RSA/DSA 默认是DSA

```shell
#生成 keystore的密码及相应信息的密钥库
$ keytool -keystore keystore -alias jetty -genkey -keyalg RSA
What is your first and last name?
  [Unknown]:  zhulin
What is the name of your organizational unit?
  [Unknown]:  hnit
What is the name of your organization?
  [Unknown]:  hnit
What is the name of your City or Locality?
  [Unknown]:  hy
What is the name of your State or Province?
  [Unknown]:  ch
What is the two-letter country code for this unit?
  [Unknown]:  ch
Is CN=zhulin, OU=hnit, O=hnit, L=hy, ST=ch, C=ch correct?
  [no]:  y
Enter key password for <jetty>
	(RETURN if same as keystore password):  
Re-enter new password:
#注意：
#密钥库的密码至少必须6个字符，可以是纯数字或者字母或者数字和字母的组合等等
#密钥库的密码最好和<jetty> 的密钥相同，方便记忆

#将keystore 拷贝到 azkaban web服务器根目录中
$ mv keystore web/
```

### 8.4、时间同步配置

```shell
#先配置好服务器节点上的时区
#如果在/usr/share/zoneinfo/这个目录下不存在时区配置文件Asia/Shanghai，就要用 tzselect 生成。
$ tzselect
Please identify a location so that time zone rules can be set correctly.
Please select a continent or ocean.
 1) Africa
 2) Americas
 3) Antarctica
 4) Arctic Ocean
 5) Asia
 6) Atlantic Ocean
 7) Australia
 8) Europe
 9) Indian Ocean
10) Pacific Ocean
11) none - I want to specify the time zone using the Posix TZ format.
#? 5
Please select a country.
 1) Afghanistan		  18) Israel		    35) Palestine
 2) Armenia		  19) Japan		    36) Philippines
 3) Azerbaijan		  20) Jordan		    37) Qatar
 4) Bahrain		  21) Kazakhstan	    38) Russia
 5) Bangladesh		  22) Korea (North)	    39) Saudi Arabia
 6) Bhutan		  23) Korea (South)	    40) Singapore
 7) Brunei		  24) Kuwait		    41) Sri Lanka
 8) Cambodia		  25) Kyrgyzstan	    42) Syria
 9) China		  26) Laos		    43) Taiwan
10) Cyprus		  27) Lebanon		    44) Tajikistan
11) East Timor		  28) Macau		    45) Thailand
12) Georgia		  29) Malaysia		    46) Turkmenistan
13) Hong Kong		  30) Mongolia		    47) United Arab Emirates
14) India		  31) Myanmar (Burma)	    48) Uzbekistan
15) Indonesia		  32) Nepal		    49) Vietnam
16) Iran		  33) Oman		    50) Yemen
17) Iraq		  34) Pakistan
#? 9
Please select one of the following time zone regions.
1) Beijing Time
2) Xinjiang Time
#? 1

The following information has been given:

	China
	Beijing Time

Therefore TZ='Asia/Shanghai' will be used.
Local time is now:	Thu Jun 30 09:27:16 CST 2022.
Universal Time is now:	Thu Jun 30 01:27:16 UTC 2022.
Is the above information OK?
1) Yes
2) No
#? 1

You can make this change permanent for yourself by appending the line
	TZ='Asia/Shanghai'; export TZ
to the file '.profile' in your home directory; then log out and log in again.

Here is that TZ value again, this time on standard output so that you
can use the /usr/bin/tzselect command in shell scripts:
Asia/Shanghai

#拷贝该时区文件，覆盖系统本地时区配置
$ cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
#集群时间同步（同时发给三个窗口）
$ sudo date -s '2018-10-18 16:39:30'
```

### 8.5、配置文件

#### 8.5.1、Web服务器配置

```shell
#进入azkaban web服务器安装目录 conf目录，打开azkaban.properties文件
$ cd /usr/local/azbakan/web/conf/
$ vim azkaban.properties

#按照如下配置修改azkaban.properties文件。
    ##Azkaban Personalization Settings
    ##服务器UI名称,用于服务器上方显示的名字
    azkaban.name=Test
    ##描述
    azkaban.label=My Local Azkaban
    ##UI颜色
    azkaban.color=#FF3601
    azkaban.default.servlet.path=/index
    ##默认web server存放web文件的目录
    web.resource.dir=/usr/local/azkaban/web/web/
    ##默认时区,已改为亚洲/上海 默认为美国
    default.timezone.id=Asia/Shanghai

    ##Azkaban UserManager class
    user.manager.class=azkaban.user.XmlUserManager
    ##用户权限管理默认类（绝对路径）
    user.manager.xml.file=/usr/local/azkaban/web/conf/azkaban-users.xml

    ##Loader for projects
    ##global配置文件所在位置（绝对路径）
    executor.global.properties=/usr/local/azkaban/executor/conf/global.properties
    azkaban.project.dir=projects

    ##数据库类型
    database.type=mysql
    ##端口号
    mysql.port=3306
    ##数据库连接IP
    mysql.host=node1
    ##数据库实例名
    mysql.database=azkaban?useSSL=false&serverTimezone=UTC&characterEncoding=utf-8
    ##数据库用户名
    mysql.user=root
    ##数据库密码
    mysql.password=aaaa
    ##最大连接数
    mysql.numconnections=100

    ## Velocity dev mode
    velocity.dev.mode=false

    # Azkaban Jetty server properties.
    # Jetty服务器属性.
    #最大线程数
    jetty.maxThreads=25
    #Jetty SSL端口
    jetty.ssl.port=8443
    #Jetty端口
    jetty.port=8081
    #SSL文件名（绝对路径）
    jetty.keystore=/usr/local/azkaban/web/keystore
    #SSL文件密码
    jetty.password=aaaaaa
    #Jetty主密码与keystore文件相同
    jetty.keypassword=aaaaaa
    #SSL文件名（绝对路径）
    jetty.truststore=/usr/local/azkaban/web/keystore
    #SSL文件密码
    jetty.trustpassword=aaaaaa

    # Azkaban Executor settings
    executor.port=12321

    # mail settings
    mail.sender=
    mail.host=
    job.failure.email=
    job.success.email=

    lockdown.create.projects=false

    cache.directory=cache
    
#在azkaban web服务器安装目录 conf目录，按照如下配置修改azkaban-users.xml 文件，增加管理员用户,配置两种角色 admin, metrics。 
$  vim azkaban-users.xml
<azkaban-users>
	<user username="azkaban" password="azkaban" roles="admin" groups="azkaban" />
	<user username="metrics" password="metrics" roles="metrics"/>
	<user username="admin" password="admin" roles="admin,metrics" />
	<role name="admin" permissions="ADMIN" />
	<role name="metrics" permissions="METRICS"/>
</azkaban-users>
```

#### 8.5.2、执行服务器executor配置

```shell
#进入执行服务器executor安装目录conf，打开azkaban.properties
$ cd /usr/local/azkaban/executor/conf
$ vim azkaban.properties

#按照如下配置修改azkaban.properties文件。
    #Azkaban
    #时区
    default.timezone.id=Asia/Shanghai

    # Azkaban JobTypes Plugins
    #jobtype 插件所在位置
    azkaban.jobtype.plugin.dir=plugins/jobtypes

    #Loader for projects
    executor.global.properties=/usr/local/azkaban/executor/conf/global.properties
    azkaban.project.dir=projects

    database.type=mysql
    mysql.port=3306
    mysql.host=node1
    mysql.database=azkaban?useSSL=false&serverTimezone=UTC&characterEncoding=utf-8
    mysql.user=root
    mysql.password=aa
    mysql.numconnections=100

    # Azkaban Executor settings
    #最大线程数
    executor.maxThreads=50
    #端口号(如修改,请与web服务中一致)
    executor.port=12321
    #线程数
    executor.flow.threads=30
```

#### 8.5.3、启动executor服务器

```shell
#在executor服务器目录下执行启动命令
$ cd /usr/local/azkaban/executor/bin
$ azkaban-executor-start.sh
#配置环境变量
$ vim /etc/profile
    #Azkaban
    export AZKABAN_EXECUTRO_HOME=/usr/local/azkaban/executor
    export PATH=$PATH:$AZKABAN_EXECUTRO_HOME/bin

#启动/关闭命令
$ azkaban-executor-start.sh
$ azkaban-executor-shutdown.sh
```

#### 8.5.4、启动web服务器

```shell
#在azkaban web服务器目录下执行启动命令
$ azkaban-web-start.sh 
$ cd /usr/local/azkaban/web/bin
$ azkaban-web-start.sh
#配置环境变量
$ vim /etc/profile
    #Azkaban
    export AZKABAN_WEB_HOME=/usr/local/azkaban/web
    export PATH=$PATH:$AZKABAN_WEB_HOME/bin
```

## 九、GangLia安装（编译方式安装）

#### 服务端监控（安装gweb）

**安装gemtad**

```shell
$ yum -y install apr-devel apr-util check-devel cairo-devel pango-devel libxml2-devel rpm-build glib2-devel dbus-devel freetype-devel fontconfig-devel gcc gcc-c++ expat-devel python-devel libXrender-develS
$ yum install -y libart_lgpl-devel pcre-devel libtool
$ yum install  -y rrdtool rrdtool-devel
$ cd /usr/local
$ mkdir tools
$ cd tools
$ wget http://www.mirrorservice.org/sites/download.savannah.gnu.org/releases/confuse/confuse-2.7.tar.gz
$ tar zxvf confuse-2.7.tar.gz
$ cd confuse-2.7
$ ./configure  --prefix=/usr/local/ganglia-tools/confuse CFLAGS=-fPIC --disable-nls --libdir=/usr/local/ganglia-tools/confuse/lib64
$ make && make install
$ cd /usr/local/tools/
$ wget https://sourceforge.net/projects/ganglia/files/ganglia%20monitoring%20core/3.7.2/ganglia-3.7.2.tar.gz
$ tar zxf ganglia-3.7.2.tar.gz
$ cd ganglia-3.7.2
$ ./configure --prefix=/usr/local/ganglia --enable-gexec --enable-status --with-gmetad --with-libconfuse=/usr/local/ganglia-tools/confuse 
$ make && make install
$ cp gmetad/gmetad.init /etc/init.d/gmetad
$ ln -s /usr/local/ganglia/sbin/gmetad /usr/sbin/gmetad
```

**安装gweb**

```shell
$ yum install httpd httpd-devel php -y
$ yum -y install rsync
$ cd /usr/local/tools
$ wget https://sourceforge.net/projects/ganglia/files/ganglia-web/3.7.2/ganglia-web-3.7.2.tar.gz
$ tar zxvf /usr/tools/ganglia-web-3.7.2.tar.gz -C /var/www/html/
$ cd /var/www/html/
$ mv ganglia-web-3.7.2 ganglia
$ cd /var/www/html/ganglia/
$ useradd -M -s /sbin/nologin www-data
$ make install
$ chown apache:apache -R /var/lib/ganglia-web/
```

**修改配置**

```shell
#修改启动脚本
$ vi /etc/init.d/gmetad
    GMETAD=/usr/sbin/gmetad  #这句话可以自行更改gmetad的命令，当然也能向我们前面做了软连接
    start() {
        [ -f /usr/local/ganglia/etc/gmetad.conf  ] || exit 6  #这里将配置文件改成现在的位置，不然启动没反应
    
#创建rrds目录
$ mkdir /var/lib/ganglia/rrds -p
$ chown -R nobody:nobody  /var/lib/ganglia/rrds

#修改gmetad配置文件
$ vi /usr/local/ganglia/etc/gmetad.conf
	data_source "master" 172.168.0.10:8649   #这也是我们以后经常修改的地方，""里面是组名称  后面是去哪个IP的那个端口去采集gmond数据

#修改web界面时间
$ cd /var/www/html/ganglia
$ vim header.php
    <?php
    session_start();
    ini_set('date.timezone','PRC');  #加入这条

```

#### 客户端监控

```shell
$ yum -y install apr-devel apr-util check-devel cairo-devel pango-devel libxml2-devel rpm-build glib2-devel dbus-devel freetype-devel fontconfig-devel gcc gcc-c++ expat-devel python-devel libXrender-devel

$ yum install -y libart_lgpl-devel pcre-devel libtool
$ mkdir /usr/local/tools
$ cd /usr/local/tools
$ wget http://www.mirrorservice.org/sites/download.savannah.gnu.org/releases/confuse/confuse-2.7.tar.gz
$ tar zxvf confuse-2.7.tar.gz
$ cd confuse-2.7
$ ./configure  --prefix=/usr/local/ganglia-tools/confuse CFLAGS=-fPIC --disable-nls --libdir=/usr/local/ganglia-tools/confuse/lib64
$ make && make install
$ cd /usr/local/tools
$ wget https://sourceforge.net/projects/ganglia/files/ganglia%20monitoring%20core/3.7.2/ganglia-3.7.2.tar.gz
$ tar zxvf ganglia-3.7.2.tar.gz
$ cd ganglia-3.7.2
$ ./configure --prefix=/usr/local/ganglia --enable-gexec --enable-status  --with-libconfuse=/usr/local/ganglia-tools/confuse
$ make && make install
$ /usr/local/ganglia/sbin/gmond -t >/usr/local/ganglia/etc/gmond.conf
$ cp /tools/ganglia-3.7.2/gmond/gmond.init /etc/init.d/gmond
$ mkdir -p /usr/local/ganglia/var/run
$ /etc/init.d/gmond restart
```

#### 服务启动

```shell
#服务端启动
$ systemctl restart httpd
$ systemctl start gmetad 
$ systemctl start gmond

#客户端启动
$ systemctl start gmond
```

#### flume运行命令

```shell
$ nohup flume-ng agent --conf conf/ --conf-file /usr/local/flume/jobs/cloudDiskLog/cloudDiskLogs-hdfs.conf --name d2 -Dflume.monitoring.type=ganglia -Dflume.monitoring.hosts=172.168.0.10:8649 &
```

## 十、集群脚本

### 10.1、集群分发脚本

```shell
#!/bin/bash 
 
#1. 判断参数个数 
# 判断参数是否小于1
if [ $# -lt 1 ]   
then 
    echo Not Enough Arguement! 
    exit; 
fi 

#2. 遍历集群所有机器 
for host in master node1 node2 node3
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
```

### 10.2、Hadoop集群启动脚本

```shell
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
        ssh master "/usr/local/hadoop/sbin/start-dfs.sh" 
        echo " --------------- 启动 yarn ---------------" 
        ssh master "/usr/local/hadoop/sbin/start-yarn.sh" 
        echo " --------------- 启动 historyserver ---------------" 
        ssh node1 "/usr/local/hadoop/bin/mapred --daemon start historyserver" 
;; 
"stop") 
        echo " =================== 关闭 hadoop 集群 ===================" 
 
        echo " --------------- 关闭 historyserver ---------------" 
        ssh node1 "/usr/local/hadoop/bin/mapred --daemon stop historyserver" 
        echo " --------------- 关闭 yarn ---------------" 
        ssh master "/usr/local/hadoop/sbin/stop-yarn.sh" 
        echo " --------------- 关闭 hdfs ---------------" 
        ssh master "/usr/local/hadoop/sbin/stop-dfs.sh" 
;;
*) 
    echo "Input Args Error..." 
;; 
esac 
```

```shell
#!/bin/bash

if [ $# -lt 1 ]
then
	echo "Not Enough Arguement!"
	exit;
fi

case $1 in
"start")
	echo " =================== 启动HAHadoop集群 ================="
	echo " ------------------ 启动Zookeeper集群 -----------------"
	ssh master myzk.sh start
	echo " ------------------ 启动Hadoop集群 --------------------"
	ssh master myhadoop.sh start
;;
"stop")
	echo "------------------- 关闭Hadoop集群 --------------------"
	ssh master myhadoop.sh stop
	echo "-----------------  关闭Zookeeper集群 ------------------"
	ssh master myzk.sh stop
;;
*)
	echo "Input Args Error..."
;;
esac
```

### 10.3、Zookeeper集群脚本

```shell
#!/bin/bash 
 
if [ $# -lt 1 ] 
then 
    echo "No Args Input..." 
    exit ; 
fi 
 
case $1 in 
"start")
	echo ==================== "master" ==========================
	ssh master zkServer.sh start
	echo ==================== "node1" ===========================
	ssh node1 zkServer.sh start
	echo ==================== "node2" ===========================
	ssh node2 zkServer.sh start
;;
"stop")
	echo =================== "master" ===========================
	ssh master zkServer.sh stop
	echo =================== "node1" ============================
	ssh node1 zkServer.sh stop
	echo =================== "node2" ============================
	ssh node2 zkServer.sh stop
;;
"status")
        echo =================== "master" ===========================
        ssh master zkServer.sh status
        echo =================== "node1" ============================
        ssh node1 zkServer.sh status
        echo =================== "node2" ============================
        ssh node2 zkServer.sh status
;;
*)
	echo "Input Args error..."
;;
esac
```

### 10.4、Jn集群脚本

```shell
#!/bin/bash 
 
if [ $# -lt 1 ] 
then 
    echo "No Args Input..." 
    exit ; 
fi 
 
case $1 in 
"start") 
        echo " --------------- node1 ---------------" 
        ssh node1 hdfs --daemon start journalnode
        echo " --------------- node2 ---------------" 
        ssh node2 hdfs --daemon start journalnode
        echo " --------------- node3 ---------------" 
        ssh node3 hdfs --daemon start journalnode
;;
"stop")
	echo " --------------- node1 ---------------" 
        ssh node1 hdfs --daemon stop journalnode
        echo " --------------- node2 ---------------" 
        ssh node2 hdfs --daemon stop journalnode
        echo " --------------- node3 ---------------" 
        ssh node3 hdfs --daemon stop journalnode
;;
*) 
    echo "Input Args Error..." 
;; 
esac 
```

### 10.5、Azkaban脚本

```shell
#!/bin/bash 
 
if [ $# -lt 1 ] 
then 
    echo "No Args Input..." 
    exit ; 
fi 

case $1 in 
"start")
	echo ================== "启动Azkaban" ==================
	cd /opt/azkabanlogs
	echo "-----------------启动azkaban-exec----------------"
	azkaban-executor-start.sh
	echo "-----------------启动azkaban-web-----------------"
	azkaban-web-start.sh
;;
("stop")
	echo ================== "关闭Azkaban" ==================
	echo "-----------------关闭azkaban-exec----------------"
        azkaban-executor-shutdown.sh
	echo "-----------------关闭azkaban-web-----------------"
        azkaban-web-shutdown.sh
;;
*) 
    echo "Input Args Error..." 
;; 
esac 
```

### 10.6、查看集群Jps脚本

```shell
#!/bin/bash 
 
echo ============集群状态===============
for host in master node1 node2 node3
do 
        echo =============== $host =============== 
        ssh $host jps  
done 
echo ===========执行完毕================
```

### 10.7、GangLia集群脚本

```shell
#!/bin/bash 
 
if [ $# -lt 1 ] 
then 
    echo "No Args Input..." 
    exit ; 
fi 

case $1 in 
"start")
	echo ================== "启动GangLia" ==================
	echo "------------------启动服务端-----------------------"
	ssh master systemctl start httpd
	ssh master systemctl start gmetad
	ssh master systemctl start gmond
	echo "------------------启动客户端-----------------------"
	ssh node1 systemctl start gmond
	ssh node2 systemctl start gmond
	ssh node3 systemctl start gmond
;;
("stop")
	echo ================== "关闭GangLia" ==================
	echo "------------------关闭服务端-----------------------"
	ssh master systemctl stop httpd
	ssh master systemctl stop gmetad
	ssh master systemctl stop gmond
	echo "------------------关闭客户端-----------------------"
	ssh node1 systemctl stop gmond
	ssh node2 systemctl stop gmond
	ssh node3 systemctl stop gmond
;;
*) 
    echo "Input Args Error..." 
;; 
esac 
```
