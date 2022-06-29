# Sqoop

## 一、概述

Apache Sqoop（SQL-to-Hadoop）项目旨在协助RDBMS与Hadoop之间进行高效的大数据交流。用户可以在 Sqoop 的帮助下，轻松地把关系型数据库的数据导入到 Hadoop 与其相关的系统 (如HBase和Hive)中；同时也可以把数据从 Hadoop 系统里抽取并导出到关系型数据库里。

Sqoop是一个在结构化数据和Hadoop之间进行批量数据迁移的工具，结构化数据可以是MySQL、Oracle等RDBMS。Sqoop底层用MapReduce程序实现抽取、转换、加载，MapReduce天生的特性保证了并行化和高容错率，而且相比Kettle等传统ETL工具，任务跑在Hadoop集群上，减少了ETL服务器资源的使用情况。在特定场景下，抽取过程会有很大的性能提升。

Sqoop可以将数据从关系数据库系统或大型机导入 HDFS。导入过程的输入是数据库表或大型机数据集。对于数据库，Sqoop 会将表逐行读入 HDFS。对于大型机数据集，Sqoop 会将每个大型机数据集的记录读入 HDFS。此导入过程的输出是一组文件，其中包含导入的表或数据集的副本。导入过程是并行执行的。因此，输出将位于多个文件中。这些文件可以是分隔的文本文件（例如，用逗号或制表符分隔每个字段），或者是包含序列化记录数据的二进制 Avro 或 SequenceFiles。

命令

## 二、Sqoop工具详解

### 2.1、Import详解

**通用参数**

--connect <jdbc-uri>、指定JDBC连接字符串 || --connection-manager <class-name>、指定要使用的连接管理器类 || --driver <class-name>、手动指定要使用的JDBC驱动程序类 || --verbose、工作时打印更多信息 || --relaxed-isolation、将连接事务隔离设置为为映射程序读取未提交的数据。

**验证参数**

--validate、启用数据复制验证，仅支持单表复制。 || --validator <class-name>、指定要使用的验证器类 || --validation-threshold <class-name> ||--validation-failurehandler <class-name>

**import控制参数**

| Argument                                                | Description                                        |
| ------------------------------------------------------- | -------------------------------------------------- |
| --append                                                | 在HDFS中向已有的数据集追加数据                     |
| --as-avrodatafile（sequencefile、textfile、parquetfile) | 导入数据到avro文件、序列文件、text、Parquet文件    |
| --columns <col,col,col…>                                | 选择导入的表的列                                   |
| --delete-target-dir                                     | 删除导入的目标目录，若存在则删除                   |
| --direct                                                | 如果数据库存在直接连接器，则使用直接连接器         |
| --fetch-size <n>                                        | 一次从数据库中读取的条目数                         |
| -m,--num-mappers <n>                                    | 使用n个map任务并行导入                             |
| --table <table-name>                                    | 导入的表                                           |
| --target-dir <dir>                                      | 导入hdfs的目标文件                                 |
| --temporary-rootdir <dir>                               | 导入时创建的临时文件的HDFS目录(覆盖默认的“_sqoop”) |
| --warehouse-dir <dir>                                   | HDFS父目录用于表的目的地                           |
| --compression-codec <c>                                 | 使用Hadoop编解码器(默认为gzip)                     |
| --where <where clause>                                  | 导入期间使用where判断                              |
| --boundary-query                                        | 导入的边界值                                       |
| --query                                                 | 自定义sql语句                                      |

```shell
1、$ sqoop import --connect jdbc:mysql://zhulinz.top:3306/testsqoop?serverTimezone=UTC --username zhulin -P --table project --target-dir /mysql/project --delete-target-dir

2、$ sqoop import \
  --query 'SELECT a.*, b.* FROM a JOIN b on (a.id == b.id) WHERE $CONDITIONS' \
  --split-by a.id --target-dir /mysql/
  # --query不能与--table、--colum合用    指定$CONDITIONS表明分区列
  
3、$ sqoop import --connect jdbc:mysql://zhulinz.top:3306/testsqoop?serverTimezone=UTC --username zhulin -P --table project --warehouse-dir /mysql/projects
#分析：将导入的文件 保存到 hdfs://zy2/input/project
#与上面的对比： 在   指定的   /zy2/input下创建一个目录，名字为表名.    此时这个表名就当成了一个数据仓库名  (warehouse )

4、$ sqoop import --connect jdbc:mysql://zhulinz.top:3306/testsqoop?serverTimezone=UTC --username zhulin --password zhulin0804 --table project  --warehouse-dir /mysql/input  --columns  'id,name,type' -m 1 --delete-target-dir
#分析：-m 1 表示只用到一个mapper, 一个mapper对应一个切片，对应一个输出文件. 
       --columns   指定列名
       --where    指定条件
       因为用了  --table, 所以以上会自动地拼装sql 语句. , 不能与    -e or -query 合用. 
       
5、$ sqoop import --connect jdbc:mysql://zhulinz.top:3306/testsqoop?serverTimezone=UTC --username zhulin --password zhulin0804 --target-dir /zy5/input  --query 'select id,name,type from project where id>2 and  $CONDITIONS'  --split-by project.id -m 1
#分析:-query 不能与 --table, --column 合用. 
      指定  $CONDITIONS  表明分区列.
      --target-dir  保存数据的数仓名字. 
      
6、$ sqoop import --connect jdbc:mysql://zhulinz.top:3306/testsqoop?serverTimezone=UTC --username zhulin --password zhulin0804 --table project --target-dir /mysql/input1  --direct   -m 1
#分析:  --direct    使用    mysqldump   命令完成导入工作    因为是集群，map任务是分配到每个节点运行，所以每个节点都要有mysqldump命令.

==================================增量导入===================================
7、$ sqoop import --connect jdbc:mysql://zhulinz:3306/testsqoop?serverTimezone=UTC --username zhulin --password zhulin0804  --target-dir /zy7/input  --table project -m 1 --check-column id   --incremental append --last-value 3

insert into project( name,type,description,create_at,status)
values( 'project5',5,'project5 zy','2019-07-25',0);
insert into project( name,type,description,create_at,status)
values( 'project6',5,'project5 zy','2019-07-25',0);
insert into project( name,type,description,create_at,status)
values( 'project7',5,'project5 zy','2019-07-25',0);
insert into project( name,type,description,create_at,status)
values( 'project8',5,'project5 zy','2019-07-25',0);

8、$ sqoop import --connect jdbc:mysql://zhulinz:3306/testsqoop?serverTimezone=UTC --username zhulin --password zhulin0804  --target-dir /zy7/input  --table project -m 1 --check-column id   --incremental append --last-value 7

insert into project( name,type,description,create_at,status)
values( 'project6',5,'project5 zy','2019-07-25',0);
insert into project( name,type,description,create_at,status)
values( 'project7',5,'project5 zy','2019-07-25',0);
insert into project( name,type,description,create_at,status)
values( 'project8',5,'project5 zy','2019-07-25',0);
insert into project( name,type,description,create_at,status)
values( 'project9',5,'project5 zy','2019-07-25',0);

分析：以上运行 增量导入两次，生成了两个文件.   均按  last-value 导入. 

9、$ sqoop import   --connect jdbc:mysql://zhulinz:3306/testsqoop?serverTimezone=UTC --username zhulin --password zhulin0804  --target-dir /zy8/input  --table project -m 1 --check-column update_at  --incremental lastmodified  --last-value "2022-06-28 16:45:12" --append
```

## 三、Sqoop安装

### 3.1、配置环境变量

```shell
#进入配置文件
$ vim /etc/profile

#Sqoop
export SQOOP_HOME=/usr/local/sqoop147
export PATH=$PATH:$SQOOP_HOME/bin
```

### 3.2、配置Sqoop的配置

```shell
#进入Sqoop配置文件
$ cd /usr/local/sqoop147/conf
$ vim sqoop-env.sh

#Set path to where bin/hadoop is available
export HADOOP_COMMON_HOME=/usr/local/hadoop

#Set path to where hadoop-*-core.jar is available
export HADOOP_MAPRED_HOME=/usr/local/hadoop
```

### 3.3、复制驱动包

将mysql的驱动包放到sqoop的lib目录下，将sqoop的驱动包放到hadoop的lib目录下
