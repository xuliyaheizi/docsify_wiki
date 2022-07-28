# 集合面试题

##  一、Java集合类

### 1.1、什么是 Java 集合框架？

集合代表了一组`对象`（和数组一样，但是数组长度不能变，集合可以），Java集合框架定义了一套规范，用来表示、操作集合，是具体操作与实现细节解耦。

- 必须是`高性能`的。基本集合（`动态数组、链表、树、哈希表`）的实现也必须是高效的。
- 允许不同类型的集合，以类似的方式工作，具有高度的`互操作性`。
- 对一个集合的`扩展与适应`必须是简单的。

Java集合有Collection、Map；Java集合框架的内容：

- **接口：**代表集合的抽象数据类型。例如`Collection、List、Set、Map`等。定义多个接口的意义是为了以`不同的方式操作集合对象`。
- **实现（类）：**是集合接口的具体实现。可重复使用的数据结构，例如：`ArrayList、LinkedList、HashSet、HashMap`。
- **算法：**实现集合接口的对象里的方法执行的一些有用的计算，例如：`搜索和排序`。这些算法称为`多态`，那是因为相同的方法可以在相似的接口上有着不同的实现。

### 1.2、为什么 Collection 不扩展 Cloneable 和 Serializable 接口？

集合接口指定一组称为元素的对象。如何维护元素取决于 `Collection 的具体实现`。例如，某些 Collection 实现（如 List）允许重复元素，而其他实现（如 Set）则不允许。许多 Collection 实现都有一个公共的克隆方法。但是，将它包含在 Collection 的所有实现中并没有真正的意义。这是因为 Collection 是一种抽象表示。重要的是执行。在处理实际实现时，克隆或序列化的语义和含义会发挥作用；所以具体的实现应该决定它应该如何被克隆或序列化，或者即使它可以被克隆或序列化。因此，在所有实现中强制克隆和序列化实际上不太灵活且限制性更强。具体的实现应该决定它是否可以被克隆或序列化。

`List：有序、可重复；Set：有序、不可重复；`

### 1.3、为什么 Map 接口不扩展 Collection 接口？

尽管 Map 接口及其实现是 `Collections Framework` 的一部分，但 Map 不是集合，集合也不是 Map。因此 Map 扩展 Collection 没有意义，反之亦然。如果 Map 扩展了 Collection 接口，那么元素在哪里？Map 包含键值对，它提供了检索键或值列表作为集合的方法，但它不适合“元素组”范式。

首先Map提供的是`键值对映射（即Key和value的映射）`，而collection提供的是`一组数据（并不是键值对映射）`。如果map继承了collection接口，那么所有实现了map接口的类到底是用map的键值对映射数据还是用collection的一组数据呢（就我们平常所用的hashMap、hashTable、treeMap等都是键值对，所以它继承collection完全没意义），而且map如果继承了collection接口的话还违反了面向对象的接口分离原则。

### 1.4、什么是迭代器？

`Iterator 接口`提供了迭代任何 Collection 的方法。我们可以使用迭代器方法从集合中获取迭代器实例。迭代器在 Java 集合框架中取代了枚举。迭代器允许调用者在迭代期间从底层集合中`移除`元素。

### 1.5、枚举和迭代器接口有什么区别？

枚举的速度是迭代器的两倍，并且使用的内存非常少。枚举是非常基础的，适合基本需求。但是 Iterator 比 Enumeration 更安全，因为它总是拒绝其他线程修改它正在迭代的集合对象。迭代器在 Java 集合框架中取代了枚举。迭代器允许调用者从`底层集合中删除元素`，而枚举是不可能的。迭代器方法名称已得到改进，使其功能清晰。

### 1.6、为什么没有像 Iterator.add() 这样的方法来向集合中添加元素？

语义尚不清楚，因为 Iterator 的合同不保证迭代的顺序。但是请注意，ListIterator 确实提供了添加操作，因为它确实保证了迭代的顺序。

### 1.7、为什么没有 Iterator 接口的具体实现？

Iterator 接口声明了用于迭代集合的方法，但它的实现由 Collection 实现类负责。每个返回迭代器进行遍历的集合类都有自己的迭代器实现嵌套类。这允许集合类选择迭代器是快速故障还是故障安全。例如 ArrayList 迭代器是故障快速的，而 CopyOnWriteArrayList 迭代器是故障安全的。

### 1.8、哪些集合类提供对其元素的随机访问？

`ArrayList`、`HashMap`、`TreeMap`、`Hashtable `类提供对其元素的随机访问。

### 1.9、什么是枚举集？

java.util.EnumSet是设置实现以与枚举类型一起使用。枚举集合中的所有元素都必须来自一个枚举类型，该枚举类型在创建集合时显式或隐式指定。EnumSet 未同步，并且不允许使用 null 元素。它还提供了一些有用的方法，如 copyOf(Collection c)、of(E first, E... rest) 和 ComplementOf(EnumSet s)。

### 1.10、哪些集合类是线程安全的？

`Vector`、`Hashtable`、`Properties `和 `Stack `是同步类，因此它们是线程安全的，可以在多线程环境中使用。Java 1.5 Concurrent API 包括一些允许在迭代时修改集合的集合类，因为它们在集合的克隆上工作，因此它们在多线程环境中使用是安全的。

### 1.11、什么是并发集合类？

Java 1.5 Concurrent 包 ( java.util.concurrent) 包含线程安全的集合类，允许在迭代时修改集合。按照设计，迭代器是快速失败的并抛ConcurrentModificationException。其中一些类是CopyOnWriteArrayList, ConcurrentHashMap, CopyOnWriteArraySet。阅读这些帖子以更详细地了解它们。 

避免ConcurrentModificationException、CopyOnWriteArrayList 示例HashMap 与 ConcurrentHashMap

### 1.12、什么是集合类？

java.util.Collections是一个实用程序类，仅由操作或返回集合的静态方法组成。它包含对集合进行操作的多态算法、“包装器”，它返回一个由指定集合支持的新集合，以及其他一些零碎的东西。这个类包含集合框架算法的方法，例如二进制搜索、排序、改组、反向等。

### 1.13、什么是 Comparable 和 Comparator 接口？

Java 提供了 Comparable 接口，如果我们想使用 Arrays 或 Collections 排序方法，任何自定义类都应该实现该接口。Comparable 接口具有用于排序方法的 compareTo(T obj) 方法。我们应该重写此方法，如果“this”对象小于、等于或大于作为参数传递的对象，则它返回负整数、零或正整数。但是，在大多数现实生活场景中，我们希望根据不同的参数进行排序。例如，作为 CEO，我想根据 Salary 对员工进行排序，HR 想根据年龄对员工进行排序。这是我们需要使用Comparator接口的情况，因为Comparable.compareTo(Object o)方法实现只能基于一个字段排序，我们不能选择要排序的字段 Object.Comparator 接口compare(Object o1, Object o2)方法需要实现，需要两个 Object 参数，它应该以返回的方式实现如果第一个参数小于第二个参数，则返回负整数；如果它们相等，则返回零；如果第一个参数大于第二个参数，则返回正整数。

### 1.14、Comparable 和 Comparator 接口有什么区别？

Comparable 和 Comparator 接口用于对`对象的集合或数组进行排序`。**Comparable 接口用于提供对象的自然排序**，我们可以使用它来提供基于单一逻辑的排序。
**Comparator 接口用于提供不同的排序算法**，我们可以选择我们想要使用的比较器来对给定的对象集合进行排序。

### 1.15、我们如何对对象列表进行排序？

如果我们需要对一个`对象数组进行排序`，我们可以使用`Arrays.sort()`。 如果我们需要对对象列表进行排序，我们可以使用`Collections.sort`。这两个类都重载了 sort() 方法，用于`自然排序（使用 Comparable）或基于标准排序（使用 Comparator）`。Collections 内部使用 Arrays 排序方法，因此两者具有相同的性能，只是 Collections 需要一些时间来将列表转换为数组。

### 1.16、在将 Collection 作为参数传递给函数时，我们如何确保函数无法修改它？

我们可以在将它作为参数传递之前使用方法创建一个只读集合，Collections.unmodifiableCollection(Collection c)，这将确保任何更改集合的操作都会抛出
UnsupportedOperationException。

### 1.17、我们如何从给定的集合创建一个同步的集合？

我们可以Collections.synchronizedCollection(Collection c)用来获取由指定集合支持的同步（线程安全）集合。

### 1.18、Collections Framework 中实现的常用算法有哪些？

Java Collections Framework 提供了排序和搜索等常用的算法实现。Collections 类包含这些方法实现。这些算法中的大多数都适用于 List，但其中一些适用于所有类型的集合。其中一些是排序、搜索、改组、最小值-最大值。

### 1.19、什么是大 O 符号？举几个例子？

Big-O 表示法根据数据结构中元素的数量来描述算法的性能。由于 Collection 类实际上是数据结构，我们通常倾向于使用 Big-O 表示法来根据时间、内存和性能来选择要使用的集合实现。示例 1：ArrayListget(index i)是一个常量时间操作，不依赖于数量列表中的元素。所以它在 Big-O 表示法中的性能是 O(1)。
示例 2：对数组或列表的线性搜索性能是 O(n)，因为我们需要搜索整个元素列表才能找到元素。

### 1.20、与 Java Collections Framework 相关的最佳实践是什么？

根据需要选择正确的集合类型，例如如果大小是固定的，我们可能希望使用 Array 而不是 ArrayList。如果我们必须按插入顺序遍历 Map，我们需要使用 TreeMap。如果我们不想重复，我们应该使用 Set。一些集合类允许指定初始容量，因此如果我们估计要存储的元素数量，我们可以使用它来避免重新散列或调整大小。根据接口而不是实现来编写程序，它允许我们在以后轻松地更改实现。始终使用泛型来确保类型安全，并在运行时避免 ClassCastException。使用 JDK 提供的不可变类作为 Map 中的键，以避免为我们的自定义类实现 hashCode() 和 equals()。尽可能将 Collections 实用程序类用于算法或获取只读、同步或空集合，而不是编写自己的实现。它将增强代码重用，具有更高的稳定性和低可维护性。

### 1.21、Collection和Collections的区别

1. Collection是集合的上级**接口**，继承它的有Set和List接口
2. Collections是集合的**工具类**，提供了一系列的静态方法对集合的搜索、查找、同步等操作

### 1.22、Enumeration和Iterator接口的区别

Iterator替代了Enumeration，Enumeration是一个旧的迭代器了。 

 与Enumeration相比，Iterator更加安全，**因为当一个集合正在被遍历的时候，它会阻止其它线程去修改集合**。 

-  我们在做练习的时候，迭代时会不会经常出错，抛出ConcurrentModificationException异常，说我们在遍历的时候还在修改元素。 
-  这其实就是fail-fast机制~ 

 **区别有三点：** 

-  Iterator的方法名比Enumeration更科学 
-  Iterator有fail-fast机制，比Enumeration更安全 
-  Iterator能够删除元素，Enumeration并不能删除元素

### 1.23、说说什么是fail-fast机制？

fail-fast机制，即`快速失败机制`，是java集合框架中的一种**错误检测机制**。`多线程`下用`迭代器遍历一个集合对象时`，如果遍历过程中对集合对象的内容进行了`修改（增加、删除）`，则会抛出`Concurrent Modification Exception`。fail-fast机制并不保证在不同步的修改下一定会抛出异常，这种机制一般仅用于检测bug。

**原理：**迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个 `modCount `变量。集合在被遍历期间如果内容发生变化，就会改变modCount的值。每当迭代器使用hashNext()/next()遍历下一个元素之前，都会检测`modCount变量是否为expectedmodCount值`，是的话就返回遍历；否则抛出异常，终止遍历。

例如：当某一个线程A通过iterator去遍历某集合的过程中，若集合的内容被其他的线程所改变了，那么线程A访问集合时，就会抛出ConcurrentModifificationException异常，产生fail-fast事件。这里的操作主要是指add、remove和clear，对集合元素个数进行修改。

解决办法：使用java.util.concurrent包下的类去取代java.util包下的类。

可以这么理解：在遍历之前，把 modCount 记下来 expectModCount，后面 expectModCount 去和 modCount 进行比较，如果不相等了，证明已并发了，被修改了，于是抛出ConcurrentModifificationException 异常。

## 二、Map（重中之重）

### 2.1、HashMap 在 Java 中是如何工作的？

Map.EntryHashMap 在**静态嵌套类实现中存储键值对**。HashMap 做`散列算法`，在`put and get`方法中使用`hashCode()`和`equals()`方法。当我们通过传递键值对调用put方法时，HashMap使用Key hashCode()和散列来找出索引来存储键值对，条目存储在` LinkedList（数组每个位置存储链表）`中，因此如果已经存在条目，则使用 equals() 方法检查传递的键是否已存在，如果`存在则覆盖值`，否则创建新条目并存储此键值条目.当我们通过传递Key调用get方法时，它再次使用hashCode()来查找数组中的索引，然后使用equals()方法找到正确的Entry并返回它的值。下图将清楚地解释这些细节。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207131853000.png" alt="image-20220713185302133" style="width:50%;" />

关于 HashMap 需要了解的其他重要事项是`容量`、`负载因子`、`阈值调整大小`。HashMap `初始默认容量为 16`，`负载因子为 0.75`。`阈值是容量乘以负载因子`，每当我们尝试添加条目时，如果映射大小大于阈值，HashMap 会将映射的内容重新散列到具有更大容量的新数组中。容量始终是 2 的幂，因此如果您知道需要存储大量键值对，例如在缓存数据库中的数据时，最好使用正确的容量和负载因子初始化 HashMap。

HashMap的初始容量设置计算：（需要存储的元素个数/负载因子-1）

### 2.2、可以使用任何类作为 Map 键吗？

可以`使用任何类`作为 Map `Key`，但是如果类`重写了equals()方法`，也应该重写`hashCode()方法`。

- 如果类重写了 equals 方法，它也应该重写 hashCode 方法。
- 类的所有实例需要遵循与 equals 和 hashCode 相关的规则。
- 如果一个类没有使用 equals，你不应该在 hashCode 中使用它。
- 咱们自定义 key 类的最佳实践是使之为不可变的，这样，hashCode 值可以被缓存起来，拥有更好的性能。不可变的类也可以确保 hashCode 和 equals 在未来不会改变，这样就会解决与可变相关的问题了。

### 2.3、Map 接口提供了哪些不同的 Collection 视图？

Map 接口提供了三个集合视图： 

**Set keySet()**：返回此映射中包含的键的 Set 视图。集合由地图支持，因此对地图的更改会反映在集合中，反之亦然。如果在对集合进行迭代时修改了映射（通过迭代器自己的删除操作除外），则迭代的结果是不确定的。该集合支持元素移除，即通过 Iterator.remove、Set.remove、removeAll、retainAll 和 clear 操作从映射中移除相应的映射。它不支持 add 或 addAll 操作。 

**集合值()**：返回此映射中包含的值的集合视图。集合由地图支持，因此对地图的更改会反映在集合中，反之亦然。如果在对集合进行迭代时修改了映射（通过迭代器自己的删除操作除外），则迭代的结果是不确定的。该集合支持元素移除，即通过 Iterator.remove、Collection.remove、removeAll、retainAll 和 clear 操作从映射中移除相应的映射。它不支持 add 或 addAll 操作。 

**Set<Map.Entry<K, V>> entrySet()**：返回此映射中包含的映射的 Set 视图。集合由地图支持，因此对地图的更改会反映在集合中，反之亦然。如果在对集合进行迭代时修改了映射（通过迭代器自己的删除操作或通过迭代器返回的映射条目上的 setValue 操作除外），则迭代的结果是未定义的。该集合支持元素移除，即通过 Iterator.remove、Set.remove、removeAll、retainAll 和 clear 操作从映射中移除相应的映射。它不支持 add 或 addAll 操作。

### 2.4、HashMap 和 Hashtable有什么区别？

HashMap和HashTable都实现了Map接口，两者的区别：

- `HashMap`的键和值都允许有`null值`存在；而`Hashtable`不允许。
- 都实现了`Map`、`Cloneable`、`Serializbale`。
- HashMap继承的是`AbstractMap`，并且AbstractMap也实现了Map接口。Hashtable继承的是`Dictionary`。
- HashMap是`非线程安全`的；Hashtable是`线程安全`的（大部分 public 修饰的普通方法都是 `synchronized `字段修饰的）。由于线程安全的问题，HashMap的效率要比Hashtable高。
- HashMap提供了一组键来迭代，因此它是快速失败的，但Hashtable提供了不支持此功能的键的枚举

一般现在不建议用Hashtable:
①HashTable是遗留类，内部实现很多没优化和冗余。
②即使在多线程环境下，现在也有同步的`ConcurrentHashMap`替代，没有必要因为是多线程而用Hashtable

### 2.5、如何在 HashMap 和 TreeMap 之间做出决定？

对于在 Map 中插入、删除和定位元素，HashMap 提供了最佳选择。但是，如果您需要按排序顺序遍历键，那么 TreeMap 是您更好的选择。根据集合的大小，将元素添加到 HashMap，然后将映射转换为 TreeMap 以进行排序键遍历可能会更快。

### 2.6、HashMap 和 HashSet 区别

`HashSet` 底层就是基于 `HashMap` 实现的。（`HashSet` 的源码非常非常少，因为除了 `clone()`、`writeObject()`、`readObject()`是 `HashSet` 自己不得不实现之外，其他方法都是直接调用 `HashMap` 中的方法。

|               `HashMap`                |                          `HashSet`                           |
| :------------------------------------: | :----------------------------------------------------------: |
|           实现了 `Map` 接口            |                       实现 `Set` 接口                        |
|               存储键值对               |                          仅存储对象                          |
|     调用 `put()`向 map 中添加元素      |             调用 `add()`方法向 `Set` 中添加元素              |
| `HashMap` 使用键（Key）计算 `hashcode` | `HashSet` 使用成员对象来计算 `hashcode` 值，对于两个对象来说 `hashcode` 可能相同，所以`equals()`方法用来判断对象的相等性 |

### 2.7、HashMap 和 TreeMap 区别

`TreeMap` 和`HashMap` 都继承自`AbstractMap` ，但是需要注意的是`TreeMap`它还实现了`NavigableMap`接口和`SortedMap` 接口。

### 2.8、HashMap 的底层实现

JDK1.8 之前 HashMap 底层是`数组和链表`结合在一起使用也就是`链表散列`。HashMap通过key的`hashCode`经过`扰动函数`处理过后得到 hash 值，然后通过 (n - 1) & hash 判断当前元素存放的位置（这里的 n 指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突。

所谓扰动函数指的就是HashMap的`hash()`。使用 hash 方法也就是扰动函数是为了防止一些实现比较差的 hashCode() 方法换句话说使用扰动函数之后可以`减少碰撞`。

### 2.9、HashMap 的长度为什么是 2 的幂次方

为了能让 HashMap `存取高效`，`尽量较少碰撞`，也就是要`尽量把数据分配均匀`。Hash 值的范围值`-2147483648 到 2147483647`，前后加起来大概 40 亿的映射空间，只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个 40 亿长度的数组，内存是放不下的。所以这个散列值是不能直接拿来用的。用之前还要先做对数组的长度取模运算，得到的余数才能用来要存放的位置也就是对应的数组下标。这个数组下标的计算方法是“ `(n - 1) & hash`”。（n 代表数组长度）。这也就解释了 HashMap 的长度为什么是 2 的幂次方。

在hash值的运算中，取余（%）操作中如果除数是2的幂次，则等价于与其除数减1的（&）操作，相当于hash%length=hash&(length-1)的前提length是2的幂次。并且采用二进制位操作（&），相对于取余操作能够提高运算效率。

- 第一个原因是方便哈希取余

  将元素放在table数组上面，是用`hash值%数组大小定位位置`，而HashMap是用`hash值&(数组大小-1)`，却能和前面达到一样的效果，这就得益于HashMap的大小是2的倍数，2的倍数意味着该数的二进制位只有一位为1，而该数-1就可以得到二进制位上1变成0，后面的0变成1，再通过&运算，就可以得到和%一样的效果，并且位运算比%的效率高得多。

  HashMap的容量是2的n次幂时，(n-1)的2进制也就是1111111***111这样形式的，这样与添加元素的hash值进行位运算时，能够充分的散列，使得添加的元素均匀分布在HashMap的每个位置上，减少hash碰撞。

- 第二个方面是在扩容时，利用扩容后的大小也是2的倍数，将已经产生hash碰撞的元素完美的转移到新的table中去。

### 2.10、Java中HashMap的key值要是为类对象则该类需要满足什么条件？

**需要同时重写该类的hashCode()方法和它的equals()方法**。 

-  从源码可以得知，在插入元素的时候是**先算出该对象的hashCode**。如果hashcode相等话的。那么表明该对象是存储在同一个位置上的。 
-  如果调用equals()方法，**两个key相同**，则**替换元素** 
-  如果调用equals()方法，**两个key不相同**，则说明该**hashCode仅仅是碰巧相同**，此时是散列冲突，将新增的元素放在桶子上 

 一般来说，我们会认为：**只要两个对象的成员变量的值是相等的，那么我们就认为这两个对象是相等的**！因为，Object底层比较的是两个对象的地址，而对我们开发来说这样的意义并不大~这也就为什么我们要重写equals()方法 

 重写了equals()方法，就要重写hashCode()的方法。因为**equals()认定了这两个对象相同**，而**同一个对象调用hashCode()方法时**，是应该返回相同的值的！

### 2.11、HashMap桶的位置是怎么找到的？为什么用 & 不用 % ，为什么是(length - 1)，为什么要右移16位

- 通过hash算法，计算hash值，然后定位到桶位置
- 求余运算：`a%b相当于a-(a/b)*b`;&是按位与运算，是CPU的一条指令；从计算机层面上看&的运算效率更高。

### 2.12、HashMap添加删除查询的时间复杂度和空间复杂度?

- 哈希表的查找、添加、删除的时间复杂度都是O(1)

### 2.13、HashMap与ConcurrentHashMap的异同？

- 都是`Key-Value`形式的存储数据
- HashMap是`线程不安全`的，ConcurrentHashMap是`JUC下的线程安全`
- HashMap的底层数据结构是`数组、链表（JDK1.8之前）`，JDK1.8之后，底层数据结构是`数组、链表、红黑树`。当`链表的长度达到8`的时候，且`数组的元素个数大于等于64`的时候，链表的`查询速度不如红黑树`，因此链表会转化为红黑树。
- HashMap的`初始容量为16`，当添加后数组的元素个数达到原数组的`长度*负载因子（0.75）`的时候，会进行`扩容`，以`数组的2倍长度`进行扩容。
- `ConcurrentHashMap`在JDK1.8之前是用`分段锁`来实现的`Segment+HashEntry`，Segment数组大小默认为`16`，`2的n次方`。JDK1.8之后采用`node+CAS+Synchronized`来保证并发安全进行实现。

### 2.14、对红黑树了解多少？为什么不用二叉树/平衡树？

红黑树本质上是一种二叉查找树，为了保持平衡，在二叉查找树的基础上增加了一些规则：

1. 每个节点要么是红色，要么是黑色。
2. 根节点永远是黑色的。
3. 所有的叶子节点都是黑色（叶子节点就是图中的null节点）。
4. 每个红色的两个子节点一定是黑色的。
5. 从任一节点到其子树中每个叶子节点的路径都包含相同数量的黑色节点。

![image-20220727203125808](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207272031779.png)

> 为什么不用二叉树？

红黑树是一种`平衡的二叉树`，插入、删除、查找的**最坏时间复杂度都为O(logn) **，避免了**二叉树最坏情况下的O(n)时间复杂度**。

> 为什么不用平衡二叉树？

平衡二叉树是比红黑树`更严格的平衡树`，为了保持平衡，需要`旋转的次数更多`，也就说**平衡二叉树保持平衡的效率更低**，所以平衡二叉树的插入和删除的效率比红黑树要低。

### 2.15、HashMap的put流程？

![image-20220727203618342](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207272036445.png)

1. 首先进行哈希值的扰动，获取一个新的哈希值。`(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);`

2. 判断tab是否位空或者长度为0，如果是则进行扩容操作。

   ```
   if ((tab = table) == null || (n = tab.length) == 0)
       n = (tab = resize()).length;
   ```

3. 根据哈希值计算下标，如果对应小标正好没有存放数据，则直接插入即可否则需要覆盖。`tab[i = (n - 1) & hash])`

4. 判断tab[i]是否为树节点，否则向链表中插入数据，是则向树中插入节点。

5. 如果链表中插入节点的时候，链表长度大于等于8，则需要把链表转换为红黑树。`treeifyBin(tab, hash);`

6. 最后所有元素处理完成后，判断是否超过阈值；`threshold`，超过则扩容。

### 2.16、HashMap元素的查找？

![image-20220727203711313](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207272037563.png)

1. 使用扰动函数，获取新的哈希值
2. 计算数组下标，获取节点
3. 当前节点和key匹配，直接返回
4. 否则，当前节点是否为树节点，查找红黑树
5. 否则，遍历链表查找

### 2.17、HashMap的哈希/扰动函数是怎么设计的？

HashMap的哈希函数是先拿到 key 的hashCode，是一个32位的int类型的数值，然后让 hashCode的高16位和低16位进行`异或操作`。**作用：**降低哈希碰撞的概率。

```java
static final int hash(Object key) {
    int h;
    // key的hashCode和key的hashCode右移16位做异或运算
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

### 2.18、哈希函数的构造方法？

- **除留取余法：**H（key)=key%p（p<=N）,关键字除以一个不大于哈希表长度的正整数p，所得余数为地址，当然HashMap里进行了优化改造，效率更高，散列也更均衡。
- **直接定址法：**直接根据key来映射到对应的数组位置。
- **数字分析法：**取key的某些数字（例如十位和百位）作为映射的位置。
- **平方取中法：**取key平方的中间几位作为映射的位置。
- **折叠法：**将key分割成位数相同的几段，然后把它们的叠加和作为映射的位置。

![image-20220727204640415](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207272046170.png)

### 2.19、解决哈希冲突有哪些方法？

- **链地址法：**在冲突的位置拉一个链表，把冲突的元素放进去。

- **开放定址法：**开放定址法就是从冲突的位置再接着往下找，给冲突元素找个空位。

  找到空闲位置的方法也有很多种：

  - 线行探查法: 从冲突的位置开始，依次判断下一个位置是否空闲，直至找到空闲位置
  - 平方探查法: 从冲突的位置x开始，第一次增加`1^2`个位置，第二次增加`2^2`…，直至找到空闲的位置
  - ……

  ![image-20220727205434860](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207280926885.png)

- **再哈希法：**换种哈希函数，重新计算冲突元素的地址。

- **建立公共溢出区：**再建一个数组，把冲突的元素放进去

### 2.20、HashMap是线程安全的吗？多线程下有什么问题？

HashMap不是线程安全的，可能会发生以下问题：

- 多线程下扩容死循环。JDK1.7中的HashMap的链表插入使用头插法插入元素，在多线程的情况下，扩容的时候有可能导致环形链表的出现，形成死循环。因此JDk1.8使用尾插法插入元素，在扩容时保持链表元素原本的顺序，不会出现环形链表的问题。

  > 多线程情况下链表头插法为什么会出现环形链表的情况？

  并发情况下，如果插入元素的两个线程都调用了 rehash 方法，即扩容方法，会导致链表成环的问题

  ```java
  /**
       * Transfers all entries from current table to newTable.
       */
  void transfer(Entry[] newTable, boolean rehash) {
      int newCapacity = newTable.length;
      for (Entry<K,V> e : table) {
          while(null != e) {
              Entry<K,V> next = e.next;
              if (rehash) {
                  e.hash = null == e.key ? 0 : hash(e.key);
              }
              int i = indexFor(e.hash, newCapacity);
              e.next = newTable[i];
              newTable[i] = e;
              e = next;
          }
      }
  }
  ```

  **transfer的过程：**首先获取新表的长度，之后遍历新表的每一个entry，然后每个entry中的链表，以反转的形式，形成 rehash 之后的链表。

  **并发问题：**若当前线程此时获得entry节点，但是被线程中断无法继续执行，此时线程二进入transfer函数，并把函数顺利执行，此时新表中的某个位置有了节点，之后线程一获得执行权继续执行，因为并发transfer，所以两者都是扩容的同一个链表，当线程一执行到e.next = new table[i]的时候，由于线程二之前数据迁移的原因导致此时new table[i]上就有entry存在，所以线程一执行的时候，会将next节点，设置为自己，导致自己互相使用next引用对方，因此产生链表，导致死循环。

- 多线程的put可能导致元素的丢失。多线程同时执行put操作，如果计算出来的索引位置是相同的，那会造成前一个key被后一个key覆盖，从而导致元素的丢失。此问题在JDK1.7和JDK1.8中都存在。

- put和get并发时，可能导致get为null。线程1执行put时，因为元素个数超出了threshold而导致了 rehash，线程2此时执行get，有可能导致这个问题。此问题在JDK1.7和JDK1.8中都存在。

### 2.21、有什么办法能解决HashMap线程不安全的问题呢？

Java中有HashTable、Collections.synchronizedMap、以及ConcurrentHashMap可以实现线程安全的Map。

- HashTable是直接在操作方法上加synchronized关键字，锁住整个table数组，粒度比较大。
- Collections.synchronizedMap是使用Collections集合工具的内部类，通过传入Map 封装出一个 SynchronizedMap对象，内部定义一个对象锁，方法内通过对象锁实现。
- ConcurrentHashMap在JDK1.7中使用分段锁，在JDK1.8中使用CAS+Synchronized。

### 2.22、可以具体说一下ConcurrentHashMap的实现吗？

ConcurrentHashMap在JDK1.7中是**使用分段锁实现的**，在JDK1.8中**是基于CAS+Synchronized实现的。**

> JDK1.7分段锁

从结构上来说，分段锁机制里面包含了一个Segment数组，Segment继承于ReentrantLock，Segment则包含HashEntry的数组，HashEntry本身就是一个链表的结构，具有保存key、value的能力能指向下一个节点的指针。

实际上就是相当于每个Segment都是一个HashMap，默认的Segment长度也是16，也就是支持16个线程的并发写，Segment之间相互不会受到影响。

![image-20220727215929054](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207272159137.png)

**put流程**

整个流程和HashMap非常类似，只不过是先定位到具体的Segment，然后通过ReentrantLock去操作而已，后面的流程和HashMap基本是一样的。

1. 计算hash，定位到Segment，segment如果是空就先初始化。
2. 使用ReentrantLock加锁，如果获取锁失败则尝试自旋，自旋超过次数就阻塞获取，保证一定获取锁成功。
3. 遍历HashEntry，就是和HashMap一样，数组中key和hash一样就直接替换，不存在就再插入链表，链表同样操作。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207272206666.jpg" alt="微信图片_20220727220441" style="width:33%;" />

**get流程**

get也很简单，key通过hash定位到segment，再遍历链表定位到具体的元素上，需要注意的是value是volatile的，所以get是不需要加锁的。

> JDK1.8 CAS+Synchronized

jdk1.8实现线程安全不是在数据结构上下功夫，它的数据结构和HashMap是一样的，数组+链表+红黑树。它实现线程安全的关键点在于put流程。

**put流程**

1. 首先计算hash，遍历node数组，如果node是空的话，就通过CAS+自旋的方式初始化。

   ```java
   tab = initTable();
   ```

   node数组初始化

   ```java
   private final Node<K,V>[] initTable() {
       Node<K,V>[] tab; int sc;
       while ((tab = table) == null || tab.length == 0) {
           //如果正在初始化或者扩容
           if ((sc = sizeCtl) < 0)
               //等待
               Thread.yield(); // lost initialization race; just spin
           else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {   //CAS操作
               try {
                   if ((tab = table) == null || tab.length == 0) {
                       int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                       @SuppressWarnings("unchecked")
                       Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                       table = tab = nt;
                       sc = n - (n >>> 2);
                   }
               } finally {
                   sizeCtl = sc;
               }
               break;
           }
       }
       return tab;
   }
   ```

2. 如果当前数组位置是空则直接通过CAS自旋写入数据

   ```java
   static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                       Node<K,V> c, Node<K,V> v) {
       return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
   }
   ```

3. 如果hash==MOVED，说明需要扩容，执行扩容

   ```java
   else if ((fh = f.hash) == MOVED)
       tab = helpTransfer(tab, f);
   ```

   ```java
   final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
       Node<K,V>[] nextTab; int sc;
       if (tab != null && (f instanceof ForwardingNode) &&
           (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
           int rs = resizeStamp(tab.length);
           while (nextTab == nextTable && table == tab &&
                  (sc = sizeCtl) < 0) {
               if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                   sc == rs + MAX_RESIZERS || transferIndex <= 0)
                   break;
               if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                   transfer(tab, nextTab);
                   break;
               }
           }
           return nextTab;
       }
       return table;
   }
   ```

4. 如果都不满足，就使用Synchronized写入数据，写入数据同样判断链表、红黑树，链表写入和HashMap的方式一样，key hash一样被覆盖，反之就尾插法，链表长度超过8就转换红黑树。

   ![微信图片_20220727221428](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207272214659.jpg)

   **get查询**

   get很简单，和HashMap基本相同，通过key计算位置，table该位置key相同就返回，如果是红黑树按照红黑树获取，否则就遍历链表获取。

### 2.23、HashMap的内部节点是有序的吗？

HashMap是无序的，根据hash值随机插入。如果要想使用有序的Map，可以使用LinkedHashMap或者TreeMap。

### 2.24、LinkedHashMap怎么实现有序的？

LinkedHashMap维护了一个双向链表，有头尾节点，同时 LinkedHashMap 节点 Entry 内部除了继承 HashMap 的Node属性，还有before和after用于标识前置节点和后置节点。

![image-20220727222601652](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207280928178.png)

可以实现按插入的顺序或访问顺序排序。

![image-20220727222637892](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207272226342.png)

### 2.25、说说TreeMap是如何实现有序的？

TreeMap是按照key的自然顺序或者Comparator的顺序进行排序，内部是通过红黑树来实现。所以要么key所属的类实现Comparable接口，或者自定义一个实现了Comparator接口的比较器，传给TreeMap用于key的比较。

![image-20220727223334298](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207272233986.png)

## 三、List（有序、可重复）

### 3.1、ArrayList和Vector有什么异同？

`ArrayList `和 `Vector `在许多方面都是相似的类。 

- 两者都是`基于索引`的，并由`内部数组备份`。 
- 两者都`保持插入顺序`，我们可以按`插入顺序获取元素`。 
- ArrayList 和 Vector 的`迭代器`实现在设计上都是`快速失败`的。 
- ArrayList 和 Vector 都`允许空值和使用索引号`随机访问元素。 

这些是 ArrayList 和 Vector 之间的区别。 

- `Vector `是同步的，而 `ArrayList `是不同步的。但是，如果您在迭代时寻找列表的修改，您应该使用 `CopyOnWriteArrayList`。 
- ArrayList 比 Vector 快，因为它`不会因为同步而产生任何开销`。 
- ArrayList 更加通用，因为我们可以使用 Collections 实用程序类轻松地从中获取同步列表或只读列表。

### 3.2、Array 和 ArrayList 有什么区别？什么时候使用 Array 而不是 ArrayList？

- 数组可以包含`原始对象或对象`，而 ArrayList `只能包含对象`。（`数组支持基本数据类型和引用类型；ArrayList支持引用类型`）
- 数组是`固定大小`的，而 ArrayList 大小是`动态`的。（ArrayList可以根据`自身的情况对数组进行扩容`，达到动态数组的作用）
- 数组并没有像ArrayList那样提供很多特性，比如addAll、removeAll、iterator等。虽然ArrayList是我们处理list时的明显选择，但很少有时候数组很好用。 
- 如果列表的大小是固定的，并且主要用于`存储和遍历`它们。 
- 对于原始数据类型的列表，虽然集合使用自动装箱来减少编码工作，但在处理固定大小的原始数据类型时仍然会使它们变慢。 
- 如果您正在处理固定的多维情况，使用远比 List<List<>>

### 3.4、ArrayList 和 LinkedList 有什么区别？

**ArrayList**（查询效率高，增删效率低）

- 优点：ArrayList是实现了`基于动态数组`的数据结构，因为地址连续，一旦数据存储好了，`查询效率会比较高`（在内存里是连续的）
- 缺点：由于`地址连续，插入和删除操作后`，`ArrayList需要移动数据`，因此效率会比较低。

**LinkedList**（查询效率低，增删效率高）

- 优点：LinkedList是`基于双向链表`的数据结构，`地址是任意`的，在开辟内存空间的时候不需要等一个连续的地址。对于新增和删除操作，LinkedList比较占优势。LinkedList适用于`要头尾操作或插入指定位置`的场景。
- 缺点：由于LinkedList需要`移动指针`，查询操作性能比较低。

**使用场景分析**

- 当需要对数据进行对`随机访问`的时候，选用ArrayList
- 当需要对数据进行`多次增加删除修改`时，采用LinkedList



- ArrayList 和 LinkedList 都实现了 List 接口，但它们之间存在一些差异。 
- ArrayList 是由 Array 支持的基于索引的数据结构，因此它提供对其元素的随机访问，性能为 O(1)，但 LinkedList 将数据存储为节点列表，其中每个节点都链接到它的前一个节点和下一个节点。所以即使有一种使用索引获取元素的方法，但在内部它是从开始遍历到索引节点然后返回元素，所以性能是O(n)，比ArrayList慢。 
- 与 ArrayList 相比，LinkedList 中元素的插入、添加或删除更快，因为在中间添加元素时没有调整数组大小或更新索引的概念。 
- LinkedList 比 ArrayList 消耗更多内存，因为 LinkedList 中的每个节点都存储了前一个和下一个元素的引用。

### 3.5、ArrayList集合加入1万条数据，应该怎么提高效率

ArrayList的默认初始容量为10，要插入大量数据的时候需要不断扩容，而扩容是非常影响性能的。因此，现在明确了10万条数据了，我们可以**直接在初始化的时候就设置ArrayList的容量**！

### 3.6、说出ArrayList,LinkedList的存储性能和特性

 ArrayList的底层是数组，LinkedList的底层是双向链表。 

-  ArrayList它支持以角标位置进行索引出对应的元素(随机访问)，而LinkedList则需要遍历整个链表来获取对应的元素。因此一般来说ArrayList的访问速度是要比LinkedList要快的 
-  ArrayList由于是数组，对于删除和修改而言消耗是比较大(复制和移动数组实现)，LinkedList是双向链表删除和修改只需要修改对应的指针即可，消耗是很小的。因此一般来说LinkedList的增删速度是要比ArrayList要快的 

####  扩展：

 ArrayList的增删**未必**就是比LinkedList要慢。 

-  如果增删都是在**末尾**来操作【每次调用的都是remove()和add()】，此时ArrayList就不需要移动和复制数组来进行操作了。如果数据量有百万级的时，**速度是会比LinkedList要快的**。(我测试过) 

-  如果

  删除操作

  的位置是在

  中间

  。由于LinkedList的消耗主要是在遍历上，ArrayList的消耗主要是在移动和复制上(底层调用的是arraycopy()方法，是native方法)。    

  -  LinkedList的遍历速度是要慢于ArrayList的复制移动速度的 
  -  如果数据量有百万级的时，**还是ArrayList要快**。

### 3.7、ArrayList的扩容机制？

ArrayList是基于数组的集合，数组的容量在定义的时候确定的。如果数组满了，再插入，就会数组溢出。在插入的时候，会先检查是否需要扩容，如果当前容量+1超过数组长度，就会进行扩容。创建一个1.5倍的新数组，然后把原数组的值拷贝过去。

### 3.8、CopyOnWriteArraylList写时复制

是线程安全版本的ArrayList。采用了一种**读写分离的并发策略**。`CopyOnWriteArrayList`容器允许`并发读`，读操作是无锁的，性能较高。写操作，比如先容器中添加一个元素，则首先将当前容器复制一份，然后在新副本上执行写操作，结束之后，再将原容器的引用指向新容器。（读写分离，读操作无锁，写操作是在副本上执行写操作）

### 3.9、ArrayList的序列化机制？为什么用transient修饰数组？

ArrayList使用`transient`修饰存储元素的`elementData[]`的数组。`transient`关键字的作用是让被修饰的成员属性不被序列化。

> ArrayList为什么不直接序列化元素数组呢？

出于效率的考虑，数组长度可能为100，但实际只用了50，剩下的50不用其实可以不用序列化，这样可以提高序列化和反序列化的效率，还可以节省内存空间。

> ArrayList怎么序列化？

ArrayList通过两个方法`readObject、writeObject`自定义序列化和反序列化策略，实际直接使用两个流`ObjectOutputStream`和`ObjectInputStream`来进行序列化和反序列化。

![微信图片_20220727202006](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207272020078.png)

### 3.10、有哪几种实现ArrayList线程安全的方法？

fail-fast是一种可能触发的机制，实际上，ArrayList的线程安全仍然没有保证，一般，保证ArrayList线程安全的方案有：

- 使用`Vector`代替ArrayList（不推荐，Vector是一个历史遗留类）。
- 使用`Collections.synchronizedList`包装ArrayList，然后操作包装后的List。
- 使用`CopyOnWriteArrayList`（写时复制，线程安全的）代替ArrayList。
- 使用ArrayList的时候，通过`同步机制`去控制ArrayList的读写。

## 四、队列Queue

### 4.1、什么是阻塞队列？

java.util.concurrent.BlockingQueue是一个队列，它支持在检索和删除元素时等待队列变为非空，并在添加元素时等待队列中的空间可用的操作。BlockingQueue 接口是 java 集合框架的一部分，主要用于用于实现生产者消费者问题。我们不需要担心等待 BlockingQueue 中的空间可供生产者使用或对象可供消费者使用，因为它由 BlockingQueue 的实现类处理。Java 提供了几种 BlockingQueue 实现，例如 ArrayBlockingQueue、LinkedBlockingQueue、PriorityBlockingQueue、SynchronousQueue 等.

### 4.2、什么是队列和堆栈，列出它们的区别？

Queue 和 Stack 都用于在处理数据之前存储数据。java.util.Queue是一个接口，其实现类存在于 java 并发包中。队列允许以先进先出 (FIFO) 顺序检索元素，但并非总是如此。还有一个 Deque 接口，允许从队列的两端检索元素。堆栈类似于队列，只是它允许以后进先出 (LIFO) 顺序检索元素。Stack 是一个扩展 Vector 的类，而 Queue 是一个接口。

## 五、Set（无序、不可重复）

### 5.1、Set里的元素是不能重复的，那么用什么方法来区分重复与否呢? 是用==还是equals()?

Set的添加方法利用的HashMap的put()

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}

//
private transient HashMap<E,Object> map;
public HashSet() {
    map = new HashMap<>();
}
```

```java
if (p.hash == hash &&
    ((k = p.key) == key || (key != null && key.equals(k))))
    e = p;
else if (p instanceof TreeNode)
    e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
else {
    for (int binCount = 0; ; ++binCount) {
        if ((e = p.next) == null) {
            p.next = newNode(hash, key, value, null);
            if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                treeifyBin(tab, hash);
            break;
        }
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            break;
        p = e;
    }
}
```

HashMap的put源码，添加元素的时候，如果key（对应的Set集合的元素）相等，则修改value的值（map.put(e, PRESENT)==null；value的值为PRESENT）。而在Set集合中，value值仅仅是一个Object对象罢了(**该对象对Set本身而言是无用的**)。

### 5.2、说说HashSet的底层实现？

HashSet 底层就是基于 HashMap 实现的。（ HashSet 的源码⾮常⾮常少，因为除了 clone() 、 writeObject() 、 readObject() 是 HashSet⾃⼰不得不实现之外，其他⽅法都是直接调⽤ HashMap 中的⽅法。

HashSet的add方法，直接调用HashMap的put方法，将添加的元素作为key，new一个Object作为value，直接调用HashMap的put方法，它会根据返回值是否为空来判断是否插入元素成功。

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

![image-20220727223624142](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207272236464.png)

而在HashMap的putVal方法中，进行了一系列判断，最后的结果是，只有在key在table数组中不存在的时候，才会返回插入的值。

```java
if (e != null) { // existing mapping for key
    V oldValue = e.value;
    if (!onlyIfAbsent || oldValue == null)
        e.value = value;
    afterNodeAccess(e);
    return oldValue;
}
```

