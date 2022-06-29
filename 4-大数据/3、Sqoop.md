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

```shell
1、sqoop import --connect jdbc:mysql://zhulinz.top:3306/testsqoop?serverTimezone=UTC --username zhulin -P --table project --target-dir /mysql/project
```



```shell
sqoop-list-databases --connect jdbc:mysql://zhulinz.top:3306/mysql?serverTimezone=UTC --username zhulin -P --verbose
```
