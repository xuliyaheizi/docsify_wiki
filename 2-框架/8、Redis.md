# Redis

## 一、NoSql

NoSQL(NoSQL = **Not Only SQL** )，意即“不仅仅是 SQL”，泛指**非关系型的数据库**。NoSQL 不依赖业务逻辑方式存储，而以简单的 `key-value` 模式存储。因此大大的增加了数据库的扩展能力。

- 不遵循 SQL 标准。 
- 不支持 ACID。 
- 远超于 SQL 的性能。

**适用场景**

- 对数据高并发的读写 
- 海量数据的读写 
- 对数据高可扩展性的

## 二、Redis

**Redis** 是一个开源的 `key-value 存储系统`。和 `Memcached `类似，它支持存储的 value 类型相对更多，包括 `string(字符串)`、`list(链表)`、`set(集合)`、`zset(sorted set --有序集合)`和 `hash（哈希类型）`。这些数据类型都支持 `push/pop、add/remove` 及取交集并集和差集及更丰富的操作，而且这些操作都是**原子性**的。在此基础上，Redis 支持各种不同方式的排序。与 memcached 一样，为了保证效率，数据都是**缓存在内存中**。区别的是 Redis 会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件。并且在此基础上实现了`master-slave(主从)同步`。

**配合关系型数据库做高速缓存**

- 高频次，热门访问的数据，降低数据库 IO 
- 分布式架构，做 session 共享

**多样的数据结构存储持久化数据**

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220521193155461.png" alt="image-20220521193155461"/>

**redis介绍**

- 默认 16 个数据库，类似数组下标从 0 开始，初始默认使用 0 号库 
- 使用命令` select <dbid> `来切换数据库。如: select 8 
- 统一密码管理，所有库同样密码。 
- dbsize 查看当前数据库的 key 的数量 
- flushdb 清空当前库 
- flushall 通杀全部库

Redis 是**单线程+多路 IO 复用技术** 

`多路复用`是指使用**一个线程来检查多个文件描述符（Socket）的就绪状态**，比如调用select 和 poll 函数，传入多个文件描述符，如果有一个文件描述符就绪，则返回，否则阻塞直到超时。得到就绪状态后进行真正的操作可以在同一个线程里执行，也可以启动线程执行（比如使用线程池） 串行 vs 多线程+锁（memcached） vs 单线程+多路 IO 复用(Redis)。

（与 Memcache 三点不同: `支持多数据类型`，`支持持久化`，`单线程+多路 IO 复用`）。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220521194217299.png" alt="image-20220521194217299"/>

### 2.1、Redis的单线程和多路IO复用机制

Redis是`单线程`，主要是指**Redis的网络IO和键值对读写**是由一个线程来完成的，这也是Redis对外提供键值存储服务的主要流程。Redis的其他功能，如持久化、异步删除、集群数据同步等，是由额外的线程执行的。

Redis在启动的时候，是会**启动后台线程（BIO）的**。

- `Redis在 2.6 版本`，会启动2个后台线程，分别`处理关闭文件`、`AOF刷盘`这两个任务。
- `Redis在 4.0 版主之后`，新增了一个后台线程，用来`异步释放Redis内存`，也就是`lazyfree`线程。例如执行 `unlink key / flushdb async / flushall async`等命令，会把这些删除操作交给`后台线程`来执行。好处是不会**导致Redis主线程卡顿**。因此，要删除一个大key时，不要使用`del`命令删除，因为`del是在主线程处理`的，会导致Redis主线程卡顿。应该使用` unlink `命令来异步删除。

后台线程相当于一个消费者，生产者把耗时任务丢到任务队列中，消费者（BIO）不停轮询这个队列，拿出任务就去执行对应的方法即可。

![image-20220725134733751](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207251347179.png)

`关闭文件`、`AOF 刷盘`、`释放内存`这三个任务都有各自的任务队列：

- **BIO_CLOSE_FILE，关闭文件任务队列：**当队列有任务后，后台线程会调用` close(fd) `，将文件关闭；
- **BIO_AOF_FSYNC，AOF刷盘任务队列：**当 AOF 日志配置成` everysec `选项后，主线程会把 AOF 写日志操作封装成一个任务，也放到队列中。当发现队列有任务后，后台线程会调用` fsync(fd)`，将 AOF 文件刷盘。
- **BIO_LAZY_FREE，lazy free 任务队列：**当队列有任务后，后台线程会` free(obj) 释放对象 / free(dict) 删除数据库所有对象 / free(skiplist) 释放跳表对象`。

#### Redis为什么使用单线程？

为了避免不必要的多线程频繁切换上下文，多线程的设计会增加系统的复杂度。且Redis的性能瓶颈不在于CPU，而在于机器的内存大小。

多线程的开销问题。使用多线程，可以增加系统吞吐率，或是可以增加系统扩展性；对于一个多线程的系统来说，在有合理的资源分配的情况下，可以增加系统中处理请求操作的资源实体，进而提升系统能够同时处理的请求数。采用多线程后，系统的吞吐率可能会稳步增加，但是没有良好的系统设计的话，在后期，随着线程数的增加，系统吞吐率的增长就迟缓了。该情况的原因在于，系统中通常会存在被多线程同时访问的共享资源，比如一个共享的数据结构。当有多个线程要修改这个共享资源时，为了保证共享资源的正确性，就需要有额外的机制进行保证，而这个额外的机制，就会带来额外的开销。

所有在Redis的开发中，采用多线程开发一般会引入同步原语来保护共享资源的并发访问，如此会降低系统代码的易调试性和可维护性。

#### 单线程Redis效率高的原因？

Redis的大部分操作在内存上完成，且采用了高效的数据结构（哈希表和跳表）；Redis采用了IO多路复用机制，使其在网络IO操作中能并发处理大量的客户端请求，实现吞吐率。

**IO多路复用**

以 Get 请求为例，SimpleKV 为了处理一个 Get 请求，需要监听客户端请求（bind/listen），和客户端建立连接（accept），从 socket 中读取请求（recv），解析客户端发送请求（parse），根据请求类型读取键值数据（get），最后给客户端返回结果，即向 socket 中写回数据（send）。

下图显示了这一过程，其中，bind/listen、accept、recv、parse 和 send 属于网络 IO 处理，而 get 属于键值数据操作。既然 Redis 是单线程，那么，最基本的一种实现是在一个线程中依次执行上面说的这些操作。

![image-20220724091958903](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207240920145.png)

但是，在这里的网络 IO 操作中，有潜在的阻塞点，分别是 accept() 和 recv()。当 Redis 监听到一个客户端有连接请求，但一直未能成功建立起连接时，会阻塞在 accept() 函数这里，导致其他客户端无法和 Redis 建立连接。类似的，当 Redis 通过 recv() 从一个客户端读取数据时，如果数据一直没有到达，Redis 也会一直阻塞在 recv()。

这就导致 Redis 整个线程阻塞，无法处理其他客户端请求，效率很低。不过，幸运的是，socket 网络模型本身支持非阻塞模式。

### 2.2、Redis的线程模型

![redis单线程模型.drawio](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207251351835.webp)

蓝色部分是一个事件循环，由主线程负责，可以看到`网络I/O和命令处理`都是单线程。Redis初始化时：

1. 调用` epoll_create() `创建一个 epoll 对象和调用 socket() 一个服务端 socket。
2. 调用` bind() `绑定端口和调用` listen() `监听该 socket。
3. 调用` epoll_ctl() `将` listem socket `加入到 epoll，同时**注册连接事件**处理函数。

在初始化完成后，主线程就会进入到一个`事件循环函数`中

1. 首先，会调用处理发送队列函数，看发送队列里是否有任务，如果有发送任务，则通过` write `函数将客户端发送缓存区里的数据发送出去。如果这一轮数据没有发送完，就会注册写事件处理函数，等待` epoll_wait `发现可写后再处理。
2. 接着，调用` epoll_wait `函数等待事件的到来：
   1. 如果是**连接事件**到来，则会调用连接事件处理函数，该函数的作用：调用 accept 获取已连接的 socket -> 调用 epoll_ctl将已连接的 socket 加入到 epoll -> 注册读事件处理函数。
   2. 如果是**读事件**到来，则会调用读事件处理函数，该函数的作用：调用 read 获取客户端发送的数据 -> 解析命令 -> 处理命令 -> 将客户端对象添加到发送队列中 -> 将执行结果写到发送缓存区等待发送。
   3. 如果是**写事件**到来，则会调用写事件处理函数，该函数的作用：通过 write 函数将客户端发送缓存区里的数据发送出去，如果这一轮数据没有发送完，就会继续注册写事件处理函数，等待 epoll_wait 发现可写后再去处理。

## 三、常用五大数据类型

### 键（key）

```tex
keys *				查看当前库所有 key (匹配：keys *1) 
exists key			判断某个 key 是否存在 
type key		  	查看你的 key 是什么类型 
del key				删除指定的 key 数据 
unlink key			根据 value		选择非阻塞删除		仅将 keys 从 keyspace 元数据中删除，真正的删除会在后续异步操作。 
expire key 10		10 秒钟：为给定的 key 设置过期时间 
ttl key				查看还有多少秒过期，-1 表示永不过期，-2 表示已过期 
select			 	命令切换数据库 
dbsizex				查看当前数据库的 key 的数量 
flushdb				清空当前库 
flushall		  	通杀全部库 
```

### 3.1、字符串（String）

- String 是 Redis 最基本的类型，你可以理解成与 Memcached 一模一样的类型，`一个 key对应一个 value`。 
- String 类型是**二进制安全**的。意味着 Redis 的 string 可以**包含任何数据**。比如 jpg 图片或者序列化的对象。 
- String 类型是 Redis 最基本的数据类型，一个 Redis 中字符串 value 最多可以是 **512M**。

```tex
set <key><value>							添加键值对
get <key>									查询对应键值 
append <key><value>				  			将给定的<value> 追加到原值的末尾 
strlen <key>								获得值的长度 
setnx <key><value>							只有在 key 不存在时 设置 key 的值
incr <key> 			        				将 key 中储存的数字值增 1 只能对数字值操作，如果为空，新增值为 1 
decr <key> 				 					将 key 中储存的数字值减 1 只能对数字值操作，如果为空，新增值为-1 
incrby / decrby <key><步长>				   将 key 中储存的数字值增减。自定义步长。
mset <key1><value1><key2><value2> .....		同时设置一个或多个 key-value 对 
mget <key1><key2><key3> .....				同时获取一个或多个 value 
msetnx <key1><value1><key2><value2> .....	同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。
			原子性，有一个失败则都失败
getrange <key><起始位置><结束位置>			  获得值的范围，类似 java 中的 substring，前包，后包 
setrange <key><起始位置><value>				 用 <value> 覆写<key>所储存的字符串值，从<起始位置>开始(索引从 0 开始)。 
setex <key><过期时间><value>				 设置键值的同时，设置过期时间，单位秒。 
getset <key><value>							以新换旧，设置了新值同时获得旧值。
```

**原子性** 

所谓**原子**操作是指不会被线程调度机制打断的操作；这种操作一旦开始，就一直运行到结束，中间不会有任何 context switch （切换到另一个线程）。 

- 在单线程中， 能够在单条指令中完成的操作都可以认为是"原子操作"，因为中断只能发生于指令之间。 
- 在多线程中，不能被其它进程（线程）打断的操作就叫原子操作。Redis 单命令的原子性主要得益于 Redis 的单线程。 

**案例：** 

java 中的 i++是否是原子操作？**不是** 

i=0;两个线程分别对 i 进行++100 次,值是多少？ **2~200**

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220522112446066.png" alt="image-20220522112446066"/>

#### 数据结构

String 的数据结构为`简单动态字符串`(Simple Dynamic String,缩写 SDS)。是可以修改的字符串，内部结构实现上类似于 Java 的 `ArrayList`，采用**预分配冗余空间**的方式来减少内存的频繁分配.

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220522112539492.png" alt="image-20220522112539492"/>

如图中所示，内部为当前字符串实际分配的空间 `capacity `一般要**高于实际字符串长度len**。当字符串长度小于 1M 时，扩容都是加倍现有的空间，如果超过 1M，扩容时一次只会多扩 1M 的空间。需要注意的是字符串最大长度为 `512M`。

### 3.2、链表（List）

**单键多值Redis** 

`链表`是简单的`字符串列表`，按照`插入顺序排序`。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。它的底层实际是个`双向链表`，对两端的操作性能很高，通过索引下标的操作中间的节点性能会较差。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220522112807853.png" alt="image-20220522112807853"/>

```tex
lpush/rpush <key><value1><value2><value3> ....	从左边/右边插入一个或多个值。 
lpop/rpop <key>									从左边/右边吐出一个值。值在键在，值光键亡。 
rpoplpush <key1><key2>							从<key1>列表右边吐出一个值，插到<key2>列表左边。 
lrange <key><start><stop>						按照索引下标获得元素(从左到右) 
lrange mylist 0 -1								0 左边第一个，-1 右边第一个，（0-1 表示获取所有） 
lindex <key><index>								按照索引下标获得元素(从左到右) 
llen <key>										获得列表长度 
linsert <key> before <value><newvalue>			在<value>的后面插入<newvalue>插入值 
lrem <key><n><value>			        		从左边删除 n 个 value(从左到右) 
lset<key><index><value>							将列表 key 下标为 index 的值替换成 value
```

#### 数据结构

List 的数据结构为`快速链表 quickList`。首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构是 `ziplist`，也即是**压缩列表**。它将所有的元素紧挨着一起存储，分配的是一块连续的内存。当`数据量比较多`的时候才会改成` quicklist`。因为普通的链表需要的`附加指针空间太大`，`会比较浪费空间`。比如这个列表里存的只是 int 类型的数据，结构上还需要两个额外的指针` prev `和` next`。 

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220522113324227.png" alt="image-20220522113324227"/>

Redis 将`链表`和` ziplist `结合起来组成了` quicklist`。也就是将多个 ziplist 使用`双向指针`串起来使用。这样既满足了`快速的插入删除性能`，又不会出现`太大的空间冗余`。

### 3.3、集合（Set）

`Redis set `对外提供的功能与 list 类似是一个列表的功能，特殊之处在于` set `是可以**自动去重**的，当你需要存储一个`列表数据`，又不希望出现`重复数据`时，set 是一个很好的选择，并且 set 提供了`判断某个成员是否在一个 set 集合内的重要接口`，这个也是 list 所不能提供的。Redis 的` set `是` string `类型的`无序集合`。它底层其实是一个` value 为 null 的 hash 表`，所以`添加，删除，查找`的**复杂度都是** **O(1)**。一个算法，随着数据的增加，执行时间的长短，如果是 O(1)，数据增加，查找数据的时间不变。

```tex
sadd <key><value1><value2> ....			将一个或多个 member 元素加入到集合 key 中，已经存在的 member 元素将被忽略 
smembers <key>				           	取出该集合的所有值。 
sismember <key><value>			       	判断集合<key>是否为含有该<value>值，有 1，没有 0 
scard<key>			                   	返回该集合的元素个数。
srem <key><value1><value2> .... 	    删除集合中的某个元素。 
spop <key>				               	随机从该集合中吐出一个值。 
srandmember <key><n>		            随机从该集合中取出 n 个值。不会从集合中删除 。 
smove <source><destination>value		把集合中一个值从一个集合移动到另一个集合 
sinter <key1><key2>			      		返回两个集合的交集元素。 
sunion <key1><key2>			    		返回两个集合的并集元素。 
sdiff <key1><key2>						返回两个集合的差集元素(key1 中的，不包含 key2 中的)
```

#### 数据结构

Set 数据结构是 `dict 字典`，字典是用`哈希表`实现的。Java 中 HashSet 的内部实现使用的是 `HashMap`，只不过所有的` value 都指向同一个对象`。Redis 的 set 结构也是一样，它的内部也使用 hash 结构，所有的 value 都指向同一个内部值。

### 3.4、哈希（Hash）

- `Redis hash `是一个`键值对集合`。 
- `Redis hash `是一个` string `类型的` field `和` value `的映射表，hash 特别适合用于存储对象。 
- 类似 Java 里面的` Map<String,Object> `。
- 用户 ID 为查找的 key，存储的 value 用户对象包含姓名，年龄，生日等信息，如果用普通的 key/value 结构来存储 

主要有以下 2 种存储方式：

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220522113735993.png" alt="image-20220522113735993" width="45%" />

```tex
hset <key><field><value>			  	          	给<key>集合中的 <field>键赋值<value> 
hget <key1><field>					  		     	从<key1>集合<field>取出 value 
hmset <key1><field1><value1><field2><value2>...    	批量设置 hash 的值 
hexists<key1><field>				       		 	查看哈希表 key 中，给定域 field 是否存在。 
hkeys <key>						           	     	列出该 hash 集合的所有 field 
hvals <key>							             	列出该 hash 集合的所有 value 
hincrby <key><field><increment>				      	为哈希表 key 中的域 field 的值加上增量 1 -1 
hsetnx <key><field><value>						 	将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在 .
```

#### 数据结构

Hash 类型对应的数据结构是两种：`ziplist（压缩列表）`，`hashtable（哈希表）`。当`field-value `长度较短且个数较少时，使用` ziplist`，否则使用` hashtable`。

### 3.5、有序集合Zset（sorted set）

Redis **有序集合 zset 与普通集合 set** 非常相似，是一个`没有重复元素的字符串集合`。不同之处是`有序集合的每个成员`都关联了一个**评分（score）**，这个评分（score）被用来按照从`最低分到最高分`的方式排序集合中的成员。集合的成员是唯一的，但是评分可用是重复的 。因为元素是`有序`的, 所以你也可以很快的根据评分（score）或者次序（position）来获取一个范围的元素。访问有序集合的中间元素也是非常快的,因此你能够使用有序集合作为一个没有重复成员的智能列表。

```tex
zadd <key><score1><value1><score2><value2>… 		将一个或多个 member 元素及其 score 值加入到有序集 key 当中。 
zrange <key><start><stop> [WITHSCORES] 	        	返回有序集 key 中，下标在<start><stop>之间的元素 带 WITHSCORES，可以让分数一起和值返回到结果集。 
zrangebyscore key minmax [withscores] [limit offset count] 				
返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。 有序集成员按 score 值递增(从小到大)次序排列。 

zrevrangebyscore key maxmin [withscores] [limit offset count] 	同上，改为从大到小排列。
zincrby <key><increment><value> 		       					为元素的 score 加上增量 
zrem <key><value>			                   					删除该集合下，指定值的元素 
zcount <key><min><max>					      					统计该集合，分数区间内的元素个数 
zrank <key><value>							 					返回该值在集合中的排名，从 0 开始。
```

#### 数据结构

`SortedSet(zset) `是 Redis 提供的一个非常特别的数据结构，一方面它`等价于 Java的数据结构 Map<String, Double>`，可以给每一个元素 value 赋予一个**权重 score**，另一方面它又类似于` TreeSet`，内部的元素会按照`权重 score 进行排序`，可以得到每个元素的名次，还可以通过` score 的范围来获取元素的列表`。 

zset 底层使用了两个数据结构 

- `hash`，hash的作用就是**关联元素 value 和权重 score**，保障元素 value 的`唯一性`，可以通过元素 value 找到相应的 score 值。 
- `跳跃表`，跳跃表的目的`在于给元素 value 排序`，根据 score 的范围获取元素列表。

#### 跳跃表（跳表）

有序集合在生活中比较常见，例如根据成绩对学生排名，根据得分对玩家排名等。对于有序集合的底层实现，可以用数组、平衡树、链表等。数组不便元素的插入、删除；平衡树或红黑树虽然效率高但结构复杂；链表查询需要遍历所有效率低。Redis采用的是跳跃表。跳跃表效率堪比红黑树，实现远比红黑树简单。

2、实例

对比有序链表和跳跃表，从链表中查询出 51 

（1） 有序链表 

  要查找值为 51 的元素，需要从第一个元素开始依次查找、比较才能找到。共需要 6 次比较。 

（2） 跳跃表 

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220522114511831.png" alt="image-20220522114511831" width="50%" />

- 从第 2 层开始，1 节点比 51 节点小，向后比较。21 节点比 51 节点小，继续向后比较，后面就是 NULL 了，所以从 21 节点向下到第 1 层 
- 在第 1 层，41 节点比 51 节点小，继续向后，61 节点比 51 节点大，所以从 41 向下 
- 在第 0 层，51 节点为要查找的节点，节点被找到，共查找 4 次。 
- 从此可以看出跳跃表比有序链表效率要高 

## 四、Redis的发布与订阅

Redis 发布订阅 (pub/sub) 是一种**消息通信模式**：`发送者 (pub) 发送消息`，`订阅者(sub) 接收消息`。Redis 客户端可以订阅任意数量的频道。 

客户端可以订阅频道如下图 

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220522121140826.png" alt="image-20220522121140826" width="40%" />

当给这个频道发布消息后，消息就会发送给订阅的客户端

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220522121201473.png" alt="image-20220522121201473" width="40%" />

```
SUBSCRIBE channel1		 	客户端订阅 channel1
publish channel1 hello	 	另一个客户端，给 channel1 发布消息 hello
发布的消息没有持久化，如果在订阅的客户端收不到 hello，只能收到订阅后发布 的消息
```

## 五、Redis新数据类型

### 5.1、Bitmaps

现代计算机用二进制（位） 作为信息的基础单位， 1 个字节等于 8 位， 例如“abc” 字符串是由 3 个字节组成， 但实际在计算机存储时将其用二进制表示， “abc”分别对应的 ASCII 码分别是 97、 98、 99， 对应的二进制分别是 01100001、 01100010和 01100011，如下图 

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220522184731379.png" alt="image-20220522184731379" width="50%" />

合理地使用操作位能够有效地提高内存使用率和开发效率。Redis 提供了 Bitmaps 这个“数据类型”可以实现对位的操作： 

- Bitmaps 本身不是一种数据类型， 实际上它就是字符串（key-value） ，但是它可以对字符串的位进行操作。 
- Bitmaps 单独提供了一套命令， 所以在 Redis 中使用 Bitmaps 和使用字符串的方法不太相同。 可以把 Bitmaps 想象成一个以位为单位的数组，数组的每个单元只能存储 0 和 1， 数组的下标在 Bitmaps 中叫做偏移量。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220522184824449.png" alt="image-20220522184824449" width="50%" />

```
setbit<key><offset><value>				设置 Bitmaps 中某个偏移量的值（0 或 1）
注： 
很多应用的用户 id 以一个指定数字（例如 10000） 开头， 直接将用户 id 和 Bitmaps 的偏移量对应势必会造成一定的浪费， 通常的做法是每次做 setbit 操作时将 用户 id 减去这个指定数字。 在第一次初始化 Bitmaps 时， 假如偏移量非常大， 那么整个初始化过程执行会 比较慢， 可能会造成 Redis 的阻塞。

getbit<key><offset>						获取 Bitmaps 中某个偏移量的值

bitcount<key>[start end] 				统计字符串从 start 字节到 end 字节比特值为 1 的数量
统计字符串被设置为 1 的 bit 数。一般情况下，给定的整个字符串都会被进行计数，通 过指定额外的 start 或 end 参数，可以让计数只在特定的位上进行。start 和 end 参数 的设置，都可以使用负数值：比如 -1 表示最后一个位，而 -2 表示倒数第二个位， start、end 是指 bit 组的字节的下标数，二者皆包含。

bitop and(or/not/xor) <destkey> [key…]
bitop 是一个复合操作， 它可以做多个 Bitmaps 的 and（交集） 、 or（并集） 、 not （非） 、 xor（异或） 操作并将结果保存在 destkey 中。
```

### 5.2、HyperLogLog

  在工作当中，我们经常会遇到与统计相关的功能需求，比如统计网站 PV（PageView 页面访问量）,可以使用 Redis 的 incr、incrby 轻松实现。 但像 UV（UniqueVisitor，独立访客）、独立 IP 数、搜索记录数等需要去重和计数的问题如何解决？这种求集合中不重复元素个数的问题称为基数问题。 解决基数问题有很多种方案： 

- 数据存储在 MySQL 表中，使用 distinct count 计算不重复个数 
- 使用 Redis 提供的 hash、set、bitmaps 等数据结构来处理 

  以上的方案结果精确，但随着数据不断增加，导致占用空间越来越大，对于非常大的数据集是不切实际的。能否能够降低一定的精度来平衡存储空间？Redis 推出了 HyperLogLog Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定的、并且是很小的。 

  在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。 

**什么是基数?** 

  比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 

  基数(不重复元素)为 5。 基数估计就是在误差可接受的范围内，快速计算基数。

```
pfadd <key>< element> [element ...] 			    	添加指定元素到 HyperLogLog 中
执行命令后 HLL 估计的 近似基数发生变化，则返回 1，否则返回 0。
pfcount<key> [key ...] 						       		计算 HLL 的近似基数，可以计算多个 HLL，比如用 HLL 存储每 天的 UV，计算一周的 UV 可以使用 7 天的 UV 合并计算即可
pfmerge<destkey><sourcekey> [sourcekey ...] 			将一个或多个 HLL 合并后的结果存 储在另一个 HLL 中，比如每月活跃用户可以使用每天的活跃用户来合并计算可得
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

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220523234928318.png" alt="image-20220523234928318"/>

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

  事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。 

- 没有隔离级别的概念

  队列中的命令没有提交之前都不会实际被执行，因为事务提交前任何指令都不会被实际执行。

- 不保证原子性

  事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚。

## 九、持久化

### 9.1、持久化之RDB

在`指定的时间间隔内`将`内存中的数据集快照写入磁盘`， 也就是行话讲的 `Snapshot`快照，它恢复时是将快照文件直接读到内存里。

> 备份是如何执行的

Redis 会`单独创建（fork）一个子进程来进行持久化`，会先将数据写入到 一个**临时文件**中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。 整个过程中，主进程是`不进行任何 IO 操作的`，这就`确保了极高的性能 `，如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那 RDB 方式要比 AOF 方式更加的高效。RDB的缺点**是最后一次持久化后的数据可能丢失**。

> Fork

- Fork 的作用是`复制一个与当前进程一样的进程`。新进程的`所有数据（变量、环境变量、程序计数器等）`数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程。
- 在 `Linux`程序中，fork() 会产生一个和父进程完全相同的子进程，但子进程在此后多会 exec 系统调用，出于效率考虑，Linux 中引入了“**写时复制技术**” 。
- **一般情况父进程和子进程会共用同一段物理内存**，只有进程空间的各段的内容要发生变化时，才会将父进程的内容复制一份给子进程。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220524000140862.png" alt="image-20220524000140862" width="45%" />

> 优势

- 适合大规模的数据恢复 
- 对数据完整性和一致性要求不高更适合使用 
- 节省磁盘空间 
- 恢复速度快

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220524000316624.png" alt="image-20220524000316624" width="45%" />

> 劣势

- Fork 的时候，内存中的数据被克隆了一份，大致 2 倍的膨胀性需要考虑 
- 虽然 Redis 在 fork 时使用了**写时拷贝技术**，但是如果数据庞大时还是比较消耗性能。 
- 在备份周期在一定间隔时间做一次备份，所以如果 Redis 意外 down 掉的话，就会丢失最后一次快照后的所有修改。 

### 9.2、AOF日志持久化

是指`所有的命令行记录`以Redis命令请求协议的格式`完全持久化存储`，保存为AOF文件。Redis在执行完一条写操作命令后，就会把该命令以追加的命令写入到一个文件里，然后Redis重启时，会读取该文件记录的命令，再以`逐一执行命令的方式`来进行数据恢复。

![image-20220725162508622](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207251639720.png)

**优点：**

- 数据安全，`AOF持久化`可以配置`appendfsync`属性，有`always`，每进行一次命令操作就记录到AOF文件中一次。
- 通过append模式写文件，即使途中服务器宕机，可以通过`redis-check-aof`工具解决`数据一致性问题`。
- AOF机制的`rewrite`模式。AOF文件没被rewrite之前（文件过大时会对命令进行合并重写），可以删除其中的某些命令（比如误操作的flushall）

**缺点：**

- AOF文件比RDB文件大，且恢复速度慢。
- 数据集大的时候，比RDB启动效率低。

> 为什么先执行命令，再把数据写入日志？

- **避免额外的检查开销：**先执行命令，可以检查命令语法是否有错误，有错误将命令执行失败也不会写入日志中。避免了在使用日志恢复数据的时候，命令执行失败。
- **不会阻塞当前写操作命令的执行：**因为写操作命令执行成功后，才会将命令记录到日志中。

**风险也有**

- **数据可能会丢失：**执行写操作命令和记录日志是两个过程，在Redis还没将命令写入到硬盘时，服务器发生宕机了，就会有数据丢失的风险。
- **可能阻塞其他操作：**由于写操作命令执行成功后才记录到AOF日志，所以不会阻塞当前命令的执行。但是因为AOF日志也是在主线程中执行，所以将日志文件写入磁盘的 时候，可能会阻塞后续的操作无法执行。

> AOF的写回策略

![4eeef4dd1bedd2ffe0b84d4eaa0dbdea](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207251639896.png)

1. Redis 执行完`写操作命令`后，会将命令追加到` server.aof_buf 缓冲区`；
2. 然后通过` write() 系统`调用，将` aof_buf 缓冲区`的数据写入到 AOF 文件，此时数据并没有写入到硬盘，而是拷贝到了`内核缓冲区 page cache`，等待内核将数据写入硬盘；
3. 具体内核缓冲区的数据什么时候写入到硬盘，由内核决定。

Redis 提供了 3 种写回硬盘的策略，控制的就是上面说的第三步的过程。 在 Redis.conf 配置文件中的 appendfsync 配置项可以有以下 3 种参数可填：

- **Always**，这个单词的意思是「总是」，所以它的意思是每次写操作命令执行完后，同步将 AOF 日志数据写回硬盘；
- **Everysec**，这个单词的意思是「每秒」，所以它的意思是每次写操作命令执行完后，先将命令写入到 AOF 文件的内核缓冲区，然后每隔一秒将缓冲区里的内容写回到硬盘；
- **No**，意味着不由 Redis 控制写回硬盘的时机，转交给操作系统控制写回的时机，也就是每次写操作命令执行完后，先将命令写入到 AOF 文件的内核缓冲区，再由操作系统决定何时将缓冲区内容写回硬盘。

![image-20220725163405740](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207251639746.png)

> AOF日志过大，会触发什么机制？

AOF 日志是一个文件，随着执行的写操作命令越来越多，文件的大小会越来越大。 如果当 AOF 日志文件过大就会带来性能问题，比如重启 Redis 后，需要读 AOF 文件的内容以恢复数据，如果文件过大，整个恢复的过程就会很慢。所以，Redis 为了避免 AOF 文件越写越大，提供了 **AOF 重写机制**，当 AOF 文件的`大小超过所设定的阈值后`，Redis 就会启用 AOF 重写机制，来`压缩 AOF 文件`。

**AOF 重写机制**是在重写时，读取当前数据库中的所有`键值对`，然后将`每一个键值对用一条命令`记录到「新的 AOF 文件」，等到全部记录完后，就将新的 AOF 文件替换掉现有的 AOF 文件。

举个例子，在没有使用重写机制前，假设前后执行了「*set name xiaolin*」和「*set name xiaolincoding*」这两个命令的话，就会将这两个命令记录到 AOF 文件。

![image-20220725163502942](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207251639852.png)

但是**在使用重写机制后，就会读取 name 最新的 value（键值对） ，然后用一条 「set name xiaolincoding」命令记录到新的 AOF 文件**，之前的第一个命令就没有必要记录了，因为它属于「历史」命令，没有作用了。这样一来，一个键值对在重写日志中只用一条命令就行了。

重写工作完成后，就会将新的 AOF 文件覆盖现有的 AOF 文件，这就相当于压缩了 AOF 文件，使得 AOF 文件体积变小了。

>  重写AOF日志的过程是怎样的？

Redis 的**重写 AOF 过程是由后台子进程` bgrewriteaof `来完成的**，这么做可以达到两个好处：

- 子进程进行 AOF 重写期间，主进程可以继续处理命令请求，从而`避免阻塞主进程`；
- **子进程带有主进程的数据副本**，这里使用子进程而不是线程，因为如果是使用线程，多线程之间会共享内存，那么在修改共享内存数据的时候，需要通过加锁来保证数据的安全，而这样就会降低性能。而使用子进程，创建子进程时，父子进程是共享内存数据的，不过这个共享的内存只能以只读的方式，而当父子进程任意一方修改了该共享内存，就会发生`「写时复制」`，于是父子进程就有了独立的数据副本，就不用加锁来保证数据安全。

触发重写机制后，主进程就会创建重写 AOF 的子进程，此时父子进程共享物理内存，重写子进程只会对这个内存进行`只读`，重写 AOF 子进程会读取数据库里的所有数据，并逐一把内存数据的键值对转换成一条命令，再将命令记录到重写日志（新的 AOF 文件）。

**但是重写过程中，主进程依然可以正常处理命令**，那问题来了，重写 AOF 日志过程中，如果主进程修改了已经存在 key-value，那么会发生写时复制，此时这个 key-value 数据在子进程的内存数据就跟主进程的内存数据不一致了，这时要怎么办呢？

> 为了解决这种数据不一致问题（由于写时复制，键值对数据在子进程的内存数据与主进程的内存数据不一致了），Redis 设置了一个 **AOF 重写缓冲区**，这个缓冲区在创建 bgrewriteaof 子进程之后开始使用。

在重写 AOF 期间，当 Redis 执行完一个写命令之后，它会**同时将这个写命令写入到 「AOF 缓冲区」和 「AOF 重写缓冲区」**。

![image-20220725164018683](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207251640869.png)

也就是说，在 bgrewriteaof 子进程执行 AOF 重写期间，主进程需要执行以下三个工作:

- 执行客户端发来的命令；
- 将执行后的写命令追加到 「AOF 缓冲区」；
- 将执行后的写命令追加到 「AOF 重写缓冲区」；

当子进程完成 AOF 重写工作（*扫描数据库中所有数据，逐一把内存数据的键值对转换成一条命令，再将命令记录到重写日志*）后，会向主进程发送一条信号，信号是进程间通讯的一种方式，且是异步的。

主进程收到该信号后，会调用一个信号处理函数，该函数主要做以下工作：

- 将 AOF 重写缓冲区中的所有内容追加到新的 AOF 的文件中，使得新旧两个 AOF 文件所保存的数据库状态一致；
- 新的 AOF 的文件进行改名，覆盖现有的 AOF 文件。

信号函数执行完后，主进程就可以继续像往常一样处理命令了。

### 9.3、混合持久化机制

RDB的优点是`数据恢复速度快`，但是执行快照记录的`频率不好把握`。频率太低，会损失较多数据（未及时执行快照，而服务器宕机）。频率太快，就会影响性能；AOF优点是丢失数据少，但是数据恢复不快。

为了集成两者的优点，Redis4.0提出了混合使用**AOF日志和内存快照**（混合持久化），既保证了Redis重启速度，又降低了数据丢失风险。

混合持久化工作在**AOF日志重写过程中**，在AOF重写日志时，fork() 出来的重写子进程会先将**与主线程共享的内存数据以RDB方式写入到AOF文件中**，然后主线程处理的操作命令会被记录在`重写缓冲区`里，重写缓冲区里的**增量命令会以AOF方式写入到AOF文件**，写入完成后通知主进程将新的含有RDB格式和AOF格式的AOF文件替换旧的AOF文件。使用了混合持久化的AOF文件，前半部分是`RDB格式的全量数据`，后半部分是`AOF格式的增量数据`。

![image-20220725234807813](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207260000446.png)

好处在于，重启Redis加载数据的时候，由于前半部分是RDB内容，加载的速度会很快。加载完RDB的内容后，才会加载后半部分的AOF内容（Redis后台子进程重写AOF期间，主线程处理的操作命令），可以使得数据更少的丢失。

**混合持久化的优点：**结合了RDB和AOF持久化的优点，开头为RDB的格式，使得Redis可以更快的启动，同时结合了AOF的优点，可以减少损失更多数据的风险。

**混合持久化的缺点：**AOF文件中添加了RDB格式的内容，使得AOF文件的可读性变得很差；兼容性差，混合持久化的方式只适用于Redis4.0版本之后。

## 十、主从复制

将一台Redis服务的数据，复制到其他Redis服务器上。前者称为`主节点（master）`，后者称为`从节点（slave）`。数据的复制是`单向`的，只能从主节点到从节点。

- `读写分离`，性能扩展（应用读取多个从节点的数据，只能往主节点中写数据）
- `容灾快速恢复`。防止数据丢失，redis可以实现`高可用`，同时实现`数据的冗余备份`。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220531231808630.png" alt="image-20220531231808630" width="50%;" />

**主从复制的作用：**

- `数据冗余`：实现数据的热备份，也是持久化的另一种方式。
- `针对单机故障问题`：一个节点故障，其他节点可以继续提供服务，不影响用户使用。实现`快速恢复故障`，这也是服务冗余。
- `读写分离`：master服务主要用来写，slave服务主要读数据。提高服务的负载能力。
- `负载均衡`：同时配合读写分离，由主节点提供写服务，从节点提供读服务，分担服务器的负载。大大提高Redis服务的并发量和负载。
- `高可用的基石`：主从复制是哨兵和集群模式能够实施的基础。

### docker配置redis主从复制

**配置文件**

```tex
requirepass 123456 //设置密码
appendonly yes
protected-mode no  //取消保护模式
```

```tex
//docker启动redis命令
docker run -d -p 4030:6739 -v /home/redis/reids-master/redis.conf:/etc/reids/redis.conf --name redis-master redis --redis-server /etc/redis/redis.conf

// -v 宿主机目录:容器目录    --redis-server 以配置文件目录启动
```

```
//从机配置文件
requirepass 123456
appendonly yes
slaveof zhulinz.top 4030  //主机地址
masterauth 123456		//主机密码
```

**复制原理**

- Slave 启动成功连接到 master 后会发送一个 sync 命令 。
- Master 接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令， 在后台进程执行完毕之后，master 将传送整个数据文件到 slave,以完成一次完全同步。
- 全量复制：而 slave 服务在接收到数据库文件数据后，将其存盘并加载到内存中。 
- 增量复制：Master 继续将新的所有收集到的修改命令依次传给 slave,完成同步。
- 但是只要是重新连接 master,`一次完全同步（全量复制)`将被自动执行。

<img src="https://img-blog.csdnimg.cn/20210823213605854.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDE0MzExNA==,size_16,color_FFFFFF,t_70" alt="img" width="67%;" />

### Redis的同步机制

redis的主从同步机制可以确保redis的master和slave之间的数据同步。同步方式包括全量复制和增量复制。

#### 全量拷贝

![image-20220723161923695](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207231619761.png)

1. `Slave`第一次启动时，连接`Master`，发送`psync`命令，格式为` psync {runId} {offset}`

   > `{runId}` 为master的`运行id`；`{offset}`为slave自己的`复制偏移量`。
   > slave第一次连接master时，slave并不知道master的runId，也不知道自己偏移量，这时候slave会传一个问号和-1，告诉master节点是`第一次同步`。
   >
   > 格式为`psync ? -1`

2. 当`master`接收到 `psync ? -1` 时，知道slave是要`全量复制`，就会将自己的` runId `和` offset `告知` slave `，回复命令` fullresync {runId} {offset} `。同时，master会执行bgsave命令来生成rdb文件，期间的所有写命令将被写入缓冲区。

   > slave接受到master的回复命令后，会保存master的runId和offset，slave此时处于同步状态。
   > slave处于同步状态，如果此时收到请求，当配置参数`slave-server-stale-data yes`时，会响应当前请求；`slave-server-stale-data no`，返回错误。

3. master `bgsave`执行完毕，向slave发送rdb文件。rdb文件发送完毕后，开始向slave发送缓冲区中的写命令。

4. slave收到rdb文件，丢弃所有旧数据，开始载入rdb文件。

5. rdb文件同步结束之后，slave执行从master缓冲区发送过来的所以写命令。

6. 此后 master 每执行一个写命令，就向slave发送相同的写命令。

#### 增量拷贝

1. 如果出现网络闪断或者命令丢失异常情况时，当主从连接恢复后，由于从节点之前保存了自身已复制的偏移量和主节点的运行ID。因此会把它们当做psync参数发送给主节点，要求进行部分复制操作。格式为` pxync {runId} {offset}`。
2. 主节点接到pxync命令后首先核对参数runId是否与自身一致。如果一致，说明之前复制的是当前主节点，之后根据参数offset在自身复制积压缓冲区查找，如果偏移量之后的数据存在缓冲区中，则对从节点发送 +continue响应，表示可以进行部分复制；否则进行全量复制。
3. 主节点根据偏移量把复制积压缓冲区里的数据发送给从节点，保证主从复制进入正常状态。

**同步故障处理**

1. **拷贝超时**

   对于数据量较大的主节点，比如生成的rdb文件超过6GB以上时要格外小心。传输文件这一步操作非常耗时，速度取决于主从节点之间网络带宽，通过细致分析 full resync和 master slave这两行日志的时间差，可以算出rdb文件从创建到传输完毕消耗的总时间。如果总时间超过了 repl-timeout 所配置的值（默认60秒），从节点将放弃接受rdb文件并清理已经下载的临时文件，导致全量复制失败。

   针对数据量较大的节点，建议调大 repl-timeout 参数防止出现全量同步数据超时。

2. **积压缓冲区拷贝溢出**

   slave节点从开始接收rdb文件到接收完成期间，主节点仍然响应读写命令，因此主节点会把这期间写入命令保存在复制积压缓冲区内，当从节点加载完rdb文件后，主节点再把缓冲区的数据发送给从节点，保证主从数据一致性。

   如果主节点创建和传输RDB的时间过长，对于高流量写入场景非常容易造成主节点复制客户端缓冲区溢出。默认配置为`client-output-buffer-limit slave 256MB 64MB 60`，如果60秒内缓冲区消耗持续大于64MB或者直接超过256MB时，主节点将直接关闭复制客户端连接，造成全量同步失败。

   因此，运维人员需要根据主节点数据量和写命令并发量调整client-output-buffer-limit slave配置，避免全量复制期间客户端缓冲区溢出。对于主节点，当发送完所有的数据后就认为全量复制完成，打印成功日志：synchronization with slave127.0.0.1：6380 succeeded

3. **slave全量同步的响应问题**

   slave节点接收完主节点传送来的全部数据后会清空自身旧数据，执行flash old data，然后加载rdb文件。对于较大的rdb文件，这一步操作依然比较耗时。

   对于线上做读写分离的场景，从节点也负责响应读命令，如果slave节点正出于全量复制阶段，那么slave节点在响应读命令可能拿到过期或错误的数据。对于这种场景，redis复制提供了slave-server-stale-data yes参数（默认开启），如果开启则slave节点依然响应所有命令。对于无法容忍不一致的应用场景可以设置no来关闭命令执行，此时从节点除了info和slaveof命令之外所有的命令只返回sync with master in progress信息。

**节点运行Id**

1. 每个Redis节点启动后，都会动态分配一个40位的十六进制字符串作为运行ID，即`{runId}`。

   > 运行ID的主要作用是用来唯一识别Redis节点，比如从节点保存主节点的运行ID识别自己正在复制的是哪个主节点。

2. 如果只使用ip+port的方式识别主节点，那么主节点重启变更了整体数据集（如替换rdb/aof文件），从节点再基于偏移量复制数据将是不安全的，因此当运行ID变化后从节点将做全量复制。

3. 可以运行info server命令查看当前节点的运行ID。

**偏移量拷贝**

1. 参与复制的主从节点都会维护自身复制偏移量，即{offset}。
2. 主节点（master）在处理完写入命令后，会把命令的字节长度做累加记录，统计信息在info relication中的 master_repl_offset 指标中。
3. 从节点（slave）在接收到主节点发送的命令后，也会累加记录自身的偏移量，统计信息在info relication命令的slave_repl_offset指标中
4. 从节点（slave）每秒钟上报自身的复制偏移量给主节点，因此主节点也会保存从节点的复制偏移量。

**积压缓冲区拷贝**

1. 存在于主节点（master），默认大小为1MB，可以通过参数rel_backlog_size来修改默认大小。
2. 复制积压缓冲区是保存在主节点上的一个固定长度的队列。当从节点（slave）连接主节点时被创建，这时主节点（master）响应写命令时，不但会把命令发送给从节点，还会写入复制积压缓冲区。
3. 由于缓冲区本质上是先进先出（FIFO）的定长队列，所以能实现保存最近已复制数据的功能，用于部分复制和复制命令丢失的数据补救。复制缓冲区相关统计信息可以通过主节点的info replication命令查看。

## 十一、应用场景

使用缓存大多的目的是为了提升响应效率和并发量，减轻数据库的压力。

缓存穿透、缓存雪崩和缓存击穿的发生，都是因为在某些特殊情况下，缓存失去了预期的功能所致。当缓存失效或没有抵挡住流量，流量直接涌入到数据库中，在高并发情况下，可能直接击垮数据库，导致整个系统崩溃。

### 11.1、缓存穿透

通常流程是：一个请求过来，先查询是否在`缓存`中，如果缓存中存在，则直接返回。如果缓存中不存在对应的数据，则检索数据库，如果数据库中存在对应的数据，则更新缓存并返回结果。如果数据库中也不存在对应的数据，则`返回空或错误`。

**缓存穿透**是指`缓存`和`数据库`中都没有的数据，而用户不断发起请求。由于缓存是`不命中时被动写`的。并且出于`容错考虑`，如果从`底层数据库`中查不到数据则不写入`缓存`，这将导致这个不存在的数据每次请求都要到底层数据库中去查询，失去了缓存的意义。当高并发或有人利用不存在的Key频繁攻击时，数据库的压力骤增，甚至崩溃，这就是`缓存穿透`问题。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220602150658494.png"/>

**发生的场景：**

- 原来数据是存在的，但由于某些原因（`误删除、主动清理`等）在缓存和数据库层面被删除了，但前端和前置的应用程序依旧保有这些数据。
- 恶意攻击行为，利用不存在的Key或者恶意尝试导致产生`大量不存在的业务`数据请求。

**解决方案：**

- `设置并发锁`，防止大量请求数据库
- `对空值缓存`：如果一个查询返回的数据为空（不管是数据是否不存在），把这个空结果（null）进行缓存，设置空结果的过期时间会很短，最长不超过五分钟。当数据库被`写入或更新`该key的新数据时，缓存必须同时被刷新，避免数据不一致。
- `设置白名单`：利用`bitmaps`类型定义一个可访问的名单，`名单id作为bitmaps的偏移量`，每次访问和bitmaps的id进行比较，如果不对则进行拦截，不允许访问。
- `采用布隆过滤器`：布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。
- `进行实时监控`：当发现 Redis 的命中率开始急速降低，需要排查访问对象和访问的数据，和运维人员配合，可以设置黑名单限制服务。

### 11.2、缓存击穿

缓存击穿是指`缓存中没有但数据库中有的数据`（一般是缓存时间到期，`单个key`），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220602150915573.png" alt="image-20220602150915573"/>

**解决方案：**

- `预先设置热门数据`：在redis高峰访问之前，把一些热门数据提前存入到redis里面，加大这些热门数据key的时长。
- `实时调整`：现场监控哪些数据热门，实时调整key的过期时长。
- `使用互斥锁（Mutex Key）`：在缓存失效的时候（判断拿出来的值为空），不是立即去load db，先使用缓存工具的某些带成功操作返回值的操作（比如Redis的SETNX）去set一个mutex key，当操作返回成功时，再进行load db操作，并回设缓存，最后删除mutex key。当操作返回失败，证明有线程在load db，当前线程睡眠一段时间再去重试整个get缓存的方法。
- `提前使用互斥锁（Mutex Key）`：在value内部设置一个比缓存（Redis）过期时间短的过期时间标识，当异步线程发现该值快过期了，马上延长内置的这个时间，并重新从数据库中加载数据，设置到缓存中去。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220601230545180.png" alt="image-20220601230545180"/>

### 11.3、缓存雪崩

在使用缓存时，通常会对`缓存设置过期时间`，一方面目的是`保持缓存与数据库数据的一致性`，另一方面是`减少冷缓存占用过多的内存空间`。

缓存雪崩是指` Redis 服务器`在某个时间大量失效（针对`多个key缓存`），突然造成数据库访问压力急剧增大，像雪崩一样，redis雪崩危害巨大，甚至有可能服务器宕机，给公司造成巨大的经济损失。

正常情况

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220602151310778.png" alt="image-20220602151310778" width="67%;" />

缓存失效瞬间

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220602151450025.png" alt="image-20220602151450025" width="67%;" />

**解决方案：**

- `构建多级缓存架构`：nginx 缓存 + redis 缓存 +其他缓存（ehcache 等）。
- `使用锁或队列`：用锁或队列保证不会有`大量的线程对数据库一次性进行读写`，从而避免失效时大量的并发请求落到底层存储系统上。不适用高并发情况。
- `设置过期标志更新缓存`：记录`缓存数据是否过期（设置提前量）`，若过期会触发通知另外的线程在后台去更新实际key的缓存。
- `将缓存失效时间分散开`：比如我们可以在原有的失效时间基础上增加一个随机值，比如 1-5 分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。
- `双Key策略：`主Key设置过期时间，备Key不设置过期时间，当主Key失效时，直接返回备Key值。在更新缓存的时候，同时更新主key和备key的数据。

![image-20220726133923350](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207261339883.png)

### 11.3、如何设计一个缓存策略，可以动态缓存热点数据？

由于数据存储受限，系统并不是将所有数据都需要存放到缓存中的，而**只是将其中一部分热点数据缓存起来，**所以我们要设计一个热点数据动态缓存的策略。

热点数据动态缓存的策略总体思路：**通过数据最新访问时间来做排名，并过滤掉不常访问的数据，只留下经常访问的数据。**

以电商平台场景中的例子，现在要求只缓存用户经常访问的 Top 1000 的商品。具体细节如下：

- 先通过缓存系统做一个排序队列（比如存放 1000 个商品），系统会根据商品的访问时间，更新队列信息，越是最近访问的商品排名越靠前；
- 同时系统会定期过滤掉队列中排名最后的 200 个商品，然后再从数据库中随机读取出 200 个商品加入队列中；
- 这样当请求每次到达的时候，会先从队列中获取商品 ID，如果命中，就根据 ID 再从另一个缓存数据结构中读取实际的商品信息，并返回。

在 Redis 中可以用 zadd 方法和 zrange 方法来完成排序队列和获取 200 个商品的操作。

## 十二、Redis集群

以Redis的主从复制、哨兵模式、切片模式来实现高可用Redis服务。

> 主从复制

主从复制是Redis高可用服务的最基础的保证，**实现方案**是将从前的一台的Redis服务器，同步数据到多台从Redis服务器上，即一主多从的模式，且主从服务器之间采用的`读写分离`的方式。

主服务器可以进行`读写操作`，当发生写操作时自动将写操作同步给从服务器，而从服务器一般是只读，并接受主服务器同步过来写操作命令，然后执行这条命令。（主从服务器之间的命令复制是`异步`进行的）。

![image-20220726001308111](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207260013261.png)

在主从服务器命令传播阶段，主服务器收到新的写命令后，会发送给从服务器。但是，主服务器并不会等到从服务器实际执行完命令后，发送结果给客户端。而是**主服务器自己在本地执行完命令后就会向客户端发送结果**。如果从服务器还没有执行主服务器同步过来的命令，此时主从服务器之间的数据就不一致了。（`无法保证强一致性，主从数据时时刻刻保持一致，数据不一致是无法避免的`）。

> 哨兵模式

在Redis的主从服务器出现故障宕机时，需要手动进行恢复。为了解决该问题，Redis增加了哨兵模式，哨兵模式的作用是监控主从服务器，并且提供了主从节点故障转移的功能。

![image-20220726002108541](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207260021144.png)

> 切片集群模式

当Redis缓存数据量大到一台服务器无法缓存时，就需要使用切片集群（Redis Cluster）方案，将数据分布在不同的服务器上，来降低系统对单主节点的依赖，从而提高Redis服务的读写性能。

Redis Cluster方案采用哈希槽（Hash Slot），来处理数据与节点之间的映射关系。在Redis Cluster方案中，`一个切片集群共有16384个哈希槽`。这些哈希槽类似于数据分区，每个键值对都会根据它的key，被映射到一个哈希槽中：

- 根据键值对的key，按照CRC16算法计算一个16bit的值。
- 再用16bit值对16384取模，得到0~16383范围内的模数，每个模数代表一个相应编号的哈希槽。

**这些哈希槽是如何被映射到具体的Redis节点上的？**两种方案：

- **平均分配：**在使用 cluster create 命令创建 Redis集群时，Redis会自动把所有哈希槽平均分布到集群节点上。（比如集群中有9个节点，则每个节点上的槽的个数为 16384/9）。
- **手动分配：**可以使用 cluster meet 命令手动建立节点间的连接，组成集群，再使用 cluster addslots 命令，指定每个节点上的哈希槽个数。

![image-20220726093120915](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207260931168.png)

上图中的切片集群一共有 3 个节点，假设有 4 个哈希槽（Slot 0～Slot 3）时，我们就可以通过命令手动分配哈希槽，比如节点 1 保存哈希槽 0 和 1，节点 2 保存哈希槽 2 和 3。

```c
redis-cli -h 192.168.1.10 –p 6379 cluster addslots 0,1
redis-cli -h 192.168.1.11 –p 6379 cluster addslots 2,3
```

然后在集群运行的过程中，key1 和 key2 计算完 CRC16 值后，对哈希槽总个数 5 进行取模，再根据各自的模数结果，就可以被映射到对应的节点 1 和节点 3 上了。

需要注意的是，在手动分配哈希槽时，需要把 16384 个槽都分配完，否则 Redis 集群无法正常工作。

## 十三、Redis的过期删除与内存淘汰

### 13.1、Redis使用的过期删除策略是什么？

Redis 是可以对 key 设置过期时间的，因此需要有相应的机制将已过期的键值对删除，而做这个工作的就是过期键值删除策略。每当我们对一个 key 设置了过期时间时，Redis 会把该 key 带上过期时间存储到一个**过期字典**（expires dict）中，也就是说「过期字典」`保存了数据库中所有 key 的过期时间`。

当我们查询一个 key 时，Redis 首先检查该 key 是否存在于过期字典中：

- 如果不在，则正常读取键值；
- 如果存在，则会获取该 key 的过期时间，然后与当前系统时间进行比对，如果比系统时间大，那就没有过期，否则判定该 key 已过期。

Redis 使用的过期删除策略是**「惰性删除+定期删除」这两种策略配和使用。**

> 什么是惰性删除策略？

惰性删除策略的做法是**不主动删除全部过期键，而是每次从数据库访问 key 时，都先检测 key 是否过期，如果过期则删除 key。**

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207260938553.webp" alt="惰性删除" style="width: 33%;" />

**优点：**

- 因为每次访问时，才会检查 key 是否过期，所以此策略只会使用很少的系统资源，因此，惰性删除策略对CPU时间最友好。

**缺点：**

- 如果一个 key 已经过期，而这个 key 又仍然保留在数据库中，那么只要这个过期 key 一直没有被访问，它所占用的内存就不会被释放，造成了一定空间的内存浪费。所以，惰性删除策略对内存不友好。

> 什么是定期删除策略？

**定期删除策略：**每个一段时间（默认100ms）**随机**从数据库中取出一定数量的 key 进行检查，并删除其中的过期 key。

**定期删除的流程：**

1. 从过期字典中随机抽取 20 个key；
2. 检查这 20 个key是否过期，并删除已过期的key。
3. 如果本轮检查的已过期key的数量，超过5个（20/4），也就是`已过期key的数量占比随机抽取key的数量大于25%`，则继续重复步骤1；如果已过期的key比例小于25%，则停止继续删除过期key，然后等待下一轮检查。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207260956677.webp" alt="定时删除流程" style="width:30%;" />



**优点：**

- 通过限制删除操作执行的时长和频率，来减少删除操作对CPU的影响，同时也能删除一部分过期的数据来减少过期键对空间的无效占用。

**缺点：**

- 难以确定删除操作执行的时长和频率。如果执行的太频繁，就会对CPU不太好；如果执行的太少，就和惰性删除差不多了，过期key占用的内存不会及时得到释放。

### 13.2、Redis持久化时，对过期键会如何处理？

RDB文件分为两个阶段，RDB文件`生成阶段`和`加载阶段`

- **RDB生成阶段：**从内存状态持久化成RDB（文件）的时候，会对 key 进行过期检查，过期的键不会被保存到新的RDB文件中，因此Redis中的过期键不会对生成新RDB文件产生任何影响。
- **RDB加载阶段：**RDB加载阶段时，要看服务器是主服务器还是从服务器：
  - **主服务器：**主服务器运行模式的情况下，载入RDB文件时，程序会对文件中保存的键进行检查，**过期键不会被载入到数据库中**。所以过期键不会对载入RDB文件的主服务器造成影响。
  - **从服务器：**从服务器运行模式下，载入RDB文件，**不论键是否过期都会被载入到数据库中**。但由于主从服务器在进行数据同步时，**从服务器的数据会被清空**。所以一般来说，过期键载入RDB文件的从服务器也不会造成影响。



AOF文件分为两个阶段，`AOF文件写入阶段`和`AOF重写阶段`

- **AOF文件写入阶段：**当Redis以AOF模式持久化时，如果数据库某个过期键还没被删除，那么`AOF文件会保留此过期键`，当此过期键被删除后，Redis会向`AOF文件追加一条DEL命令`来显式地删除该键值。
- **AOF重写阶段：**执行AOF重写时，会对Redis中的`键值对进行检查`，`已过期的键不会被保存到重写后的AOF文件中`。因此不会对AOF重写造成任何影响。

### 13.3、Redis主从模式中，对过期键会如何处理？

当 Redis 运行在主从模式下时，**从库不会进行过期扫描，从库对过期的处理是被动的**。也就是即使从库中的 key 过期了，如果有客户端访问从库时，依然可以得到 key 对应的值，像未过期的键值对一样返回。

从库的过期键处理依靠主服务器控制，**主库在 key 到期时，会在 AOF 文件里增加一条 del 指令，同步到所有的从库**，从库通过执行这条 del 指令来删除过期的 key。

## 十四、数据库与缓存保证一致性？

### 14.1、先更新数据库，还是先更新缓存？

> 先更新数据库，再更新缓存

可能由于并发问题导致缓存与数据库中的数据不一致的现象。

举例：两个请求A和B，同时更新同一条数据，则可能出现以下的顺序：

![image-20220726134419713](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207261344271.png)

A请求先将数据库更新为1，在还没来得及更新缓存时，B请求将数据库的数据更新为2，紧接着也将缓存中数据更新为2，然后A请求更新缓存中的数据为1。此时数据库中的数据为2，而缓存中的数据是1，就出现了缓存和数据库中的数据不一致的现象。（由于并发问题，数据库更新与缓存更新是两个操作，不具有原子性）

> 先更新缓存，再更新数据库

与以上出现的现象可能相同

![image-20220726134822176](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207261348247.png)

**所以，无论是先更新数据库，还是先更新缓存。这两个方案都存在并发问题。当两个请求并发更新同一条数据的时候，可能会出现缓存和数据库中的数据不一致的现象。**

### 14.2、先更新数据库，还是先删除缓存？

> 旁路缓存策略（Cache Aside）：不更新缓存，而是缓存中的数据。然后，到读取数据的时候，发现缓存中没了数据之后，再从数据库中读取数据，更新到缓存中。

![image-20220726135232605](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207261352833.png)

**写策略的步骤：**

- 更新数据库中的数据；
- 删除缓存中的数据。

**读策略的步骤：**

- 如果读取的数据命中了缓存，则直接返回数据；
- 如果读取的数据没有命中缓存，则从数据库中读取数据，然后将数据写入到缓存，并且返回给用户。

> 先删除缓存，再更新数据库

假设某个用户的年龄是 20，请求 A 要更新用户年龄为 21，所以它会删除缓存中的内容。这时，另一个请求 B 要读取这个用户的年龄，它查询缓存发现未命中后，会从数据库中读取到年龄为 20，并且写入到缓存中，然后请求 A 继续更改数据库，将用户的年龄更新为 21。

![image-20220726143421756](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207261434872.png)

最终，该用户年龄在缓存中是 20（旧值），在数据库中是 21（新值），缓存和数据库的数据不一致。**先删除缓存，再更新数据库，在「读 + 写」并发的时候，还是会出现缓存和数据库的数据不一致的问题**。

> 先更新数据库，再删除缓存

假如某个用户数据在缓存中不存在，请求 A 读取数据时从数据库中查询到年龄为 20，在未写入缓存中时另一个请求 B 更新数据。它更新数据库中的年龄为 21，并且清空缓存。这时请求 A 把从数据库中读到的年龄为 20 的数据写入到缓存中。

![image-20220726144044729](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207261440922.png)

最终，该用户年龄在缓存中是 20（旧值），在数据库中是 21（新值），缓存和数据库数据不一致。由此可见先更新数据库，再删除缓存也是会出现数据不一致性的问题。**但是在实际中，这个问题出现的概率并不高。**

**因为缓存的写入通常要远远快于数据库的写入**，在实际中很难出现请求B已经更新了数据库并且删除了缓存，请求A才更新完缓存的情况。而一旦请求A早于请求B删除缓存之前更新了缓存，那么接下来的请求就会因为缓存不命中而从数据库中重新读取数据，所以不会出现这种不一致的情况。

**先更新数据库，再删除缓存**的方案是可以保证数据一致性的。

> 关于先更新数据库，再删除缓存方案，如何保证两个操作都能执行成功？

![image-20220726145239543](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207261452488.png)

在更新完数据库之后，删除缓存失败。此时数据库中的数据是新值，缓存中的值是旧值，就会出现数据不一致的问题。

解决问题的方法：

- 重试机制
- 订阅MySQL binlog，再操作缓存。

> 重试机制

我们可以引入**消息队列**，将第二个操作（删除缓存）要操作的数据加入到消息队列，由消费者来操作数据。

- 如果应用**删除缓存失败**，可以从消息队列中重新读取数据，然后再次删除缓存，这个就是**重试机制**。当然，如果重试超过的一定次数，还是没有成功，我们就需要向业务层发送报错信息了。
- 如果**删除缓存成功**，就要把数据从消息队列中移除，避免重复操作，否则就继续重试。

![image-20220726150159823](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207261502545.png)

> 订阅MySQL binglog，再操作缓存

「**先更新数据库，再删缓存**」的策略的第一步是更新数据库，那么更新数据库成功，就会产生一条变更日志，记录在 binlog 里。

于是我们就可以通过订阅 binlog 日志，拿到具体要操作的数据，然后再执行缓存删除，阿里巴巴开源的 Canal 中间件就是基于这个实现的。

Canal 模拟 MySQL 主从复制的交互协议，把自己伪装成一个 MySQL 的从节点，向 MySQL 主节点发送 dump 请求，MySQL 收到请求后，就会开始推送 Binlog 给 Canal，Canal 解析 Binlog 字节流之后，转换为便于读取的结构化数据，供下游程序订阅使用。

![image-20220726150242699](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207261502302.png)

## 十五、Redis的哨兵机制

哨兵其实也是一个运行在特殊模式下的Redis进程，也是一个节点。哨兵节点主要负责三件事情：**监控**、**选主**、**通知**。

### 15.1、如何判断主节点真的故障了？

哨兵会每个1秒给所有主从节点发送PING命令，当主从节点收到PING命令后，会发送一个响应命令给哨兵。

![image-20220726152755159](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207261527094.png)

如果主节点或者从节点没有在规定的时间内响应哨兵的 PING 命令，哨兵就会将它们标记为「**主观下线**」。这个「规定的时间」是配置项 `down-after-milliseconds` 参数设定的，单位是毫秒。

> 客观下线；客观下线只适用于主节点

之所以针对「主节点」设计「主观下线」和「客观下线」两个状态，是因为有可能「主节点」其实并没有故障，可能只是因为主节点的系统压力比较大或者网络发送了拥塞，导致主节点没有在规定时间内响应哨兵的 PING 命令。

所以，为了减少误判的情况，哨兵在部署的时候不会只部署一个节点，而是用多个节点部署成**哨兵集群**（*最少需要三台机器来部署哨兵集群*），**通过多个哨兵节点一起判断，就可以就可以避免单个哨兵因为自身网络状况不好，而误判主节点下线的情况**。同时，多个哨兵的网络同时不稳定的概率较小，由它们一起做决策，误判率也能降低。

> 如何判定主节点为客观下线的呢？

当一个哨兵判断主节点为「主观下线」后，就会向其他哨兵发起命令，其他哨兵收到这个命令后，就会根据自身和主节点的网络状况，做出赞成投票或者拒绝投票的响应。

![image-20220726153311602](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207261533359.png)

当这个哨兵的赞同票数达到哨兵配置文件中的 quorum 配置项设定的值后，这时主节点就会被该哨兵标记为「客观下线」。

例如，现在有 3 个哨兵，quorum 配置的是 2，那么一个哨兵需要 2 张赞成票，就可以标记主节点为“客观下线”了。这 2 张赞成票包括哨兵自己的一张赞成票和另外两个哨兵的赞成票。

PS：quorum 的值一般设置为哨兵个数的二分之一加1，例如 3 个哨兵就设置 2。

哨兵判断完主节点客观下线后，哨兵就要开始在多个「从节点」中，选出一个从节点来做新主节点。
