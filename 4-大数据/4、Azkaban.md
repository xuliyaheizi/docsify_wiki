# Azkaban工作流调度系统

1. 

## 安装过程

### 上传安装包

将Azkaban Web服务器、Azkaban执行服务器、Azkaban的sql执行脚本及MySQL安装包拷贝到`master`虚拟机/user/local/azkaban目录下

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202206300914647.png" alt="image-20220630091249409" width="50%;" />

### 安装

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
$mysql> create database azkaban;
$mysql> use azkaban;
$mysql> source /usr/local/azkaban/azkaban-2.5.0/create-all-sql-2.5.0.sql
```

### 生成密钥库

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

### 时间同步配置

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

### 配置文件

#### Web服务器配置

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

#### 执行服务器executor配置

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

#### 启动executor服务器

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

#### 启动web服务器

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

