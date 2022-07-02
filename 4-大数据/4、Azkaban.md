# Azkaban工作流调度系统

## 一、概述

### 1.1、工作流调度系统

一个完整的数据分析系统通常都是由大量任务单元组成：shell脚本程序、java程序、mapreduce程序、hive脚本等。各任务单元之间存在时间先后及前后依赖关系。为了很好地组织起这样的复杂执行计划，需要一个工作流调度系统来调度执行。

需求：某个业务系统每天产生20G原始数据，每天都要对其进行处理，步骤如下：

1. 通过Hadoop先将原始数据上传到Hdfs上（HDFS的操作）
2. 使用MapReduce对原始数据进行清洗（MapReduce的操作）
3. 将清洗后的数据导入到hive表中（hive导入操作）
4. 对hive中多个表的数据进行JOIN处理，得到一张hive的明细表（创建中间表）
5. 通过对明细表的统计和分析，得到结果报表信息（hive的查询操作）

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207021826058.png" alt="image-20220702182602757" width="50%;" />

### 1.2、适用场景

根据以上业务场景： （2）任务依赖（1）任务的结果，（3）任务依赖（2）任务的结果，（4）任务依赖（3）任务的结果，（5）任务依赖（4）任务的结果。一般的做法是，先执行完（1）再执行（2），再一次执行（3）（4）（5）。

这样的话，整个的执行过程都需要人工参加，并且得盯着各任务的进度。但是我们的很多任务都是在深更半夜执行的，通过写脚本设置crontab执行。其实，整个过程类似于一个有向无环图（DAG）。每个子任务相当于大任务中的一个节点，也就是，我们需要的就是一个工作流的调度器，而Azkaban就是能解决上述问题的一个调度器。

### 1.3、特点

- 兼容任何版本的hadoop
- 易于使用的Web用户界面
- 简单的工作流的上传
- 方便设置任务之间的关系
- 调度工作流
- 模块化和可插拔的插件机制
- 认证/授权(权限的工作)
- 能够杀死并重新启动工作流
- 有关失败和成功的电子邮件提醒

## 二、架构

`Azkaban`在LinkedIn上实施，以解决`Hadoop`作业依赖问题。从ETL工作到数据分析产品，工作都有需要按顺序运行。最初是单一服务器解决方案，随着多年来Hadoop用户数量的增加，Azkaban 已经发展成为一个更强大的解决方案。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207021830407.png" alt="image-20220702183009719" width="50%;" />

Azkaban由三个关键组件构成：元数据、AzkabanWebServer、AzkabanExecutorServer。

### 2.1、元数据

使用关系型数据库存储元数据和执行状态。

`AzkabanWebServer`

- **项目管理**：项目、项目权限以及上传的文件。
- **作业状态**：跟踪执行流程以及执行程序正在运行的流程。
- **以前的流程/作业**：通过以前的作业和流程执行以及访问其日志文件进行搜索。
- **计划程序**：保留计划作业的状态。
- **SLA**：保持所有的SLA规则

`AzkabanExecutorServer`

- **访问项目**：从数据库检索项目文件。
- **执行流程/作业**：检索和更新正在执行的作业流的数据
- **日志**：将作业和工作流的输出日志存储到数据库中。
- **交互依赖关系**：如果一个工作流在不同的执行器上运行，它将从数据库中获取状态。

### 2.2、AzkabanWebServer

`AzkabanWebServer`是整个Azkaban工作流系统的主要管理者，它负责Project管理、用户登录认证、定时执行工作流、跟踪工作流执行进度等一系列任务。同时，它还提供Web服务操作接口，利用该接口，用户可以使用curl或其他Ajax的方式，来执行Azkaban的相关操作。操作包括：用户登录、创建Project、上传Workflow、执行Workflow、查询Workflow的执行进度、杀掉Workflow等一系列操作，且这些操作的返回结果均是JSON格式。并且Azkaban使用方便，Azkaban使用以.job为后缀名的键值属性文件来定义工作流中的各个任务，以及使用dependencies属性来定义作业间的依赖关系链。这些作业文件和关联的代码最终以*.zip的方式通过Azkaban UI上传到Web服务器上。

### 2.3、AzkabanExecutorServer

以前版本的Azkaban在单个服务中具有AzkabanWebServer和AzkabanExecutorServer功能，目前Azkaban已将AzkabanExecutorServer分离成独立的服务器，拆分AzkabanExecutorServer的原因有如下几点：

- 某个任务流失败后，可以更方便将其重新执行
- 便于Azkaban升级

AzkabanExecutorServer主要负责具体的工作流`提交、执行`，可以启动多个执行服务器，它们通过`关系型数据库`来协调任务的执行。

### 2.4、作业流执行过程

- WebServer根据内存中缓存的各Executor的资源（WebServer有一个线程会遍历各个Active Executor，去发送Http请求获取其资源状态信息缓存到内存中），按照选择策略（包括executor资源状态、最近执行流个数等）选择一个Executor下发作业流。
- Executor判断是否设置作业粒度分配，如果未设置作业粒度分配，则在当前Executor执行所有作业；如果设置了作业粒度分配，则当前节点会成为作业分配的决策者，即分配节点。
- 分配节点从Zookeeper获取各个Executor的资源状态信息，然后根据策略选择一个Executor分配作业。
- 被分配到作业的Executor即成为执行节点，执行作业，然后更新数据库。

## 三、Azkaban安装

### 3.1、上传安装包

将Azkaban Web服务器、Azkaban执行服务器、Azkaban的sql执行脚本及MySQL安装包拷贝到`master`虚拟机/user/local/azkaban目录下

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202206300914647.png" alt="image-20220630091249409" width="50%;" />

### 3.2、安装

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

### 3.3、生成密钥库

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

### 3.4、时间同步配置

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

### 3.5、配置文件

#### 3.5.1、Web服务器配置

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

#### 3.5.2、执行服务器executor配置

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

#### 3.5.3、启动executor服务器

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

#### 3.5.4、启动web服务器

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

## 四、Azkaban Job

### 4.1、串行定时任务工作流

```shell
#zip目录结构
|--start.job
|--finish.job

#执行脚本
# start.job
type=command
command=hadoop jar /opt/mapreduce/cloudMap.jar com.zhulin.Webwork /flume/nginxlogs/access/2022-07-02 /flume/nginxlogs/mapaccess/site /flume/nginxlogs/mapaccess/client /flume/nginxlogs/mapaccess/os

# sqoop1.job 导入dao
type=command
dependencies=start
command=sqoop export --connect jdbc:mysql://121.36.215.98:3306/sqoop --username lijie -export-dir /flume/nginxlogs/mapaccess/site/part-r-00000 --table cloud_disk_log --input-fields-terminated-by '\t' --columns="type,name,num,time"

# sqoop2.job
type=command
dependencies=start
command=sqoop export --connect jdbc:mysql://121.36.215.98:3306/sqoop --username lijie -export-dir /flume/nginxlogs/mapaccess/client/part-r-00000 --table cloud_disk_log --input-fields-terminated-by '\t' --columns="type,name,num,time"

# finsh.job
type=command
dependencies=sqoop1,sqoop2
command=sqoop export --connect jdbc:mysql://121.36.215.98:3306/sqoop --username lijie -export-dir /flume/nginxlogs/mapaccess/os/part-r-00000 --table cloud_disk_log --input-fields-terminated-by '\t' --columns="type,name,num,time"
successEmail=2107450246@qq.com
failureEmail=2107450246@qq.com
```

