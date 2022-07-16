# Map-HashMap源码阅读

HashMap实现了Map接口，允许放入的值的`key`或`value`为`null`，除该类未实现同步外，其余与Hashtable大致相同（Hashtable的方法都有`synchronized`修饰）

HashMap容器`不保证元素顺序`，根据需要可能会对元素`重新哈希`，元素的顺序也会被重新打散，因此不同时间迭代同一个HashMap的顺序可能会不同。并且根据对冲突的处理方式不同，哈希表有两种实现方式，一种开放地址方式(open addressing)，另一种是冲突链表方式(Separate chaining with linked lists)。

在将键值对存入数组之前，将key通过哈希算法计算出哈希值，把`哈希值`作为`数组下标`，把该下标对应的位置作为键值对的存储位置，通过该方法建立的数组叫做`哈希表`，存储位置叫做`桶(bucket)`。数组是通过整数下标直接访问元素，哈希表是通过字符串key直接访问元素，也就说哈希表是一种特殊的数组（关联数组），哈希表广泛应用于实现数据的快速查找（在map的key集合中，一旦存储的key的数量特别多，那么在要查找某个key的时候就会变得很麻烦，数组中的key需要挨个比较，哈希的出现，使得这样的比较次数大大减少。）

## 数据结构

```java
//HashMap的初始容量大小；容量必须是2的幂次方倍数
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16  1*2^4=16

//最大容量，如果一个更高的值由任何一个带参数的构造函数隐式指定时使用。必须是 2 <= 1<<30 的幂。
static final int MAXIMUM_CAPACITY = 1 << 30;

//负载因子大小
static final float DEFAULT_LOAD_FACTOR = 0.75f;

//使用树而不是列表的 bin 计数阈值。将元素添加到至少具有这么多节点的 bin 时，bin 将转换为树。
//该值必须大于 2 并且应该至少为 8，以便与树移除中关于在收缩时转换回普通 bin 的假设相吻合。
//从链表变为树的一个阈值
static final int TREEIFY_THRESHOLD = 8;

//在调整大小操作期间 untreeifying（拆分）bin 的 bin 计数阈值。
//应小于 TREEIFY_THRESHOLD，并且最多 6 以在移除时进行收缩检测。
//从树化变为链化的条件。当树化的元素小于6则进行链化
static final int UNTREEIFY_THRESHOLD = 6;

//可对其进行树化的 bin 的最小表容量。 （否则，如果 bin 中有太多节点，则调整表的大小。）
//应至少为 4 TREEIFY_THRESHOLD 以避免调整大小和树化阈值之间的冲突。
static final int MIN_TREEIFY_CAPACITY = 64;
```

## 构造方法

```java
//无参构造方法；构造一个具有默认初始容量 (16) 和默认加载因子 (0.75) 的空 HashMap。
//只设置了负载因子，并没有进行数组的初始化
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

//有参构造方法
public HashMap(int initialCapacity) {
    //构造自定义容量大小，默认负载因子的HashMap
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}    

public HashMap(int initialCapacity, float loadFactor) {
    //判断设置的容量大小是否合理
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    //设置的容量大小超过了最大值，就用默认的最大值
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    //负载因子是否规范
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}

//返回给定目标容量的二次方
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}

public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```

