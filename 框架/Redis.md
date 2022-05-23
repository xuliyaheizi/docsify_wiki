# Redis

## 一、NoSql

NoSQL(NoSQL = **Not Only SQL** )，意即“不仅仅是 SQL”，泛指**非关系型的数据库**。NoSQL 不依赖业务逻辑方式存储，而以简单的 `key-value` 模式存储。因此大大的增加了数据库的扩展能力。

- 不遵循 SQL 标准。 
- 不支持 ACID。 
- 远超于 SQL 的性能。

适用场景

- 对数据高并发的读写 
- 海量数据的读写 
- 对数据高可扩展性的

## 二、Redis

Redis 是一个开源的 `key-value 存储系统`。和 `Memcached `类似，它支持存储的 value 类型相对更多，包括 string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和 hash（哈希类型）。这些数据类型都支持 `push/pop、add/remove` 及取交集并集和差集及更丰富的操作，而且这些操作都是**原子性**的。在此基础上，Redis 支持各种不同方式的排序。与 memcached 一样，为了保证效率，数据都是缓存在内存中。区别的是 Redis 会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件。并且在此基础上实现了`master-slave(主从)同步`。

**配合关系型数据库做高速缓存**

- 高频次，热门访问的数据，降低数据库 IO 
- 分布式架构，做 session 共享

**多样的数据结构存储持久化数据**

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220521193155461.png" alt="image-20220521193155461" style="zoom:67%;" />

**redis介绍**

- 默认 16 个数据库，类似数组下标从 0 开始，初始默认使用 0 号库 
- 使用命令 select <dbid>来切换数据库。如: select 8 
- 统一密码管理，所有库同样密码。 
- dbsize 查看当前数据库的 key 的数量 
- flushdb 清空当前库 
- flushall 通杀全部库

Redis 是**单线程+多路 IO 复用技术** 

`多路复用`是指使用一个线程来检查多个文件描述符（Socket）的就绪状态，比如调用select 和 poll 函数，传入多个文件描述符，如果有一个文件描述符就绪，则返回，否则阻塞直到超时。得到就绪状态后进行真正的操作可以在同一个线程里执行，也可以启动线程执行（比如使用线程池） 串行 vs 多线程+锁（memcached） vs 单线程+多路 IO 复用(Redis)。

（与 Memcache 三点不同: 支持多数据类型，支持持久化，单线程+多路 IO 复用）

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220521194217299.png" alt="image-20220521194217299" style="zoom:67%;" />

## 三、常用五大数据类型

### Redis键（key）

```tex
keys *			查看当前库所有 key (匹配：keys *1) 
exists key		判断某个 key 是否存在 
type key		  查看你的 key 是什么类型 
del key			删除指定的 key 数据 
unlink key		根据 value		选择非阻塞删除		仅将 keys 从 keyspace 元数据中删除，真正的删除会在后续异步操作。 
expire key 10	10 秒钟：为给定的 key 设置过期时间 
ttl key			查看还有多少秒过期，-1 表示永不过期，-2 表示已过期 
select			 命令切换数据库 
dbsizex			查看当前数据库的 key 的数量 
flushdb			清空当前库 
flushall		  通杀全部库 
```

### Redis字符串（String）

- String 是 Redis 最基本的类型，你可以理解成与 Memcached 一模一样的类型，一个 key对应一个 value。 
- String 类型是**二进制安全**的。意味着 Redis 的 string 可以**包含任何数据**。比如 jpg 图片或者序列化的对象。 
- String 类型是 Redis 最基本的数据类型，一个 Redis 中字符串 value 最多可以是 **512M**

```tex
set <key><value>							添加键值对
get <key>				                 	查询对应键值 
append <key><value>				         	将给定的<value> 追加到原值的末尾 
strlen <key>					  			获得值的长度 
setnx <key><value>							只有在 key 不存在时 设置 key 的值
incr <key> 			                     	将 key 中储存的数字值增 1 只能对数字值操作，如果为空，新增值为 1 
decr <key> 				      		    	将 key 中储存的数字值减 1 只能对数字值操作，如果为空，新增值为-1 
incrby / decrby <key><步长>		  	   	将 key 中储存的数字值增减。自定义步长。
mset <key1><value1><key2><value2> .....	  	同时设置一个或多个 key-value 对 
mget <key1><key2><key3> .....			 	同时获取一个或多个 value 
msetnx <key1><value1><key2><value2> ..... 	同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。
			原子性，有一个失败则都失败
getrange <key><起始位置><结束位置> 	       	获得值的范围，类似 java 中的 substring，前包，后包 
setrange <key><起始位置><value> 		  	用 <value> 覆写<key>所储存的字符串值，从<起始位置>开始(索引从 0 开始)。 
setex <key><过期时间><value> 		      	设置键值的同时，设置过期时间，单位秒。 
getset <key><value> 				     	以新换旧，设置了新值同时获得旧值。
```

**原子性** 

所谓**原子**操作是指不会被线程调度机制打断的操作；这种操作一旦开始，就一直运行到结束，中间不会有任何 context switch （切换到另一个线程）。 

- 在单线程中， 能够在单条指令中完成的操作都可以认为是"原子操作"，因为中断只能发生于指令之间。 
- 在多线程中，不能被其它进程（线程）打断的操作就叫原子操作。Redis 单命令的原子性主要得益于 Redis 的单线程。 

**案例：** 

java 中的 i++是否是原子操作？**不是** 

i=0;两个线程分别对 i 进行++100 次,值是多少？ **2~200**

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220522112446066.png" alt="image-20220522112446066" style="zoom:67%;" />

#### 数据结构

String 的数据结构为`简单动态字符串`(Simple Dynamic String,缩写 SDS)。是可以修改的字符串，内部结构实现上类似于 Java 的 `ArrayList`，采用**预分配冗余空间**的方式来减少内存的频繁分配.

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220522112539492.png" alt="image-20220522112539492" style="zoom:67%;" />

如图中所示，内部为当前字符串实际分配的空间 `capacity `一般要**高于实际字符串长度len**。当字符串长度小于 1M 时，扩容都是加倍现有的空间，如果超过 1M，扩容时一次只会多扩 1M 的空间。需要注意的是字符串最大长度为 `512M`。

### Redis链表（List）

单键多值Redis 

链表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。它的底层实际是个双向链表，对两端的操作性能很高，通过索引下标的操作中间的节点性能会较差。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220522112807853.png" alt="image-20220522112807853" style="zoom:67%;" />

```
lpush/rpush <key><value1><value2><value3> .... 	   从左边/右边插入一个或多个值。 
lpop/rpop <key>									从左边/右边吐出一个值。值在键在，值光键亡。 
rpoplpush <key1><key2>						     从<key1>列表右边吐出一个值，插到<key2>列表左边。 
lrange <key><start><stop> 						 按照索引下标获得元素(从左到右) 
lrange mylist 0 -1 						         0 左边第一个，-1 右边第一个，（0-1 表示获取所有） 
lindex <key><index>						         按照索引下标获得元素(从左到右) 
llen <key>								        获得列表长度 
linsert <key> before <value><newvalue>			  在<value>的后面插入<newvalue>插入值 
lrem <key><n><value>			                  从左边删除 n 个 value(从左到右) 
lset<key><index><value>					    	 将列表 key 下标为 index 的值替换成 value
```

#### 数据结构

List 的数据结构为`快速链表 quickList`。首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构是 `ziplist`，也即是**压缩列表**。它将所有的元素紧挨着一起存储，分配的是一块连续的内存。当数据量比较多的时候才会改成 quicklist。因为普通的链表需要的附加指针空间太大，会比较浪费空间。比如这个列表里存的只是 int 类型的数据，结构上还需要两个额外的指针 prev 和 next。 

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220522113324227.png" alt="image-20220522113324227" style="zoom:67%;" />

Redis 将链表和 ziplist 结合起来组成了 quicklist。也就是将多个 ziplist 使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。

### Redis集合（Set）

Redis set 对外提供的功能与 list 类似是一个列表的功能，特殊之处在于 set 是可以**自动排重**的，当你需要存储一个列表数据，又不希望出现重复数据时，set 是一个很好的选择，并且 set 提供了判断某个成员是否在一个 set 集合内的重要接口，这个也是 list 所不能提供的。Redis 的 Set 是 string 类型的无序集合。它底层其实是一个 value 为 null 的 hash 表，所以添加，删除，查找的**复杂度都是** **O(1)**。一个算法，随着数据的增加，执行时间的长短，如果是 O(1)，数据增加，查找数据的时间不变 

```tex
sadd <key><value1><value2> .....        将一个或多个 member 元素加入到集合 key 中，已经存在的 member 元素将被忽略 
smembers <key>				           取出该集合的所有值。 
sismember <key><value>			       判断集合<key>是否为含有该<value>值，有 1，没有 0 
scard<key>			                   返回该集合的元素个数。
srem <key><value1><value2> .... 	    删除集合中的某个元素。 
spop <key>				               随机从该集合中吐出一个值。 
srandmember <key><n>		            随机从该集合中取出 n 个值。不会从集合中删除 。 
smove <source><destination>value         把集合中一个值从一个集合移动到另一个集合 
sinter <key1><key2>			            返回两个集合的交集元素。 
sunion <key1><key2>			            返回两个集合的并集元素。 
sdiff <key1><key2>			            返回两个集合的差集元素(key1 中的，不包含 key2 中的)
```

#### 数据结构

Set 数据结构是 `dict 字典`，字典是用`哈希表`实现的。Java 中 HashSet 的内部实现使用的是 `HashMap`，只不过所有的 value 都指向同一个对象。Redis 的 set 结构也是一样，它的内部也使用 hash 结构，所有的 value 都指向同一个内部值

### Redis哈希（Hash）

- Redis hash 是一个键值对集合。 
- Redis hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。 
- 类似 Java 里面的 Map<String,Object> 
- 用户 ID 为查找的 key，存储的 value 用户对象包含姓名，年龄，生日等信息，如果用普通的 key/value 结构来存储 

主要有以下 2 种存储方式：

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220522113735993.png" alt="image-20220522113735993" style="zoom:67%;" />

```tex
hset <key><field><value>			  	          给<key>集合中的 <field>键赋值<value> 
hget <key1><field>					  		     从<key1>集合<field>取出 value 
hmset <key1><field1><value1><field2><value2>...    批量设置 hash 的值 
hexists<key1><field>				       		 查看哈希表 key 中，给定域 field 是否存在。 
hkeys <key>						           	     列出该 hash 集合的所有 field 
hvals <key>							             列出该 hash 集合的所有 value 
hincrby <key><field><increment>				      为哈希表 key 中的域 field 的值加上增量 1 -1 
hsetnx <key><field><value>						 将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在 .
```

#### 数据结构

Hash 类型对应的数据结构是两种：ziplist（压缩列表），hashtable（哈希表）。当field-value 长度较短且个数较少时，使用 ziplist，否则使用 hashtable。

### Redis有序集合Zset（sorted set）

Redis **有序集合 zset 与普通集合 set** 非常相似，是一个没有重复元素的字符串集合。不同之处是有序集合的每个成员都关联了一个**评分（score）**,这个评分（score）被用来按照从`最低分到最高分`的方式排序集合中的成员。集合的成员是唯一的，但是评分可用是重复的 。因为元素是`有序`的, 所以你也可以很快的根据评分（score）或者次序（position）来获取一个范围的元素。访问有序集合的中间元素也是非常快的,因此你能够使用有序集合作为一个没有重复成员的智能列表。

```
zadd <key><score1><value1><score2><value2>… 	将一个或多个 member 元素及其 score 值加入到有序集 key 当中。 
zrange <key><start><stop> [WITHSCORES] 	        
返回有序集 key 中，下标在<start><stop>之间的元素 带 WITHSCORES，可以让分数一起和值返回到结果集。 

zrangebyscore key minmax [withscores] [limit offset count] 				
返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。 有序集成员按 score 值递增(从小到大)次序排列。 

zrevrangebyscore key maxmin [withscores] [limit offset count] 同上，改为从大到小排列。
zincrby <key><increment><value> 		       	为元素的 score 加上增量 
zrem <key><value>			                   	删除该集合下，指定值的元素 
zcount <key><min><max>					      	统计该集合，分数区间内的元素个数 
zrank <key><value>							 	返回该值在集合中的排名，从 0 开始。
```

#### 数据结构

SortedSet(zset)是 Redis 提供的一个非常特别的数据结构，一方面它等价于 Java的数据结构 Map<String, Double>，可以给每一个元素 value 赋予一个权重 score，另一方面它又类似于 TreeSet，内部的元素会按照权重 score 进行排序，可以得到每个元素的名次，还可以通过 score 的范围来获取元素的列表。 

zset 底层使用了两个数据结构 

- hash，hash的作用就是**关联元素 value 和权重 score**，保障元素 value 的`唯一性`，可以通过元素 value 找到相应的 score 值。 
- 跳跃表，跳跃表的目的在于给元素 value 排序，根据 score 的范围获取元素列表。

#### 跳跃表（跳表）

有序集合在生活中比较常见，例如根据成绩对学生排名，根据得分对玩家排名等。对于有序集合的底层实现，可以用数组、平衡树、链表等。数组不便元素的插入、删除；平衡树或红黑树虽然效率高但结构复杂；链表查询需要遍历所有效率低。Redis采用的是跳跃表。跳跃表效率堪比红黑树，实现远比红黑树简单。

2、实例

对比有序链表和跳跃表，从链表中查询出 51 

（1） 有序链表 

要查找值为 51 的元素，需要从第一个元素开始依次查找、比较才能找到。共需要 6 次比较。 

（2） 跳跃表 

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220522114511831.png" alt="image-20220522114511831" style="zoom:67%;" />

- 从第 2 层开始，1 节点比 51 节点小，向后比较。21 节点比 51 节点小，继续向后比较，后面就是 NULL 了，所以从 21 节点向下到第 1 层 
- 在第 1 层，41 节点比 51 节点小，继续向后，61 节点比 51 节点大，所以从 41 向下 
- 在第 0 层，51 节点为要查找的节点，节点被找到，共查找 4 次。 
- 从此可以看出跳跃表比有序链表效率要高 

## 四、Redis的发布与订阅

Redis 发布订阅 (pub/sub) 是一种**消息通信模式**：发送者 (pub) 发送消息，订阅者(sub) 接收消息。Redis 客户端可以订阅任意数量的频道。 

客户端可以订阅频道如下图 

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220522121140826.png" alt="image-20220522121140826" style="zoom:67%;" />

当给这个频道发布消息后，消息就会发送给订阅的客户端

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220522121201473.png" alt="image-20220522121201473" style="zoom:67%;" />

```
SUBSCRIBE channel1		 客户端订阅 channel1
publish channel1 hello	 另一个客户端，给 channel1 发布消息 hello
发布的消息没有持久化，如果在订阅的客户端收不到 hello，只能收到订阅后发布 的消息
```

## 五、Redis新数据类型

### Bitmaps

现代计算机用二进制（位） 作为信息的基础单位， 1 个字节等于 8 位， 例如“abc” 字符串是由 3 个字节组成， 但实际在计算机存储时将其用二进制表示， “abc”分别对应的 ASCII 码分别是 97、 98、 99， 对应的二进制分别是 01100001、 01100010和 01100011，如下图 

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220522184731379.png" alt="image-20220522184731379" style="zoom:67%;" />

合理地使用操作位能够有效地提高内存使用率和开发效率。Redis 提供了 Bitmaps 这个“数据类型”可以实现对位的操作： 

- Bitmaps 本身不是一种数据类型， 实际上它就是字符串（key-value） ，但是它可以对字符串的位进行操作。 
- Bitmaps 单独提供了一套命令， 所以在 Redis 中使用 Bitmaps 和使用字符串的方法不太相同。 可以把 Bitmaps 想象成一个以位为单位的数组，数组的每个单元只能存储 0 和 1， 数组的下标在 Bitmaps 中叫做偏移量。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220522184824449.png" alt="image-20220522184824449" style="zoom:67%;" />

```
setbit<key><offset><value>			设置 Bitmaps 中某个偏移量的值（0 或 1）
注： 
很多应用的用户 id 以一个指定数字（例如 10000） 开头， 直接将用户 id 和 Bitmaps 的偏移量对应势必会造成一定的浪费， 通常的做法是每次做 setbit 操作时将 用户 id 减去这个指定数字。 在第一次初始化 Bitmaps 时， 假如偏移量非常大， 那么整个初始化过程执行会 比较慢， 可能会造成 Redis 的阻塞。

getbit<key><offset>					获取 Bitmaps 中某个偏移量的值

bitcount<key>[start end] 			统计字符串从 start 字节到 end 字节比特值为 1 的数量
统计字符串被设置为 1 的 bit 数。一般情况下，给定的整个字符串都会被进行计数，通 过指定额外的 start 或 end 参数，可以让计数只在特定的位上进行。start 和 end 参数 的设置，都可以使用负数值：比如 -1 表示最后一个位，而 -2 表示倒数第二个位， start、end 是指 bit 组的字节的下标数，二者皆包含。

bitop and(or/not/xor) <destkey> [key…]
bitop 是一个复合操作， 它可以做多个 Bitmaps 的 and（交集） 、 or（并集） 、 not （非） 、 xor（异或） 操作并将结果保存在 destkey 中。
```

### HyperLogLog

在工作当中，我们经常会遇到与统计相关的功能需求，比如统计网站 PV（PageView 页面访问量）,可以使用 Redis 的 incr、incrby 轻松实现。 但像 UV（UniqueVisitor，独立访客）、独立 IP 数、搜索记录数等需要去重和计数的问题如何解决？这种求集合中不重复元素个数的问题称为基数问题。 解决基数问题有很多种方案： 

- 数据存储在 MySQL 表中，使用 distinct count 计算不重复个数 
- 使用 Redis 提供的 hash、set、bitmaps 等数据结构来处理 

以上的方案结果精确，但随着数据不断增加，导致占用空间越来越大，对于非常大的数据集是不切实际的。能否能够降低一定的精度来平衡存储空间？Redis 推出了 HyperLogLog Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定的、并且是很小的。 

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。 

什么是基数? 

比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 

基数(不重复元素)为 5。 基数估计就是在误差可接受的范围内，快速计算基数。

```
pfadd <key>< element> [element ...] 			    添加指定元素到 HyperLogLog 中
执行命令后 HLL 估计的 近似基数发生变化，则返回 1，否则返回 0。
pfcount<key> [key ...] 						       计算 HLL 的近似基数，可以计算多个 HLL，比如用 HLL 存储每 天的 UV，计算一周的 UV 可以使用 7 天的 UV 合并计算即可
pfmerge<destkey><sourcekey> [sourcekey ...] 		将一个或多个 HLL 合并后的结果存 储在另一个 HLL 中，比如每月活跃用户可以使用每天的活跃用户来合并计算可得
```

## 六、Jedis测试

**Redis数据库连接**

```java
public class Connect {
    private static final String REDIS_URL = "zhulinz.top";
    private static final int PORT = 4030;
    private static final String PASSWORD = "123456";

    private static Jedis jedis;

    /**
     * redis数据库连接
     */
    public static Jedis connect() {
        jedis = new Jedis(REDIS_URL, PORT);
        jedis.auth(PASSWORD);
        //连接成功
        if (jedis.isConnected()) {
            return jedis;
        }
        return null;
    }

    public static void close() {
        jedis.close();
    }
}
```

**发送验证码**

```java
public class PhoneCode {
    private static Jedis jedis;

    public static void main(String[] args) throws InterruptedException {
        //连接redis
        jedis = Connect.connect();
        Scanner sc = new Scanner(System.in);
        System.out.println("输入手机号码:");
        String phone = sc.nextLine();
        System.out.println("正在发送验证码......");
        //获取随机验证码
        StringBuffer code = getCode();
        //号码校验
        verifyCode(phone, String.valueOf(code));
        Thread.sleep(3000);
        System.out.println("发送验证码:" + code);
        System.out.println("请输入验证码");
        String trueCode = null;
        for (int i = 0; i < 3; i++) {
            trueCode = sc.nextLine();
            if (getRedisCode(phone, trueCode)) {
                System.out.println("校验成功");
                break;
            } else {
                System.out.println("校验失败，重新输入验证码");
                continue;
            }
        }
        jedis.close();

    }

    /**
     * 验证码校验
     */
    public static boolean getRedisCode(String phone, String code) {
        //手机发送次数key
        String countKey = "VerifyCode" + phone + ":count";
        //验证码key
        String codeKey = "VerifyCode" + phone + ":code";
        String redisCode = jedis.get(codeKey);
        //判断
        if (redisCode.equals(code)) {
            return true;
        }
        return false;
    }

    /**
     * 每个手机号码每天只能发送一次，验证码放到redis中，设置过期时间
     * @param phone
     * @param code
     */
    public static void verifyCode(String phone, String code) {
        //手机发送次数key
        String countKey = "VerifyCode" + phone + ":count";
        //验证码key
        String codeKey = "VerifyCode" + phone + ":code";

        //每个手机号码只能发送三次
        String count = jedis.get(countKey);
        if (count == null) {
            //没有发送次数，第一次发送
            jedis.setex(countKey, 24 * 60 * 60, "1");
        } else if (Integer.parseInt(count) < 3) {
            //发送次数+1
            jedis.incr(countKey);
        } else {
            //发送三次  不能再发送
            System.out.println("今天发送次数已经超过三次");
            System.exit(0);
        }
        //发送验证码放到redis里面
        jedis.setex(codeKey, 120, code);
    }

    /**
     * 生成6位数字验证码
     * @return
     */
    public static StringBuffer getCode() {
        Random random = new Random();
        StringBuffer code = new StringBuffer();
        for (int i = 0; i < 6; i++) {
            int rand = random.nextInt(10);
            code.append(rand);
        }
        return code;
    }
}
```

## 七、SpringBoot整合Redis

**依赖**

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.6.4</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
            <version>2.6.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
            <version>2.11.1</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.22</version>
            <scope>compile</scope>
        </dependency>
</dependencies>
```

Redis配置类

```java
@EnableCaching
@Configuration
public class RedisConfig extends CachingConfigurerSupport {
    @Bean
    public RedisTemplate redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate template = new RedisTemplate();
        // 配置连接工厂
        template.setConnectionFactory(factory);
        //使用Jackson2JsonRedisSerializer来序列化和反序列化redis的value值（默认使用JDK的序列化方式）
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        //自定义ObjectMapper
        ObjectMapper objectMapper = new ObjectMapper();
        // 指定要序列化的域，field,get和set,以及修饰符范围，ANY是都有包括private和public
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        // 指定序列化输入的类型，类必须是非final修饰的，final修饰的类，比如String,Integer等会抛出异常
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
        //String序列化配置
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        // key采用String的序列化方式
        template.setKeySerializer(stringRedisSerializer);
        // hash的key也采用String的序列化方式
        template.setHashKeySerializer(stringRedisSerializer);
        // value序列化方式采用jackson
        template.setValueSerializer(jackson2JsonRedisSerializer);
        // hash的value序列化方式采用jackson
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }
}
```

**测试**

```java
@RestController
@RequestMapping("/redisTest")
public class RedisController {
    @Autowired
    private RedisTemplate redisTemplate;

    @GetMapping("/get")
    public String testRedis() {
        redisTemplate.opsForValue().set("k1", "v1");
        String k1 = (String) redisTemplate.opsForValue().get("k1");
        return k1;
    }
}
```

**redisTemplate封装类**		https://github.com/Anougme/-Utils/blob/master/Redis_Util/RedisUtils.java

## 八、Redis事务操作、锁机制、秒杀案例

Redis 事务是一个单独的**隔离操作**：事务中的所有命令都会**序列化**、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。 

Redis 事务的主要作用**就是串联多个命令防止别的命令插队**。

### Multi、Exec、discard

Multi命令：输入的命令都会依次进入命令队列中，不会执行，直到输入`exec`后才会依次执行命令。组队过程可通过`discard`放弃组队。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220523234928318.png" alt="image-20220523234928318" style="zoom:67%;" />

**错误处理**

- 组队中某个命令出现报告错误，执行时整个的所有队列都会被取消。
- 执行中某个命令出现错误，则只有错的命令不会执行，其余命令正常执行，事务并不会回滚。

### 锁机制

**悲观锁**

定义：每次去拿数据时都认为别人会修改，所以每次拿数据时都会给数据上锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，**读锁**，**写锁**等，都是在做操作之前先上锁。

**乐观锁**

定义：每次拿数据时都认为别人不会修改，因此不会上锁，只是在更新的时候会判断在此期间数据有没有更新，可用使用版本号等机制。**乐观锁适用于多读的应用类型，这样可以提高吞吐量。Redis 就是利用这种 check-and-set 机制实现事务的。**

**WATCH** **key** **[key ...]** 

在执行 multi 之前，先执行 watch key1 [key2],可以监视一个(或多个) key ，如果在事务**执行之前这个****(****或这些****) key** **被其他命令所改动，那么事务将被打断。**

**unwatch**

取消 WATCH 命令对所有 key 的监视。如果在执行 WATCH 命令之后，EXEC 命令或 DISCARD 命令先被执行了的话，那么就不需要再执行 UNWATCH 了。

### 事务三特性

- 单独的隔离操作

  事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会 

  被其他客户端发送来的命令请求所打断。 

- 没有隔离级别的概念

  队列中的命令没有提交之前都不会实际被执行，因为事务提交前任何指令都 

  不会被实际执行 

- 不保证原子性

  事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚

## 九、持久化

### 持久化之RDB

在指定的时间间隔内将内存中的数据集快照写入磁盘， 也就是行话讲的 `Snapshot`快照，它恢复时是将快照文件直接读到内存里。

#### 备份是如何执行的

Redis 会单独创建（fork）一个子进程来进行持久化，会先将数据写入到 一个**临时文件**中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。 整个过程中，主进程是不进行任何 IO 操作的，这就确保了极高的性能 如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那 RDB 方式要比 AOF 方式更加的高效。RDB的缺点**是最后一次持久化后的数据可能丢失**。

#### Fork

- Fork 的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等）数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程。
- 在 `Linux`程序中，fork()会产生一个和父进程完全相同的子进程，但子进程在此后多会 exec 系统调用，出于效率考虑，Linux 中引入了“**写时复制技术**” 。
- **一般情况父进程和子进程会共用同一段物理内存**，只有进程空间的各段的内容要发生变化时，才会将父进程的内容复制一份给子进程。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220524000140862.png" alt="image-20220524000140862" style="zoom:67%;" />

#### 优势

- 适合大规模的数据恢复 
- 对数据完整性和一致性要求不高更适合使用 
- 节省磁盘空间 
- 恢复速度快

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220524000316624.png" alt="image-20220524000316624" style="zoom:67%;" />

#### 劣势

- Fork 的时候，内存中的数据被克隆了一份，大致 2 倍的膨胀性需要考虑 
- 虽然 Redis 在 fork 时使用了**写时拷贝技术**,但是如果数据庞大时还是比较消耗性能。 
- 在备份周期在一定间隔时间做一次备份，所以如果 Redis 意外 down 掉的话，就会丢失最后一次快照后的所有修改。 
