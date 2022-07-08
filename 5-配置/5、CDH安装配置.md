# CDH安装配置

https://2897669115@qq.com:1953617ZhuLin084@@archive.cloudera.com/cdh6/6.3.2-patch4071/parcels/

## 一、宿主机初始化

### 1.1、配置yum源

```shell
$ yum install -y wget 
$ mkdir -p /etc/yum.repos.d/repo_bak 
$ mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/repo_bak/ 
$ wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo 
$ wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo 
$ yum clean all 
$ yum makecache 
$ yum update –y
```

### 1.2、安装Docker

#### 1.2.1、 卸载旧版本

Linux sudo命令以系统管理者的身份执行指令，也就是说，经由 sudo 所执行的指令就好像是 root 亲自执行。

使用权限：在 /etc/sudoers 中有出现的使用者。

```shell
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

#### 1.2.2 安装Docker软件包

```shell
$ sudo yum install -y yum-utils
```

#### 1.2.3 设置镜像仓库地址

```shell
# 默认是国外的
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
# 换成阿里云镜像地址
$ sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

#### 1.2.4 安装最新版Docker Engine和容器

安装前建议先将将服务器上的软件包信息现在本地缓存，以提高安装软件的速度

```shell
$ yum makecache fast

# docker-ce社区版(docker-ee企业版) 安装docker（docker的引擎、操作docker的客户端、docker容器）
$ yum -y install docker-ce docker-ce-cli containerd.io
```

#### 1.2.5 启动Docker

```shell
sudo systemctl start docker

systemctl enable docker.service
//设置开机启动
启动脚本
# docker.service
#!/bin/sh
sudo systemctl enable docker
sudo systemctl start docker

将脚本放置在/etc/init.d/目录下，修改成root执行权限，然后输入
sysv-rc-conf
```

### 1.3、安装docker命令补全工具

```shell
$ yum install -y bash-completion 
$ source /usr/share/bash-completion/completions/docker 
$ source /usr/share/bash-completion/bash_completion 
$ yum clean all
```

### 1.4、安装基本工具

```shell
$ yum install -y vim wget ntp net-tools 
$ yum clean all
```

### 1.5、关闭防火墙与SELinux模式

```shell
$ systemctl stop firewalld 
$ systemctl disable firewalld 
$ systemctl status firewalld

#关闭selinux 输入vi /etc/selinux/config,将SELINUX=disabled
    # This file controls the state of SELinux on the system.
    # SELINUX= can take one of these three values:
    #     enforcing - SELinux security policy is enforced.
    #     permissive - SELinux prints warnings instead of enforcing.
    #     disabled - No SELinux policy is loaded.
    SELINUX=disabled
    # SELINUXTYPE= can take one of three two values:
    #     targeted - Targeted processes are protected,
    #     minimum - Modification of targeted policy. Only selected processes are protected.
    #     mls - Multi Level Security protection.
    SELINUXTYPE=targeted
```

### 1.7、配置时间同步

```
安装ntp服务软件包：yum install ntp
将ntp设置为缺省启动：systemctl enable ntpd
启动ntp服务：service ntpd restart
将系统时区改为上海时间（即CST时区）：ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
输入date命令查看时间是否正确
```

## 二、创建docker容器

### 2.1、构建基本系统镜像

```dockerfile
#dockerfile
FROM docker.io/ansible/centos7-ansible
RUN yum -y install openssh-server
RUN yum -y install bind-utils
RUN yum -y install which
RUN yum -y install sudo
RUN echo "root:a"  | chpasswd
RUN echo "root ALL=(ALL)  ALL"  >> /etc/sudoers
RUN ssh-keygen -t dsa  -f /etc/ssh/ssh_host_dsa_key
RUN ssh-keygen -t rsa  -f /etc/ssh/ssh_host_rsa_key

RUN mkdir /var/run/sshd
EXPOSE 22                    # 开放的端口
CMD ["/usr/sbin/sshd","-D"]      # 执行的命令，这里为启动的命令，在/lib/systemd/system/sshd.service 可以查看到相应的启动命令

#执行
docker build -t centos7-cdh .
```

### 2.1、创建自定义网络

```shell
$ docker network create --subnet=172.10.0.0/16 hadoop_net 
$ docker network ls
```

### 2.2、启动容器

```shell
$ docker run -d --add-host hadoop1:172.10.0.2 --add-host hadoop2:172.10.0.3 --add-host hadoop3:172.10.0.4 --net hadoop_net --ip 172.10.0.2 -h hadoop1 -p 10022:22 -p 7180:7180 --restart always --name hadoop1 --privileged centos7-cdh /usr/sbin/init

$ docker run -d --add-host hadoop1:172.10.0.2 --add-host hadoop2:172.10.0.3 --add-host hadoop3:172.10.0.4  --net hadoop_net --ip 172.10.0.3 -h hadoop2 -p 20022:22 --restart always --name hadoop2 --privileged centos7-cdh /usr/sbin/init
    
$ docker run -d --add-host hadoop1:172.10.0.2 --add-host hadoop2:172.10.0.3 --add-host hadoop3:172.10.0.4  --net hadoop_net --ip 172.10.0.4 -h hadoop3 -p 30022:22 --restart always --name hadoop3 --privileged centos7-cdh /usr/sbin/init 
   
$ docker ps
```

### 2.3、本地远程连接hadoop1

```
#在本地使用管理员登录cmd执行以下
route -p add 172.10.0.0/16  192.168.30.200
```

## 三、容器安装ClouderaManager

### 3.1、安装基础环境

```shell
$ yum install -y kde-l10n-Chinese telnet reinstall glibc-common vim wget ntp net-tools 
$ yum clean all
```

### 3.2、配置中文环境变量

```
(
cat <<EOF
export LC_ALL=zh_CN.utf8
export LANG=zh_CN.utf8
export LANGUAGE=zh_CN.utf8
EOF
) >> ~/.bashrc 
$ localedef -c -f UTF-8 -i zh_CN zh_CN.utf8 
$ source ~/.bashrc 
$ echo $LANG
```

### 3.3、配置时间同步

```
安装ntp服务软件包：yum install ntp
将ntp设置为缺省启动：systemctl enable ntpd
启动ntp服务：systemctl restart ntpd
将系统时区改为上海时间（即CST时区）：ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
输入date命令查看时间是否正确
```

### 3.4、安装Mysql包

```shell
$ mkdir -p /usr/local/hadoop_CHD/mysql 
$ wget -O /usr/local/hadoop_CHD/mysql/mysql-5.7.27-1.el7.x86_64.rpm-bundle.tar https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.27-1.el7.x86_64.rpm-bundle.tar
$ tar -xvf mysql-5.7.27-1.el7.x86_64.rpm-bundle.tar 
$ yum install -y libaio numactl 
$ rpm -ivh mysql-community-common-5.7.27-1.el7.x86_64.rpm 
$ rpm -ivh mysql-community-libs-5.7.27-1.el7.x86_64.rpm 
$ rpm -ivh mysql-community-client-5.7.27-1.el7.x86_64.rpm 
$ rpm -ivh mysql-community-server-5.7.27-1.el7.x86_64.rpm 
$ rpm -ivh mysql-community-libs-compat-5.7.27-1.el7.x86_64.rpm 
$ echo character-set-server=utf8 >> /etc/my.cnf 
$ rm -rf /usr/local/hadoop_CHD/mysql/ 
$ yum clean all 
$ rpm -qa |grep mysql

#上传mysql驱动
$ mkdir -p /usr/local/hadoop_CHD/mysql-jdbc 
$ wget -O /root/hadoop_CHD/mysql-jdbc/mysql-connector-java-5.1.48.tar.gz https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.48.tar.gz 
$ ls /root/hadoop_CHD/mysql-jdbc
```

### 3.5、准备Cloudera-Manager安装包

```shell
$ mkdir -p /usr/local/hadoop_CHD/cloudera-repos 
$ wget -O /usr/local/hadoop_CHD/cloudera-repos/allkeys.asc https://archive.cloudera.com/cm6/6.3.0/allkeys.asc
$ wget -O /usr/local/hadoop_CHD/cloudera-repos/cloudera-manager-agent-6.3.0-1281944.el7.x86_64.rpm https://archive.cloudera.com/cm6/6.3.0/redhat7/yum/RPMS/x86_64/cloudera-manager-agent-6.3.0-1281944.el7.x86_64.rpm 
$ wget -O /usr/local/hadoop_CHD/cloudera-repos/cloudera-manager-daemons-6.3.0-1281944.el7.x86_64.rpm https://archive.cloudera.com/cm6/6.3.0/redhat7/yum/RPMS/x86_64/cloudera-manager-daemons-6.3.0-1281944.el7.x86_64.rpm 
$ wget -O /usr/local/hadoop_CHD/cloudera-repos/cloudera-manager-server-6.3.0-1281944.el7.x86_64.rpm https://archive.cloudera.com/cm6/6.3.0/redhat7/yum/RPMS/x86_64/cloudera-manager-server-6.3.0-1281944.el7.x86_64.rpm 
$ wget -O /usr/local/hadoop_CHD/cloudera-repos/cloudera-manager-server-db-2-6.3.0-1281944.el7.x86_64.rpm https://archive.cloudera.com/cm6/6.3.0/redhat7/yum/RPMS/x86_64/cloudera-manager-server-db-2-6.3.0-1281944.el7.x86_64.rpm 
$ wget -O /usr/local/hadoop_CHD/cloudera-repos/enterprise-debuginfo-6.3.0-1281944.el7.x86_64.rpm https://archive.cloudera.com/cm6/6.3.0/redhat7/yum/RPMS/x86_64/enterprise-debuginfo-6.3.0-1281944.el7.x86_64.rpm 
$ wget -O /usr/local/hadoop_CHD/cloudera-repos/oracle-j2sdk1.8-1.8.0+update181-1.x86_64.rpm https://archive.cloudera.com/cm6/6.3.0/redhat7/yum/RPMS/x86_64/oracle-j2sdk1.8-1.8.0+update181-1.x86_64.rpm 
$ ll /usr/local/hadoop_CHD/cloudera-repos
```

### 3.6、准备Parcel包

```shell
$ mkdir -p /usr/local/hadoop_CHD/parcel 
$ wget -O /usr/local/hadoop_CHD/parcel/ CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel https://archive.cloudera.com/cdh6/6.3.2/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel 
$ wget -O /usr/local/hadoop_CHD/parcel/manifest.json https://archive.cloudera.com/cdh6/6.3.2/parcels/manifest.json 
$ ll /usr/local/hadoop_CHD/parcel
```

### 3.7、搭建本地yum源

```shell
$ yum -y install httpd createrepo 
$ systemctl start httpd
$ systemctl enable httpd
$ cd /usr/local/hadoop_CHD/cloudera-repos/ && createrepo .
$ mv /usr/local/hadoop_CHD/cloudera-repos /var/www/html/
$ yum clean all
$ ll /var/www/html/cloudera-repos
```

### 3.8、安装jdk

```shell
$ cd /var/www/html/cloudera-repos/;rpm -ivh oracle-j2sdk1.8-1.8.0+update181-1.x86_64.rpm
```

### 3.9、数据库授权

```shell
#创建sql脚本
(
cat <<EOF
set password for root@localhost = password('aaaa');
grant all privileges on *.* to 'root'@'%' identified by 'aaaa';
flush privileges;
CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY 'aaaa';
GRANT ALL ON amon.* TO 'amon'@'%' IDENTIFIED BY 'aaaa';
GRANT ALL ON rman.* TO 'rman'@'%' IDENTIFIED BY 'aaaa';
GRANT ALL ON hue.* TO 'hue'@'%' IDENTIFIED BY 'aaaa';
GRANT ALL ON metastore.* TO 'hive'@'%' IDENTIFIED BY 'aaaa';
GRANT ALL ON sentry.* TO 'sentry'@'%' IDENTIFIED BY 'aaaa';
GRANT ALL ON nav.* TO 'nav'@'%' IDENTIFIED BY 'aaaa';
GRANT ALL ON navms.* TO 'navms'@'%' IDENTIFIED BY 'aaaa';
GRANT ALL ON oozie.* TO 'oozie'@'%' IDENTIFIED BY 'aaaa';
SHOW DATABASES;
EOF
) >> /root/c.sql

#获取MySQL初始密码
$ systemctl start mysqld && grep password /var/log/mysqld.log | sed 's/.*(............)$/1/'
#7XWdlZseA=yO

#修改密码
$ mysql> ALTER USER USER() IDENTIFIED BY 'Admin2022!';
#查看当前密码规则
$ mysql> SHOW VARIABLES LIKE 'validate_password%';
#调整密码规则
$ mysql> set global validate_password.policy=0;
$ mysql>  set global validate_password.length=1;

#执行SQL脚本
$ mysql -uroot –p
$ source /root/c.sql
```

### 3.10、配置mysql jdbc驱动

```shell
$ mkdir -p /usr/share/java/ 
$ cd /usr/local/hadoop_CHD/mysql-jdbc/;tar -zxvf mysql-connector-java-5.1.48.tar.gz 
$ cp  /usr/local/hadoop_CHD/mysql-jdbc/mysql-connector-java-5.1.48/mysql-connector-java-5.1.48-bin.jar /usr/share/java/mysql-connector-java.jar 
$ rm -rf /usr/local/hadoop_CHD/mysql-jdbc/ 
$ ls /usr/share/java/
```

### 3.11、安装Cloudera Manager

```shell
(
cat <<EOF
[cloudera-manager]
name=Cloudera Manager 6.3.0
baseurl=http://172.10.0.2/cloudera-repos/
gpgcheck=0
enabled=1
EOF
) >> /etc/yum.repos.d/cloudera-manager.repo 
$ yum clean all 
$ yum makecache 
$ yum install -y cloudera-manager-daemons cloudera-manager-agent cloudera-manager-server 
$ yum clean all 
$ rpm -qa | grep cloudera-manager
```

### 3.12、配置parcel库

```shell
$ cd /opt/cloudera/parcel-repo/;mv /usr/local/hadoop_CHD/parcel/* ./ 
$ sha1sum CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel | awk '{ print $1 }' > CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel.sha 
$ rm -rf /usr/local/hadoop_CHD/parcel/ 
$ chown -R cloudera-scm:cloudera-scm /opt/cloudera/parcel-repo/* 
$ ll /opt/cloudera/parcel-repo/
```

### 3.13、初始化scm库

```shell
/opt/cloudera/cm/schema/scm_prepare_database.sh mysql scm scm 123456Aa.
```

### 3.14、启动服务

```shell
$ systemctl start cloudera-scm-server 
$ systemctl start cloudera-scm-agent 
$ sleep 2 
$ tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log | grep "INFO WebServerImpl:com.cloudera.server.cmf.WebServerImpl: Started Jetty server"

#管理命令
systemctl start cloudera-scm-server 
systemctl status cloudera-scm-server 
systemctl stop cloudera-scm-server 

systemctl start cloudera-scm-agent 
systemctl status cloudera-scm-agent 
systemctl stop cloudera-scm-agent  
```

## 四、节点容器执行

### 4.1、安装基本工具

```shell
$ yum install -y kde-l10n-Chinese telnet reinstall glibc-common vim wget ntp net-tools 
$ yum clean all
```

### 4.2、配置中文环境变量

```shell
(
cat <<EOF
export LC_ALL=zh_CN.utf8
export LANG=zh_CN.utf8
export LANGUAGE=zh_CN.utf8
EOF
) >> ~/.bashrc 
$ localedef -c -f UTF-8 -i zh_CN zh_CN.utf8 
$ source ~/.bashrc 
$ echo $LANG
```

### 4.3、配置MySQL JDBC

```
$ mkdir -p /usr/share/java/ 
$ wget -O /usr/share/java/mysql-connector-java-5.1.48.tar.gz https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.48.tar.gz 
$ cd /usr/share/java/;tar -zxvf mysql-connector-java-5.1.48.tar.gz 
$ cp /usr/share/java/mysql-connector-java-5.1.48/mysql-connector-java-5.1.48-bin.jar /usr/share/java/mysql-connector-java.jar 
$ rm -rf mysql-connector-java-5.1.48 mysql-connector-java-5.1.48.tar.gz 
$ ls /usr/share/java/
```

## 五、CM管理平台创建CDH集群

### 5.1、登录CM管理平台

http://172.10.0.2:7180/cmf/login 账号密码：admin/admin

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207081530368.png" alt="image-20220708153014867" width="80%;" />

配置集群名称

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207081536694.png" width="80%;" />

选择集群节点

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207081539462.png" alt="image-20220708153946628" width="80%;" />

选择存储库

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207081541701.png" alt="image-20220708154118626" width="80%;" />

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207081541181.png" width="80%;" />

安装JDK

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207081542382.png" alt="image-20220708154233134" width="80%;" />

 SSH凭据，密码为容器root用户的登录密码，此处为root。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207081544777.png" alt="image-20220708154415328" style="width:80%;" />

集群安装

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207081605268.png" alt="image-20220708160459833" width="80%;" />

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207081613011.png" alt="image-20220708161331001" style="width:80%;" />

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207081615354.png" alt="image-20220708161530734" style="width:80%;" />





































## 报错

![](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207081530368.png)
