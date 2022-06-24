# Curator

为了更好的实现java操作zookeeper服务器，后来出现Curator框架，非常的强大，目前已经是apache的顶级项目，里面提供了更多丰富的操作。例如：session超时重连，主从选举，分布式计数器，分布式锁等适用于各种复杂的zookeeper场景的API封装。

**依赖引入**

```xml
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>4.3.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.3.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-client</artifactId>
    <version>4.3.0</version>
</dependency>
```

## 一、Zookeeper分布式锁实现

**创建客户端**

```java
public static CuratorFramework curatorFramework() {
    CuratorFramework curatorFramework = CuratorFrameworkFactory
        .builder()
        .connectString("zhulinz.top:2181")
        .sessionTimeoutMs(15000)
        .connectionTimeoutMs(20000)
        .retryPolicy(new ExponentialBackoffRetry(1000, 10))
        .build();
    curatorFramework.start();
    return curatorFramework;
}
```

```java
public static void main(String[] args) throws Exception {
    CuratorFramework client = curatorFramework();
    InterProcessMutex lock = new InterProcessMutex(client, "/locks/mylock");
    try {
        lock.acquire();
        System.out.println("加锁成功");
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        lock.release();
    }
}
```

### Curator加锁流程

```java
//获取锁
public void acquire() throws Exception {
    //无限等待的阻塞方法
    if (!this.internalLock(-1L, (TimeUnit)null)) {
        throw new IOException("Lost connection while trying to acquire lock: " + this.basePath);
    }
}

//acquire调用，传入超时时间-1和单位null，表示如果加锁不成功会一直阻塞直至加锁成功，不会超时。
private boolean internalLock(long time, TimeUnit unit) throws Exception {
    //获取当前线程
    Thread currentThread = Thread.currentThread();
    //使用threadData存储线程重入的情况  ConcurrentMap<Thread, LockData> threadData 一个线程安全，并且是一个高效的HashMap，支持并发访问
    LockData lockData = (LockData)this.threadData.get(currentThread);
    if (lockData != null) {
        //同一线程再次acquire，首先判断当前的映射表内（threadData）是否有该线程的锁信息，如果有则原子+1，然后返回
        lockData.lockCount.incrementAndGet();
        return true;
    } else {
        // 映射表内没有对应的锁信息，尝试通过LockInternals获取锁
        String lockPath = this.internals.attemptLock(time, unit, this.getLockNodeBytes());
        if (lockPath != null) {
            // 成功获取锁，记录信息到映射表	
            LockData newLockData = new LockData(currentThread, lockPath);
            this.threadData.put(currentThread, newLockData);
            return true;
        } else {
            return false;
        }
    }
}

//存储锁信息，zk中一个临时顺序节点对应一个锁，但让锁生效激活需要排队
private static class LockData {
    final Thread owningThread;
    final String lockPath;
    final AtomicInteger lockCount;

    private LockData(Thread owningThread, String lockPath) {
        this.lockCount = new AtomicInteger(1); //分布式锁重入次数
        this.owningThread = owningThread;
        this.lockPath = lockPath;
    }
}
```

**获取锁流程**

```java
//尝试获取锁，并返回锁对应的zk临时顺序节点的路径
String attemptLock(long time, TimeUnit unit, byte[] lockNodeBytes) throws Exception {
    //程序开始时间
    long startMillis = System.currentTimeMillis();
    //unit为null时，无限等待
    Long millisToWait = unit != null ? unit.toMillis(time) : null;
    //zk创建节点时的数据
    byte[] localLockNodeBytes = this.revocable.get() != null ? new byte[0] : lockNodeBytes;
    //当前重试次数，与CuratorFramework的重试策略有关
    int retryCount = 0;
    //在Zookeeper中创建的临时顺序节点的路径，相当于一把待激活的分布式锁 
    //激活条件：同级目录子节点，名称排序最小（排队，公平锁）
    String ourPath = null;
    boolean hasTheLock = false;
    //是否已经完成尝试获取分布式锁的操作
    boolean isDone = false;
    while(!isDone) {
        isDone = true;
        try {
            //在Zookeeper中创建临时顺序节点
            ourPath = this.driver.createsTheLock(this.client, this.path, localLockNodeBytes);
            //循环等待来激活分布式锁，实现锁的公平性
            hasTheLock = this.internalLockLoop(startMillis, millisToWait, ourPath);
        } catch (KeeperException.NoNodeException var14) {
            // 容错处理，不影响主逻辑的理解，可跳过 
            // 因为会话过期等原因，StandardLockInternalsDriver因为无法找到创建的临时顺序节点而抛出NoNodeException异常
            if (!this.client.getZookeeperClient().getRetryPolicy().allowRetry(retryCount++, System.currentTimeMillis() - startMillis, RetryLoop.getDefaultRetrySleeper())) {
                throw var14;
            }
            //程序出错，表示获取分布式锁未完成
            isDone = false;
        }
    }
    return hasTheLock ? ourPath : null;
}

//创建临时有序节点            zookeeper客户端           节点路径      节点数据
public String createsTheLock(CuratorFramework client, String path, byte[] lockNodeBytes) throws Exception {
    String ourPath;
    //判断节点数据是否为空，不为null则作为数据节点内容，否则采用默认内容（IP地址）
    if (lockNodeBytes != null) {
        // creatingParentContainersIfNeeded：用于创建容器节点 
        // withProtection：临时子节点会添加GUID前缀
        ourPath = (String)((ACLBackgroundPathAndBytesable)client                        
                           .create().creatingParentContainersIfNeeded()
                           .withProtection()
                           .withMode(CreateMode.EPHEMERAL_SEQUENTIAL)).forPath(path, lockNodeBytes);
    } else {
        // CreateMode.EPHEMERAL_SEQUENTIAL：临时顺序节点，Zookeeper能保证在节点产生的顺序性 
        // 依据顺序来激活分布式锁，从而也实现了分布式锁的公平性
        ourPath = (String)((ACLBackgroundPathAndBytesable)client                           						  		
                           .create().creatingParentContainersIfNeeded()
                           .withProtection()
                           .withMode(CreateMode.EPHEMERAL_SEQUENTIAL)).forPath(path);
    }
    return ourPath;
}

//循环等待来激活分布式锁，实现锁的公平性
private boolean internalLockLoop(long startMillis, Long millisToWait, String ourPath) throws Exception {
    boolean haveTheLock = false; //是否已经持有分布式锁
    boolean doDelete = false; //是否需要删除子节点
    try {
        if (this.revocable.get() != null) {
            ((BackgroundPathable)this.client.getData().usingWatcher(this.revocableWatcher)).forPath(ourPath);
        }
        //在没有获得锁的情况下持续循环
        while(this.client.getState() == CuratorFrameworkState.STARTED && !haveTheLock) {
            //获取排序后的子节点列表
            List<String> children = this.getSortedChildren();
            //获取前面自己创建的临时顺序子节点的名称
            String sequenceNodeName = ourPath.substring(this.basePath.length() + 1);
            // 实现锁的公平性的核心逻辑
            PredicateResults predicateResults = this.driver.getsTheLock(this.client, children, sequenceNodeName, this.maxLeases);
            //获得了锁，中断循环，返回上层
            if (predicateResults.getsTheLock()) {
                haveTheLock = true;
            } else {
                //加锁失败，监听上一临时顺序节点
                String previousSequencePath = this.basePath + "/" + predicateResults.getPathToWatch();
                //同步锁（同步代码块）
                synchronized(this) {
                    try {
                        //exists()会导致导致资源泄漏，因此exists()可以监听不存在的 ZNode，因此采用getData() 
                        //上一临时顺序节点如果被删除，会唤醒当前线程继续竞争锁，正常情况下 能直接获得锁，因为锁是公平的
                        ((BackgroundPathable)this.client.getData().usingWatcher(this.watcher)).forPath(previousSequencePath);
                        //判断是否有超时机制
                        if (millisToWait == null) {
                            //不限时等待
                            this.wait();
                        } else {
                            millisToWait = millisToWait - (System.currentTimeMillis() - startMillis);
                            startMillis = System.currentTimeMillis();
                            if (millisToWait > 0L) {
                                //限时等待被唤醒
                                this.wait(millisToWait);
                            } else {
                                //获取锁超时，标记删除之前创建的临时顺序节点
                                doDelete = true;
                                break;
                            }
                        }
                    } catch (KeeperException.NoNodeException var19) {
                    }
                }
            }
        }
    } catch (Exception var21) {
        ThreadUtils.checkInterrupted(var21);
        doDelete = true; // 标记删除，在finally删除之前创建的临时顺序节点（后台不断尝试）
        throw var21; // 重新抛出，尝试重新获取锁
    } finally {
        if (doDelete) {
            this.deleteOurPath(ourPath); //删除当前节点
        }

    }
    return haveTheLock;
}

// 实现锁的公平性的核心逻辑
public PredicateResults getsTheLock(CuratorFramework client, List<String> children, String sequenceNodeName, int maxLeases) throws Exception {
    //之前创建的临时顺序节点在排序后的子节点列表中的索引 判断当前节点的顺序来决定加锁成功与否。
    int ourIndex = children.indexOf(sequenceNodeName);
    // 校验之前创建的临时顺序节点是否有效
    validateOurIndex(sequenceNodeName, ourIndex);
    // 锁公平性的核心逻辑
    // 由InterProcessMutex的构造函数可知，maxLeases为1，即只有ourIndex为0时，线程才能持有锁，或者说该线程创建的临时顺序节点激活了锁
    // Zookeeper的临时顺序节点特性能保证跨多个JVM的线程并发创建节点时的顺序性，越早创建临时顺序节点成功的线程会更早地激活锁或获得锁
    boolean getsTheLock = ourIndex < maxLeases;
    // 如果已经获得了锁，则无需监听任何节点，否则需要监听上一顺序节点（ourIndex-1） 
    // 因为锁是公平的，因此无需监听除了（ourIndex-1）以外的所有节点，这是为了减少羊群效应，非常巧妙的设计！！
    String pathToWatch = getsTheLock ? null : (String)children.get(ourIndex - maxLeases);
    // 返回获取锁的结果，交由上层继续处理（添加监听等操作）
    return new PredicateResults(pathToWatch, getsTheLock);
}


```

**对获取的子节点进行排序**

```java
//获取排序后的子节点列表
public static List<String> getSortedChildren(CuratorFramework client, String basePath, final String lockName, final LockInternalsSorter sorter) throws Exception {
    try {
        List<String> children = (List)client.getChildren().forPath(basePath);
        List<String> sortedList = Lists.newArrayList(children);
        Collections.sort(sortedList, new Comparator<String>() {
            public int compare(String lhs, String rhs) {
                return sorter.fixForSorting(lhs, lockName).compareTo(sorter.fixForSorting(rhs, lockName));
            }
        });
        return sortedList;
    } catch (KeeperException.NoNodeException var6) {
        return Collections.emptyList();
    }
}
```

