# MySQL

## 一、基本概念

### 1.1、数据库

- **数据库（database）：**保存有组织的数据的容器（通常指一个文件或一组文件）。
- **表：**一种`结构化文件`，可用来存储某种特定类型的数据。某种特定类型数据的结构化清单。
- **模式（schema）：**关于数据库和表的布局及特性的信息。
- **列（column）：**表中的一个字段。所有表都是由一个或多个列组成的。
- **数据类型（datatype）：**所容许的数据的类型。每个表列都有相应的数据类型，它限制（或容许）该列中存储的数据。
- **行（row）：**表中的数据是按行存储的，所保存的每个记录存储在自己的行内。
- **注解（primary key）：**一列（或一组列），其值能够唯一区分表中每个行。
- **SQL：**结构化查询语言的缩写。SQL是一种专门用来与数据库通信的语言。

## 二、常用SQL语句

### 2.1、数据库操作

```sql
## 查看当前数据库
SELECT DATABASE();
## 显示当前时间、用户名、数据库版本
SELECT NOW(),USER(),VERSION();
## 创建库
CREATE DATABASE IF NOT EXISTS  数据库名 数据库选项
    数据库选项：
            CHARACTER SET charset_name
            COLLATE collation_name
## 查看已有库
SHOW DATABASES LIKE 'PATTERN' 
## 查看当前库信息
SHOW CREATE DATABASE 数据库名
## 修改库的选项信息
ALTER DATABASE 库名 选项信息
## 删除库
DROP DATABASE IF EXISTS 数据库名
        同时删除该数据库相关的目录及其目录内容
```

### 2.2、表的操作

```sql
-- 创建表
CREATE [TEMPORARY] TABLE[ IF NOT EXISTS] [库名.]表名 ( 表的结构定义 )[ 表选项]
	每个字段必须有数据类型
	最后一个字段后不能有逗号
	TEMPORARY 临时表，会话结束时表自动消失
	对于字段的定义：
		字段名 数据类型 [NOT NULL | NULL] [DEFAULT default_value] [AUTO_INCREMENT] [UNIQUE [KEY] | [PRIMARY] KEY] [COMMENT 'string']
-- 表选项
	-- 字符集
        CHARSET = charset_name
        如果表没有设定，则使用数据库字符集
    -- 存储引擎
        ENGINE = engine_name
        表在管理数据时采用的不同的数据结构，结构不同会导致处理方式、提供的特性操作等不同
        常见的引擎：InnoDB MyISAM Memory/Heap BDB Merge Example CSV MaxDB Archive
        不同的引擎在保存表的结构和数据时采用不同的方式
        MyISAM表文件含义：.frm表定义，.MYD表数据，.MYI表索引
        InnoDB表文件含义：.frm表定义，表空间数据和日志文件
        SHOW ENGINES -- 显示存储引擎的状态信息
        SHOW ENGINE 引擎名 {LOGS|STATUS} -- 显示存储引擎的日志或状态信息
    -- 自增起始数
    	AUTO_INCREMENT = 行数
    -- 数据文件目录
        DATA DIRECTORY = '目录'
    -- 索引文件目录
        INDEX DIRECTORY = '目录'
    -- 表注释
        COMMENT = 'string'
    -- 分区选项
        PARTITION BY ... (详细见手册)
-- 查看所有表
    SHOW TABLES[ LIKE 'pattern']
    SHOW TABLES FROM  库名
-- 查看表结构
    SHOW CREATE TABLE 表名 （信息更详细）
    DESC 表名 / DESCRIBE 表名 / EXPLAIN 表名 / SHOW COLUMNS FROM 表名 [LIKE 'PATTERN']
    SHOW TABLE STATUS [FROM db_name] [LIKE 'pattern']
-- 修改表
    -- 修改表本身的选项
        ALTER TABLE 表名 表的选项
        eg: ALTER TABLE 表名 ENGINE=MYISAM;
    -- 对表进行重命名
        RENAME TABLE 原表名 TO 新表名
        RENAME TABLE 原表名 TO 库名.表名 （可将表移动到另一个数据库）
        -- RENAME可以交换两个表名
    -- 修改表的字段机构（13.1.2. ALTER TABLE语法）
        ALTER TABLE 表名 操作名
        -- 操作名
            ADD[ COLUMN] 字段定义       -- 增加字段
                AFTER 字段名          -- 表示增加在该字段名后面
                FIRST               -- 表示增加在第一个
            ADD PRIMARY KEY(字段名)   -- 创建主键
            ADD UNIQUE [索引名] (字段名)-- 创建唯一索引
            ADD INDEX [索引名] (字段名) -- 创建普通索引
            DROP[ COLUMN] 字段名      -- 删除字段
            MODIFY[ COLUMN] 字段名 字段属性     -- 支持对字段属性进行修改，不能修改字段名(所有原有属性也需写上)
            CHANGE[ COLUMN] 原字段名 新字段名 字段属性      -- 支持对字段名修改
            DROP PRIMARY KEY    -- 删除主键(删除主键前需删除其AUTO_INCREMENT属性)
            DROP INDEX 索引名 -- 删除索引
            DROP FOREIGN KEY 外键    -- 删除外键
-- 删除表
    DROP TABLE[ IF EXISTS] 表名 ...
-- 清空表数据
	TRUNCATE [TABLE] 表名
-- 复制表结构
    CREATE TABLE 表名 LIKE 要复制的表名
-- 复制表结构和数据
    CREATE TABLE 表名 [AS] SELECT * FROM 要复制的表名
-- 检查表是否有错误
    CHECK TABLE tbl_name [, tbl_name] ... [option] ...
-- 优化表
    OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...
-- 修复表
    REPAIR [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ... [QUICK] [EXTENDED] [USE_FRM]
-- 分析表
    ANALYZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...
```

### 2.3、数据操作

```sql
-- 增
INSERT [INTO] 表名 [(字段列表)] VALUES (值列表)[, (值列表), ...]
	-- 如果要插入的值列表包含所有字段并且顺序一致，则可以省略字段列表。
	-- 可同时插入多条数据记录！
	REPLACE与INSERT类似，唯一的区别是对于匹配的行，现有行（与主键/唯一键比较）的数据会被替换，如果没有现有行，则插入新行。
INSERT [INTO] 表名 SET 字段名=值[, 字段名=值, ...]
-- 查
SELECT 字段列表 FROM 表名[ 其他子句]
	-- 可来自多个表的多个字段
	-- 其他子句可以不使用
	-- 字段列表可以用*代替，表示所有字段
-- 删
DELETE FROM 表名[ 删除条件子句]
没有条件子句，则会删除全部
-- 改
UPDATE 表名 SET 字段名=新值[, 字段名=新值] [更新条件]
```

### 2.4、SELECT

```sql
SELECT [ALL|DISTINCT] select_expr FROM -> WHERE -> GROUP BY [合计函数] -> HAVING -> ORDER BY -> LIMIT
a. select_expr
    -- 可以用 * 表示所有字段。
        select * from tb;
    -- 可以使用表达式（计算公式、函数调用、字段也是个表达式）
        select stu, 29+25, now() from tb;
    -- 可以为每个列使用别名。适用于简化列标识，避免多个列标识符重复。
        - 使用 as 关键字，也可省略 as.
        select stu+10 as add10 from tb;
b. FROM 子句
    用于标识查询来源。
    -- 可以为表起别名。使用as关键字。
        SELECT * FROM tb1 AS tt, tb2 AS bb;
    -- from子句后，可以同时出现多个表。
        -- 多个表会横向叠加到一起，而数据会形成一个笛卡尔积。
        SELECT * FROM tb1, tb2;
    -- 向优化符提示如何选择索引
        USE INDEX、IGNORE INDEX、FORCE INDEX
        SELECT * FROM table1 USE INDEX (key1,key2) WHERE key1=1 AND key2=2 AND key3=3;
        SELECT * FROM table1 IGNORE INDEX (key3) WHERE key1=1 AND key2=2 AND key3=3;
c. WHERE 子句
    -- 从from获得的数据源中进行筛选。
    -- 整型1表示真，0表示假。
    -- 表达式由运算符和运算数组成。
        -- 运算数：变量（字段）、值、函数返回值
        -- 运算符：
            =, <=>, <>, !=, <=, <, >=, >, !, &&, ||,
            in (not) null, (not) like, (not) in, (not) between and, is (not), and, or, not, xor
            is/is not 加上ture/false/unknown，检验某个值的真假
            <=>与<>功能相同，<=>可用于null比较
d. GROUP BY 子句, 分组子句
    GROUP BY 字段/别名 [排序方式]
    分组后会进行排序。升序：ASC，降序：DESC
    以下[合计函数]需配合 GROUP BY 使用：
    count 返回不同的非NULL值数目  count(*)、count(字段)
    sum 求和
    max 求最大值
    min 求最小值
    avg 求平均值
    group_concat 返回带有来自一个组的连接的非NULL值的字符串结果。组内字符串连接。
e. HAVING 子句，条件子句
    与 where 功能、用法相同，执行时机不同。
    where 在开始时执行检测数据，对原数据进行过滤。
    having 对筛选出的结果再次进行过滤。
    having 字段必须是查询出来的，where 字段必须是数据表存在的。
    where 不可以使用字段的别名，having 可以。因为执行WHERE代码时，可能尚未确定列值。
    where 不可以使用合计函数。一般需用合计函数才会用 having
    SQL标准要求HAVING必须引用GROUP BY子句中的列或用于合计函数中的列。
f. ORDER BY 子句，排序子句
    order by 排序字段/别名 排序方式 [,排序字段/别名 排序方式]...
    升序：ASC，降序：DESC
    支持多个字段的排序。
g. LIMIT 子句，限制结果数量子句
    仅对处理好的结果进行数量限制。将处理好的结果的看作是一个集合，按照记录出现的顺序，索引从0开始。
    limit 起始位置, 获取条数
    省略第一个参数，表示从索引0开始。limit 获取条数
h. DISTINCT, ALL 选项
    distinct 去除重复记录
    默认为 all, 全部记录
```

### 2.5、UNION

```sql
-- 将多个select查询的结果组合成一个结果集合。
SELECT ... UNION [ALL|DISTINCT] SELECT ...
-- 默认 DISTINCT 方式，即所有返回的行都是唯一的；建议，对每个SELECT查询加上小括号包裹。
ORDER BY 排序时，需加上 LIMIT 进行结合。
-- 需要各select查询的字段数量一样。
-- 每个select查询的字段列表(数量、类型)应一致，因为结果中的字段名以第一条select语句为准。
```

### 2.6、子查询

```sql
    - 子查询需用括号包裹。
-- from型
    from后要求是一个表，必须给子查询结果取个别名。
    - 简化每个查询内的条件。
    - from型需将结果生成一个临时表格，可用以原表的锁定的释放。
    - 子查询返回一个表，表型子查询。
    select * from (select * from tb where id>0) as subfrom where id>1;
-- where型
    - 子查询返回一个值，标量子查询。
    - 不需要给子查询取别名。
    - where子查询内的表，不能直接用以更新。
    select * from tb where money = (select max(money) from tb);
    -- 列子查询
        如果子查询结果返回的是一列。
        使用 in 或 not in 完成查询
        exists 和 not exists 条件
            如果子查询返回数据，则返回1或0。常用于判断条件。
            select column1 from t1 where exists (select * from t2);
    -- 行子查询
        查询条件是一个行。
        select * from t1 where (id, gender) in (select id, gender from t2);
        行构造符：(col1, col2, ...) 或 ROW(col1, col2, ...)
        行构造符通常用于与对能返回两个或两个以上列的子查询进行比较。
    -- 特殊运算符
    != all()    相当于 not in
    = some()    相当于 in。any 是 some 的别名
    != some()   不等同于 not in，不等于其中某一个。
    all, some 可以配合其他运算符一起使用。
```

### 2.7、连接查询

```sql
    将多个表的字段进行连接，可以指定连接条件。
-- 内连接(inner join)
    - 默认就是内连接，可省略inner。
    - 只有数据存在时才能发送连接。即连接结果不能出现空行。
    on 表示连接条件。其条件表达式与where类似。也可以省略条件（表示条件永远为真）
    也可用where表示连接条件。
    还有 using, 但需字段名相同。 using(字段名)
    -- 交叉连接 cross join
        即，没有条件的内连接。
        select * from tb1 cross join tb2;
-- 外连接(outer join)
    - 如果数据不存在，也会出现在连接结果中。
    -- 左外连接 left join
        如果数据不存在，左表记录会出现，而右表为null填充
    -- 右外连接 right join
        如果数据不存在，右表记录会出现，而左表为null填充
-- 自然连接(natural join)
    自动判断连接条件完成连接。
    相当于省略了using，会自动查找相同字段名。
    natural join
    natural left join
    natural right join
select info.id, info.name, info.stu_num, extra_info.hobby, extra_info.sex from info, extra_info where info.stu_num = extra_info.stu_id;
```

## 三、数据类型

### 3.1、数值类型

- 整形：`TINYINT`、`SMALLINT`、`MEDIUMINT`、`INT/INTEGER`、`BIGINT`分别使用8位、16位、24位、32位、64位存储空间，一般情况下越小的列越好。
- 浮点数：FLOAT和DOUBLE位浮点类型，DECIMAL为高精度小数类型。CPU原生支持浮点运算，但是不支持DECIMAL类型的计算，因此DECIMAL的计算比浮点类型需要更高的代价。FLOAT、DOUBLE 和 DECIMAL 都可以指定列宽，例如 DECIMAL(18, 9) 表示总共 18 位，取 9 位存储小数部分，剩下 9 位存储整数部分。

| 类型         | 大小                                     | 范围（有符号）                                               | 范围（无符号）                                               | 用途            |
| :----------- | :--------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :-------------- |
| TINYINT      | 1 Bytes                                  | (-128，127)                                                  | (0，255)                                                     | 小整数值        |
| SMALLINT     | 2 Bytes                                  | (-32 768，32 767)                                            | (0，65 535)                                                  | 大整数值        |
| MEDIUMINT    | 3 Bytes                                  | (-8 388 608，8 388 607)                                      | (0，16 777 215)                                              | 大整数值        |
| INT或INTEGER | 4 Bytes                                  | (-2 147 483 648，2 147 483 647)                              | (0，4 294 967 295)                                           | 大整数值        |
| BIGINT       | 8 Bytes                                  | (-9,223,372,036,854,775,808，9 223 372 036 854 775 807)      | (0，18 446 744 073 709 551 615)                              | 极大整数值      |
| FLOAT        | 4 Bytes                                  | (-3.402 823 466 E+38，-1.175 494 351 E-38)，0，(1.175 494 351 E-38，3.402 823 466 351 E+38) | 0，(1.175 494 351 E-38，3.402 823 466 E+38)                  | 单精度 浮点数值 |
| DOUBLE       | 8 Bytes                                  | (-1.797 693 134 862 315 7 E+308，-2.225 073 858 507 201 4 E-308)，0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) | 0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) | 双精度 浮点数值 |
| DECIMAL      | 对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2 | 依赖于M和D的值                                               | 依赖于M和D的值                                               | 小数值          |

### 3.2、字符串

主要有`CHAR`和`VARCHAR`两种类型，前者是定长的，后者是变长的。

VARCHAR这种变长类型能够节省空间，因为只需要存储必要的内容。但是在执行UPDATE时可能会使行变得比原来长，当超出一个页所能容纳的大小时，就要执行额外的操作。MyISAM会将行拆成不同的片段存储，而InnoDB则需要分裂页来使行放进页内。

VARCHAR会保留字符串末尾的空格，而CHAR会删除。

| CHAR       | 0-255 bytes           | 定长字符串                      |
| ---------- | --------------------- | ------------------------------- |
| VARCHAR    | 0-65535 bytes         | 变长字符串                      |
| TINYBLOB   | 0-255 bytes           | 不超过 255 个字符的二进制字符串 |
| TINYTEXT   | 0-255 bytes           | 短文本字符串                    |
| BLOB       | 0-65 535 bytes        | 二进制形式的长文本数据          |
| TEXT       | 0-65 535 bytes        | 长文本数据                      |
| MEDIUMBLOB | 0-16 777 215 bytes    | 二进制形式的中等长度文本数据    |
| MEDIUMTEXT | 0-16 777 215 bytes    | 中等长度文本数据                |
| LONGBLOB   | 0-4 294 967 295 bytes | 二进制形式的极大文本数据        |
| LONGTEXT   | 0-4 294 967 295 bytes | 极大文本数据                    |

### 3.3、日期和时间类型

- DATETIME：能够保存从 1001 年到 9999 年的日期和时间，精度为秒，使用8字节的存储空间。与时区无关。默认情况下，MySQL以一种可排序的、无歧义的格式显示 DATETIME值，例如“2008-01-16 22:37:08”，这是 ANSI 标准定义的日期和时间表示方法。
- TIMESTAMP：和 UNIX 时间戳相同，保存从 1970 年 1 月 1 日午夜(格林威治时间)以来的秒数，使用 4 个字节，只能表示从 1970 年 到 2038 年。它和时区有关，也就是说一个时间戳在不同的时区所代表的具体时间是不同的。

| 类型      | 大小 ( bytes) | 范围                                                         | 格式                | 用途                     |
| :-------- | :------------ | :----------------------------------------------------------- | :------------------ | :----------------------- |
| DATE      | 3             | 1000-01-01/9999-12-31                                        | YYYY-MM-DD          | 日期值                   |
| TIME      | 3             | '-838:59:59'/'838:59:59'                                     | HH:MM:SS            | 时间值或持续时间         |
| YEAR      | 1             | 1901/2155                                                    | YYYY                | 年份值                   |
| DATETIME  | 8             | '1000-01-01 00:00:00' 到 '9999-12-31 23:59:59'               | YYYY-MM-DD hh:mm:ss | 混合日期和时间值         |
| TIMESTAMP | 4             | '1970-01-01 00:00:01' UTC 到 '2038-01-19 03:14:07' UTC结束时间是第 **2147483647** 秒，北京时间 **2038-1-19 11:14:07**，格林尼治时间 2038年1月19日 凌晨 03:14:07 | YYYY-MM-DD hh:mm:ss | 混合日期和时间值，时间戳 |

### 3.4、选择优化的数据类型

- **更小的通常更好：**更小的数据类型通常更快，因为它们占用更少的磁盘、内存和CPU缓存，并且处理时需要的CPU周期也更少。
- **简单就好：**例如，整形比字符串操作代价更低；实用内建类型而不是字符串来存储日期和时间；用整形存储IP地址等。
- **尽量避免NULL：**如果查询中包含可为NULL的列，对MySQL来说更难优化，因为可为NULL的列使得索引、索引统计和值比较都更复杂。尽管把可为NULL的列改为NOT NULL带来的性能提升比较小，但如果计划在列上创建索引，就应该尽量避免设计成可为NULL的列。

### 3.5、VARCHAR和CHAR的区别？

首先，char类型的长度是`固定`的，varchar的长度是`可变`的。varchar节省了存储空间，对性能也有帮助。由于行是可变的，在UPDATE时可能会使行变得比原来更长，导致可能需要做额外的工作。如果一个行占用的空间增长，并且在页内没有更多的空间可以存储。**MyISAM会将行拆成不同的片段存储**；**InnoDB则需要分裂页来使行可以放进页内**。

**VARCHAR的使用场景：**字符串的最大长度比平均长度大很多；列的更新很少，所以碎片不是问题；使用了像UTF-8这样复杂的字符集，每个字符都使用不同的字节数进行存储。

当存储CHAR值时，MySQL会删除所有末尾空格。CHAR值会根据需要采用空格进行填充以方便比较。CHAR适合存储很短的字符串，或者所有值都接近同一个长度，如密码的MD5值。对于经常变更的数据，CHAR也比VARCHAR更好，因为CHAR不容易产生碎片。

### 3.6、VARCHAR（5）和VARCHAR（200）？

> 使用VARCHAR（5）和VARCHAR（200）存储“hello”的空间开销是一样的。那么使用更短的列有什么优势吗？

更长的列会消耗更多的内存，因为MySQL通常会分配固定大小的内存块来保存内存值。尤其是使用内存临时表进行排序或其他操作时会特别糟糕。在利用磁盘临时表进行排序时也同样糟糕。**最好的策略是只分配真正需要的空间**。

### 3.7、BLOB和TEXT的区别？

BLOB和TEXT都是为存储很大的数据而设计的数据类型，分别采用二进制和字符方式存储。

与其他类型不同，MySQL把每个BLOB和TEXT值当做一个独立的对象去处理。当BLOB和TEXT值太大时，InnoDB会使用专门的“外部”存储区域来进行存储，此时每个值在行内需要1~4个字节存储一个指针，然后在外部存储区域存储实际的值。

MySQL对BLOB和TEXT进行排序与其他类型是不同的：它只对每个列的最前max_sort_length个字节而不是整个字符串做排序。且MySQL也不能将BLOB或TEXT列全部长度的字符串进行索引。

## 四、存储引擎

```sql
SHOW ENGINES;
```

![image-20220729112435735](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207291124193.png)

### 4.1、InnoDB（默认的事务型存储引擎）

是MySQL默认的事务型存储引擎，**只有在需要它不支持的特性时，才考虑使用其它存储引擎**。

#### InnoDB的隔离级别

实现了四个标准的隔离级别，默认级别是`可重复读（REPEATABLE READ）`。通过`多版本并发控制（MVCC）`+`间隙锁（Next-Key Locking）`防止幻读。

- **读未提交（Read Uncommitted）：**允许事务A读取其他事务未提交的记录，`会发生脏读`、`不可重复读`、`幻读`。
- **读已提交（Read Commited）：**只允许事务A读取其他事务已提交的记录，`不会发生脏读`，但`会发生不可重复度`、`幻读`。
- **可重复读（Repeatable Read）：**事务A执行期间，多次相同查询结果一致，`不会发生脏读和不可重复读的问题，但会发生幻读的问题。`
- **可串行化（Serializable）：**即事务串行执行。

> MVCC：InnoDB中，每行数据有两个隐藏列，一列记录“创建行的 事务版本号”，一列记录“删除行的事务版本号”。
>
> select，读取的数据为 `create_version<=current_version && (del_version==null || current_version<del_version)`。
>
> insert，`create_version=current_version，del_version=current_version`。
>
> update，新插入一行，现有原数据行`del_version=current_version`与数据行`create_version=current_version`。
>
> delete，`del_version=current_version`。

> 间隙锁：不对具体的数据行加锁，而是对数据行之间的间隙加锁。

#### RR实现

1. 对于普通的select语句，基于MVCC实现。普通select使用快照读，同一事务内使用相同版本的数据快照（解决不可重复读）。
2. 对于带锁select和写操作，基于锁实现。通过Record Lock锁定某条记录，使用Gap Lock防止区间内插入新数据（解决幻读）。

![](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207291446045.png)

#### RC实现

1. 对于普通select语句，基于MVCC实现。普通select使用快照读，每次查询使用已提交的新版本数据快照（导致不可重复读）。
2. 对于带锁select和写操作，基于锁实现。通过Record Lock锁定某条记录，RC隔离级别中不存在Gap Lock（导致幻读）。

![image-20220729145636021](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207291456791.png)



主索引是`聚簇索引`，在索引中保存了数据，从而避免直接读取磁盘，因此对查询性能有很大的提升。

内部做了很多优化，包括从磁盘读取数据时采用的可预测性读、能够加快读操作并且自动创建的自适应哈希索引、能够加速插入操作的插入缓冲区等。

支持真正的在线热备份。其他存储引擎不支持在线热备份，要获取一致性视图需要停止对所有表的写入，而在读写混合场景中，停止写入可能也意味着停止读取。

### 4.2、MyISAM

设计简单，数据以紧密格式存储。对于只读数据，或者表比较小、可以容忍修复操作，则依然可以使用它。提供了大量的特性，包括压缩表、空间数据索引等。提供了大量的特性，包括压缩表、空间数据索引等。

> **不支持事务**

不支持行级锁，只能对整张表加锁，读取时会对需要读到的所有表加共享锁，写入时则对表加排它锁。但在表有读取操作的同时，也可以往表中插入新的记录，称为并发插入（CONCURRENT INSERT）。

### 4.3、InnoDB和MyISAM的区别？

- **事务：**InnoDB是事务型的，可以使用Commit和Rollback语句。
- **并发：**MyISAM只支持表级锁，而InnoDB还支持行级锁。
- **外键：**InnoDB支持外键。
- **备份：**InnoDB支持在线热备份
- **崩溃恢复：**MyISAM崩溃后发生损坏的概率比InnoDB高很多，而且恢复的速度也更慢。
- MyISAM压缩表和空间数据索引。

## 五、索引（B+树）

```sql
## 普通索引
	## 创建索引
	CREATE INDEX indexName ON table_name(columnName);
	##修改表结构（添加索引）
	ALTER table tableName ADD INDEX indexName(columnName);
	## 创建表的时候直接指定
	CREATE TBALE mytable(
    	ID INT NOT NULL,
        userName VARCHAR(16) NOT NULL,
        INDEX [indexName](username(length))
    )
    ## 删除索引
    DROP INDEX [indexName] ON mytable;
## 唯一索引
	## 创建索引
	CREATE UNIQUE INDEX indexName ON mytable(username(length));
	## 修改表结构
	ALTER TABLE mytable ADD UNIQUE [indexName](username(length));
	## 创建表的时候直接指定
	CREATE TABlE mytable(
    	ID INT NOT NULL,
        username VARCHAR(16) NOT NULL,
        UNIQUE [indexName](username(length))
    )
## 使用ALTER命令添加和删除索引
ALTER TABLE tbl_name ADD PRIMARY KEY(column_list);	该语句添加一个主键，这意味着索引值必须是唯一的，且不能为NULL。
ALTER TABLE tbl_name ADD UNIQUE index_name(column_list);	该语句创建索引的值必须是唯一的（除了NULL之外，NULL可能会出现多次）。
ALTER TABLE tbl_name ADD INDEX index_name(column_list);		添加普通索引，索引值可出现多次。
ALTER TABLE tbl_name ADD FULLTEXT index_name(column_list);	该语句指定了索引为FULLTEXT，用于全文索引。
## 显示索引信息
ALTER TABLE testalter_tbl DROP PRIMARY KEY;
```

### 5.1、索引类型

> **1、聚簇索引与非聚簇索引**

- **聚簇索引：**将数据存储和索引放到一起，都是按照一定的顺序组织，找到了索引也找到了数据，数据的物理存储和索引顺序是一致的。即：只要索引相连，那么对应的数据一定也是相邻存储在磁盘上的。

  **优势：**

  - 根据聚簇索引可以直接获取数据，相比非聚簇索引需要第二次查询效率要高。
  - 聚簇索引对于范围查询效率更高，因其数据是按大小排列的。
  - 聚簇索引适合用在排序的场合，非聚簇索引不适合。

  **劣势：**

  - 维护索引很昂贵，特别是插入新行或者主键被更新导致要分页的时候。建议在大量插入新行后，选在负载较低的时间段，通过OPTIMIZE TABLE优化表，因为必须被移动的行数据可能造成碎片。使用独享表空间可以弱化碎片。
  - 表因为使用UUID（随机ID）作为主键，使用数据存储稀疏，这就会出现聚簇索引可能比全表扫描更慢，所以建议使用int的auto_increment作为主键。
  - 如果主键比较大的时候，其辅助索引将会变得更大，因为辅助索引的叶子存储的是主键值，过长的主键值，会导致非叶子节点占用更多的物理内存。

- **非聚簇索引：**叶子节点不存储数据，存储的是数据行地址。根据索引查找到数据行的位置再去磁盘查找数据。

**InnoDB**中一定有主键，主键一定是`聚簇索引`。不手动设置，则会使用unique索引，没有unique索引，则会使用数据库内部一个行的隐藏id来当做主键索引。在聚簇索引之上创建的索引称之为辅助索引，辅助索引访问数据库总是需要二次查询。非聚簇索引都是辅助索引，像复合索引、前缀索引、唯一索引、辅助索引叶子节点存储的不再是行的物理位置，而是主键值。

聚簇索引的每一个叶子节点都包含了主键值、事务ID、用于事务和MVCC的回滚指针以及所有的剩余列。

**MyISAM**使用的是非聚簇索引，没有聚簇索引，非聚簇索引的两颗B+树看上去没有什么不同，节点的结构完全一致，知识存储的内容不同而已，主键索引B+树的节点存储了主键，辅助索引B+树存储了辅助键。表数据存储在独立的地方，这两颗B+树的叶子节点都使用一个地址指向真正的表数据，对于表数据来说，这两个键没有任何差别。由于索引树是独立的，通过辅助键索引无需访问主键的索引树。

### 5.2、B+Tree原理

> **1、数据结构**

B Tree指的是Balance Tree，是平衡树。平衡树是一颗查找树，并且`所有的叶子节点位于同一层`。是`基于B Tree和叶子节点顺序访问指针`进行实现，具有 B Tree 的平衡性，并且通过`顺序访问指针`来提高区间查询的性能。

在 B+ Tree中，一个节点中的 key 从左到右非递减排列，如果某个指针的左右相邻 key 分别是 key~i~和 key~i+1~，且不为null，则该指针指向节点的所有key大于等于key~i~且小于等于key~i+1~。

![image-20220729155212904](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207291552170.png)

> **2、操作**

进行查找操作时，首先在**根节点进行二分查找**，找到一个key所在的指针，然后`递归`地在指针所指向的节点进行查找。直到查找到`叶子节点`，然后在叶子节点上进行`二分查找`，找出 key 所对应的 data。

**插入删除**操作记录会破坏平衡树的平衡性，因此在插入删除操作之后，需要对树进行一个`分裂`、`合并`、`旋转`等操作来维护平衡性。

> **3、与红黑树的比较**

红黑树等平衡树也可以用来实现索引，但是文件系统及数据库系统普遍采用 B+ Tree 作为索引结构，主要有以下两个原因:

(一)更少的查找次数

平衡树查找操作的时间复杂度等于树高 h，而树高大致为 O(h)=O(logdN)，其中 d 为每个节点的出度。

红黑树的出度为 2，而 B+ Tree 的出度一般都非常大，所以红黑树的树高 h 很明显比 B+ Tree 大非常多，检索的次数也就更多。

(二)利用计算机预读特性

为了减少磁盘 I/O，磁盘往往不是严格按需读取，而是每次都会预读。预读过程中，磁盘进行顺序读取，顺序读取不需要进行磁盘寻道，并且只需要很短的旋转时间，因此速度会非常快。

操作系统一般将内存和磁盘分割成固态大小的块，每一块称为一页，内存与磁盘以页为单位交换数据。数据库系统将索引的一个节点的大小设置为页的大小，使得一次 I/O 就能完全载入一个节点，并且可以利用预读特性，相邻的节点也能够被预先载入。

### 5.3、MySQL索引

索引是在存储引擎层实现的，而不是在服务器层实现的，因此不同存储引擎具有不同的索引类型和实现。

> **1、B+Tree索引**

大多数 MySQL 存储引擎的默认索引类型。不需要进行全表扫描，只需要对树进行搜索即可，查找速度会快很多。除了用于查找，还可以用于排序和分组。

可以指定多个列作为索引列，多个索引列共同组成键。适用于全键值、键值范围和键前缀查找，其中键前缀查找只适用于最左前缀查找。如果不是按照索引列的顺序进行查找，则无法进行索引。

**InnoDB的 B+ Tree 索引分为主索引和辅助索引**

主索引的叶子节点 data 域记录着完整的数据记录，这种索引方式被称为聚簇索引。因为无法把数据行存放在两个不同的地方，所以一个表只能有一个聚簇索引。

![image-20220729161713974](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207291617744.png)

辅助索引的叶子节点的 data 域记录着主键的值，因此在使用辅助索引进行查找时，需要先查找到主键值，然后再到主索引中进行查找。

![image-20220729161838350](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207291620889.png)

> **2、哈希索引**

哈希索引能以 O(1) 时间进行查找，但是失去了有序性，具有以下限制：

- 无法用于排序与分组
- 只支持精确查找，无法用于部分查找和范围查找。

InnoDB存储引擎有一个特殊的功能叫“自适应哈希索引”，当某个索引值被使用的非常频繁时，会在 B+ Tree索引之上再创建一个哈希索引，这样就让 B+ Tree 索引具有哈希索引的一些优点，比如快速的哈希查找。

> **3、全文索引**

MyISAM 存储引擎支持全文索引，用于查找文本中的关键词，而不是直接比较是否相等。查找条件使用 MATCH AGAINST，而不是普通的 WHERE。

全文索引一般使用倒排索引实现，它记录着关键词到其所在文档的映射。

InnoDB 存储引擎在 MySQL 5.6.4 版本中也开始支持全文索引。

> **4、空间数据索引**

MyISAM 存储引擎支持空间数据索引(R-Tree)，可以用于地理数据存储。空间数据索引会从所有维度来索引数据，可以有效地使用任意维度来进行组合查询。

必须使用 GIS 相关的函数来维护数据。

### 5.4、索引优化

> **1、独立的列**

在进行查询的时候，索引列不能是表达式的一部分也不能是函数的参数，否则无法使用索引。

例如下面的查询不能使用 actor_id 列的索引:

```sql
SELECT actor_id FROM sakila.actor WHERE actor_id + 1 = 5;
```

> **2、多列索引**

在需要使用多个列作为条件进行查询时，使用多列索引比使用多个单例索引性能更好。例如下面的语句中，将 actor_id 和 film_id 设置为多列索引。

```sql
SELECT film_id, actor_ id FROM sakila.film_actor WHERE actor_id = 1 AND film_id = 1;
```

> **3、索引列的顺序**

让选择性强的索引列放在前面，索引的选择性是指：不重复的索引值和记录总数的比值。最大值为1，此时每个记录都有唯一的索引与其对应。选择性越高，查询效率也越高。

例如下面显示的结果中 customer_id 的选择性比 staff_id 更高，因此最好把 customer_id 列放在多列索引的前面。

```sql
SELECT COUNT(DISTINCT staff_id)/COUNT(*) AS staff_id_selectivity,
COUNT(DISTINCT customer_id)/COUNT(*) AS customer_id_selectivity,
COUNT(*)
FROM payment;
```

> **4、前缀索引**

对于 BLOB、TEXT 和 VARCHAR 类型的列，必须使用前缀索引，只索引开始的部分字符。

对于前缀长度的选取需要根据索引选择性来确定。

> **5、覆盖索引**

索引包含所有需要查询的字段的值。

具有以下优点:

- 索引通常远小于数据行的大小，只读取索引能大大减少数据访问量。
- 一些存储引擎(例如 MyISAM)在内存中只缓存索引，而数据依赖于操作系统来缓存。因此，只访问索引可以不使用系统调用(通常比较费时)。
- 对于 InnoDB 引擎，若辅助索引能够覆盖查询，则无需访问主索引。

### 5.5、索引的优点

- 大大减少了服务器需要扫描的数据行数
- 帮助服务器避免进行排序和分组，也就不需要创建临时表（B+ Tree 索引是有序的，可以用于 Order By 和 Group By 操作。临时表主要是在排序和分组过程中创建，因为不需要排序和分组，也就不需要创建临时表。）
- 将随机 I/O 变为顺序 I/O（B+ Tree 索引是有序的，也就将相邻的数据都存储在一起）。

### 5.6、索引的使用场景

- 对于非常小的表、大部分情况下简单的全表扫描比建立索引更高效。
- 对于中到大型的表，索引就非常有效。
- 但是对于特大型的表、建立和维护索引的代价将随之增长。这种情况下，需要用到一种技术可以直接区分出需要查询的一组数据，而不是一条记录一条记录地匹配。可以使用分区技术。
