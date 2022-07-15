# Collection-ArrayList源码阅读

ArrayList继承自 `AbstractList`，实现了 List 接口。底层基于`数组`实现`容量大小动态变化`（扩容机制）。允许 `null `的存在。同时还实现了 `RandomAccess`、`Cloneable`、`Serializable `接口，所以ArrayList 是支持`快速访问`、`复制`、`序列化`的。每个ArrayList都有一个容量（DEFAULT_CAPACITY默认初始容量为10），表示底层数组的实际大小，容器内存储的元素个数不能多于当前容量。容量不足的时候，容器会自动增大底层数组的大小。底层数组是一个Object数组，以便能够容纳任何类型的对象。

## 底层数据结构

```java
transient Object[] elementData; // non-private to simplify nested class access
//底层数组对象Object类型，可以容纳任何类型的对象
private int size;
```

## 构造函数

```java
//构造一个具有指定初始容量的List列表
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        //指定容量不为0，则创建一个指定容量大小的Object数组对象（Object[initialCapacity]）
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        //指定容量为0的时候，创建一个空的Object对象数组
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "initialCapacity);
    }
}

//使用默认初始容量，构造一个容量为10的列表
public ArrayList() {
    //空数组
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

//按照集合的迭代器返回的顺序构造一个包含指定集合元素的列表。
public ArrayList(Collection<? extends E> c) {
    Object[] a = c.toArray();
    if ((size = a.length) != 0) {
        if (c.getClass() == ArrayList.class) {
            elementData = a;
        } else {
            elementData = Arrays.copyOf(a, size, Object[].class);
        }
    } else {
        // replace with empty array.
        elementData = EMPTY_ELEMENTDATA;
    }
}
```

## 扩容机制

向数组添加元素时，都会去检查添加后元素的个数是否超过当前数组的长度。若超出，数组会进行扩容。数组扩容通过一个公开的方法ensureCapacity(int minCapacity)来实现。在实际添加大量元素前，我也可以使用ensureCapacity来手动增加ArrayList实例的容量，以减少递增式再分配的数量。

`懒加载模式`：ArrayList()创建ArrayList对象时，并没有初始化化底层数组elementData，等到调用add(E e)方法的时候再初始化数组elementData。节省内存。

**创建ArrayList**

```java
//使用默认初始容量，构造一个容量为10的列表
public ArrayList() {
    //空数组
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

**调用add(E e)**

```java
public boolean add(E e) {
    //确认elementData数组容量是否足够
    ensureCapacityInternal(size + 1);  // 第一次调用add方法时，size为0!!
    elementData[size++] = e;
    return true;
}
```

**调用ensureCapacityInternal**

```java
private void ensureCapacityInternal(int minCapacity) {
    //先判断是否第一次调用add(E e)方法，初始化对象数组大小；再判断是否需要扩容
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

//calculateCapacity
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    //如果elementData为“{}”，即第一次调用add(E e)方法，重新定义minCapacity的值，赋值为DEFAULT_CAPACITY=10
    //第一次调用add(E e)方法时，定义底层数组elementData的长度为10
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    //不是第一次调用add(E e)方法，则返回容量值
    return minCapacity;
}
```

**调用ensureExplicitCapacity判断是否需要扩容**

```java
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // 第一次进入时，minCapacity=10,elementData.length=0,对数组进行扩容
	// 之后再进入时，minCapacity=size+1，elementData.length=10(每次扩容后会改变)，
	// 需要minCapacity>elementData.length成立，才能扩容
    if (minCapacity - elementData.length > 0)
        //扩容
        grow(minCapacity);
}
```

**调用grow扩容方法**

```java
private void grow(int minCapacity) {
    //将数组长度赋值给oldCapacity
    int oldCapacity = elementData.length;
    //将oldCapacity右移一位（/2）再加上oldCapacity，相当于newCapacity=1.5oldCapacity（不考虑精度损失）
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 如果newCapacity还是小于minCapacity，直接将minCapacity赋值给newCapacity
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    // 特殊情况：newCapacity的值过大，直接将整型最大值赋给newCapacity，
	// 即newCapacity=Integer.MAX_VALUE
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // 将elementData的数据拷贝到扩容后的数组
    elementData = Arrays.copyOf(elementData, newCapacity);
}

//如果大于临界值，进行整型最大值的分配
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
    MAX_ARRAY_SIZE;
}
```

**数组复制方法**

```java
//add(E e) 在数组末尾添加元素
//判断原数组类型是否是对象数组
T[] copy = ((Object)newType == (Object)Object[].class)
    ? (T[]) new Object[newLength]	//类型符合则开辟一个扩容后大小的数组
    : (T[]) Array.newInstance(newType.getComponentType(), newLength);	//getComponentType()返回数组元素的class对象
//原数组；
System.arraycopy(original, 0, copy, 0,Math.min(original.length, newLength));

//add(int index, E element) 将数据插入到指定位置，操作性能非常低下，由于要开辟新数组复制元素。
//原数组；从原数组的哪个位置开始复制；目标数组；复制到目标数组的哪个位置；要复制的原数组中数组元素的数量
System.arraycopy(elementData, index, elementData, index + 1,size - index);
```

**总结：**ArrayList创建ArrayList对象时，不会定义底层数组的长度（除有参构造方法），当第一次调用add(E e)方法时，会初始化定义底层数组的长度为10，之后调用add(E e)方法时，会将添加后的数组长度与原数组长度比较，判断是否需要扩容。如需要扩容会调用grow(int minCapacity)方法进行扩容，长度变为原来的1.5倍。

## 添加方法，add()、addAll()

这两个方法都是向容器中添加新元素，这可能会导致`capacity`不足，因此在添加元素之前，都需要进行剩余空间检查，如果需要则自动扩容。扩容操作最终是通过grow()方法完成的。

`add(int index, E e)`需要先对元素进行移动，然后完成插入操作，也就意味着该方法有着线性的时间复杂度。

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
```

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207141548517.png" alt="image-20220714154742451" style="width:50%;" /><img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207141549862.png" alt="image-20220714154940501" style="width:50%;" />

`addAll()`方法能够一次添加多个元素，根据位置不同也有两个版本，一个是在末尾添加的`addAll(Collection<? extends E> c)`方法，一个是从指定位置开始插入的`addAll(int index, Collection<? extends E> c)`方法。跟`add()`方法类似，在插入之前也需要进行空间检查，如果需要则自动扩容；如果从指定位置插入，也会存在移动元素的情况。 `addAll()`的时间复杂度不仅跟插入元素的多少有关，也跟插入的位置相关。

```java
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}

public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount

    int numMoved = size - index;
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
                         numMoved);

    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```

## Set与Get方法

```java
//返回指定列表的元素
public E get(int index) {
    rangeCheck(index);

    return elementData(index);
}

//将此列表中指定位置的元素替换为指定元素。返回先前该指定处的元素
public E set(int index, E element) {
    rangeCheck(index);

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```

## remove删除方法

一个是`remove(int index)`删除指定位置的元素，另一个是`remove(Object o)`删除第一个满足`o.equals(elementData[index])`的元素。删除操作是`add()`操作的逆过程，需要将删除点之后的元素向前移动一个位置。需要注意的是为了让GC起作用，必须显式的为最后一个位置赋`null`值。

```java
//删除指定位置的元素
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    //记录指定位置的元素
    E oldValue = elementData(index);
	//记录指定位置后的元素数量
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; //清除该位置的引用，让GC起作用
	//返回指定位置的元素
    return oldValue;
}

//删除数组第一个满足o.equals(elementData[index])的元素
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}
```

