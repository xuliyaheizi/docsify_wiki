#### 线程的中断

  安全的中止则是其他线程通过调用某个线程A的interrupt()方法对其进行中断操作, 中断好比其他线程对该线程打了个招呼，“A，你要中断了”，不代表线程A会立即停止自己的工作，同样的A线程完全可以不理会这种中断请求。因为java里的线程是协作式的，不是抢占式的。线程通过检查自身的中断标志位是否被置为true来进行响应。

  线程通过方法isInterrupted()来进行判断是否被中断，也可以调用静态方法Thread.interrupted()来进行判断当前线程是否被中断，不过Thread.interrupted()会同时将中断标识位改写为false。

  如果一个线程处于了阻塞状态（如线程调用了thread.sleep、thread.join、thread.wait等），则在线程在检查中断标示时如果发现中断标示为true，则会在这些阻塞方法调用处抛出InterruptedException异常，并且在抛出异常后会立即将线程的中断标示位清除，即重新设置为false。
不建议自定义一个取消标志位来中止线程的运行。因为run方法里有阻塞调用时会无法很快检测到取消标志，线程必须从阻塞调用返回后，才会检查这个取消标志。这种情况下，使用中断会更好，因为，

1. 一般的阻塞方法，如sleep等本身就支持中断的检查，
1. 检查中断位的状态和检查取消标志位没什么区别，用中断位的状态还可以避免声明取消标志位，减少资源的消耗。

注意：处于死锁状态的线程无法被中断
```java
public class Test5_Interrupt {
    public static void main(String[] args) {
        //通过interrupted()方法检测线程是否被中断
        //Thread.currentThread() 主线程
        System.out.println(Thread.currentThread().getName() + "线程是否中断：" + Thread.interrupted());
        //设置线程中断
        Thread.currentThread().interrupt();

        //Thread.currentThread().stop();
        //通过interrupted() 方法检测线程是否被中断
        System.out.println(Thread.currentThread().getName() + "线程是否中断:" + Thread.interrupted());
        //检测interrupted()是否会清除线程状态
        System.out.println(Thread.currentThread().getName() + "线程是否中断：" + Thread.interrupted());
    }
    /*
    main线程是否中断：false
    main线程是否中断:true
    main线程是否中断：false
     */
}
```
#### 线程的终止

- 线程自然终止：自然执行完或抛出未处理异常
- stop()，resume(),suspend()已不建议使用，stop()会导致线程不会正确释放资源，suspend()容易导致死锁,它不释放资源。
- 而java线程是协作式，而非抢占式,锁是不可剥夺的. 
- 调用一个线程的interrupt() 方法中断一个线程，并不是强行关闭这个线程，只是跟这个线程打个招呼，将线程的中断标志位置为true，线程是否中断，由线程本身决定。
- isInterrupted() 判定当前线程是否处于中断状态。
- static方法interrupted() 判定当前线程是否处于中断状态，同时中断标志位改为false。
- 方法里如果抛出InterruptedException，线程的中断标志位会被复位成false，如果确实是需要中断线程，要求我们自己在catch语句块里再次调用interrupt()。

### 1.6、线程的常用方法

#### yield()方法

  使当前线程让出CPU占有权，但让出的时间是不可设定的。也不会释放锁资源。注意：并不是每个线程都需要这个锁的，而且执行yield( )的线程不一定就会持有锁，我们完全可以在释放锁后再调用yield方法。

  所有执行yield()的线程有可能在进入到就绪状态后会被操作系统再次选中马上又被执行。

```java
public class Test12_yield {
    //挂起线程
    public static void main(String[] args) {
        YieldOne y1 = new YieldOne();
        YieldOne y2 = new YieldOne();

        Thread t1 = new Thread(y1, "a");
        Thread t2 = new Thread(y2, "b");
        t1.setPriority(10);
        t1.start();

        t2.setPriority(1);  //第二个线程的优先级高
        t2.start();
    }
}

class YieldOne implements Runnable {
    @Override
    public void run() {
        if ("a".equals(Thread.currentThread().getName())) {
            Thread.yield();      //TODO:yield只会将执行权放给you先级高的线程	
            try {
                Thread.sleep(100);  //TODO:sleep不管you先级，只要调用sleep，则当前线程睡，其他接过执行权
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        for (int i = 0; i < 10; i++) {
            System.out.println(Thread.currentThread().getName() + ": " + i);
        }
    }
}
```
#### join方法

  把指定的线程加入到当前线程，可以将两个交替执行的线程合并为顺序执行。
  比如在线程B中调用了线程A的Join()方法，直到线程A执行完毕后，才会继续执行线程B。(此处为常见面试考点)

```java
public class Test13_join {
    //main线程
    public static void main(String[] args) throws InterruptedException {
        //当前是main线程
        LifeCircle lc = new LifeCircle();
        System.out.println(lc.isAlive());  //线程状态值
        lc.start();
        System.out.println(lc.isAlive());
        lc.join();   //TODO:让lc先运行完，再执行main
        System.out.println("主程序的其他操作....." + Thread.currentThread().getName());
        System.out.println(lc.isAlive());
    }
}

class LifeCircle extends Thread {
    @Override
    public void run() {
        int i = 0;
        while ((++i) < 10) {
            System.out.println(Thread.currentThread().getName() + ":" + i);
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
#### 守护线程

  和主线程共死，finally不能保证一定执行。Daemon（守护）线程是一种支持型线程，因为它主要被用作程序中后台调度以及支持性工作。这意味着，当一个Java虚拟机中不存在非Daemon线程的时候，Java虚拟机将会退出。可以通过调用`Thread.setDaemon(true)`将线程设置为Daemon线程。我们一般用不上，比如垃圾回收线程就是Daemon线程。

  Daemon线程被用作完成支持性工作，但是在Java虚拟机退出时Daemon线程中的finally块并不一定会执行。在构建Daemon线程时，不能依靠finally块中的内容来确保执行关闭或清理资源的逻辑。

```java
public class Test9_daemon {
	public static void main(String[] args) {
		Thread t1 = new CommonTread();
		//新建任务，再绑定到一个线程上
		Thread t2 = new Thread(new MyDaemon());
		//TODO: 设置守护线程
		t2.setDaemon(true);
		//正确做法：线程运行前设置守护线程
		t2.start();
		t1.start();
	}
}
//线程
class CommonTread extends Thread{
	@Override
	public void run() {
		for(int i=0;i<5;i++){
			System.out.println("用户线程第"+i+"次执行!");
			try {
				Thread.sleep(10);
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	}
}

//任务类
class MyDaemon implements Runnable{
	@Override
	public void run() {
		for(int i=0;i<20;i++){
			System.out.println("守护线程第"+i+"次执行!");
			try {
				Thread.sleep(10);
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	}
}
```
### 1.7、线程间的共享

#### synchronized内置锁

- 可以保证可见性和原子性. 
- 线程开始运行，拥有自己的栈空间，就如同一个脚本一样，按照既定的代码一步一步地执行，直到终止。但是，每个运行中的线程，如果仅仅是孤立地运行，那么没有一点儿价值，或者说价值很少，如果多个线程能够相互配合完成工作，包括数据之间的共享，协同处理事情。这将会带来巨大的价值。
- Java支持多个线程同时访问一个对象或者对象的成员变量，关键字synchronized可以修饰方法或者以同步块的形式来进行使用。
- 主要确保多个线程在同一个时刻，只能有一个线程处于方法或者同步块中，它保证了线程对变量访问的可见性和排他性，又称为内置锁机制。

**对象锁和类锁：**

- 对象锁是用于对象实例方法，或者一个对象实例上的。
- 类锁是用于类的静态方法或者一个类的class对象上的。我们知道，类的对象实例可以有很多个，但是每个类只有一个class对象，所以不同对象实例的对象锁是互不干扰的，但是每个类只有一个类锁。
- 但是有一点必须注意的是，其实类锁只是一个概念上的东西，并不是真实存在的，类锁其实锁的是每个类的对应的class对象。
- 类锁和对象锁之间也是互不干扰的。
```java
public class Test15_synchronized {
    public static void main(String[] args) {
        //创建资源类
        SellTickOp sto = new SellTickOp(40);
        Thread counter1 = new Thread(sto,"张三");
        Thread counter2 = new Thread(sto,"李四");
        Thread counter3 = new Thread(sto,"王五");
        counter1.start();
        counter2.start();
        counter3.start();
    }
}

//synchronized同步
//卖票的任务
class SellTickOp implements Runnable{
    int tickets;  //总票数
    Random r = new Random();

    public SellTickOp(int tickets){
        this.tickets=tickets;
    }

    @Override
    public void run() {
        while (true){
            synchronized (this){
                //块级同步  同步两种：方法级  synchronized  void 方法名(){}
                //块级：将同步加到要同步的代码上，注意 synchronized(参照对象){   }
                if(tickets>0){
                    System.out.println(Thread.currentThread()+"在sell第"+(tickets--)+"张票");
                    try {
                        //模拟出票的时间
                        Thread.sleep(r.nextInt(800));
                        wait(r.nextInt(800));
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }else{
                    return;
                }
            }
        }
    }
}
```
#### volatile

  适合于只有一个线程写，多个线程读的场景，因为它只能确保可见性,不能确保原子性
```java
/**
 *
 *类说明：演示violate无法提供操作的原子性
 *   结果说明: 每次的运行结果都不一样.
 */
public class VolatileUnsafe {
	
	private static class VolatileVar implements Runnable {
		private volatile int a = 0;
	    @Override
	    public void run() {
	    	String threadName = Thread.currentThread().getName();
	    	a = a++;
	    	System.out.println(threadName+":======"+a);
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			a = a+1;
	    	System.out.println(threadName+":======"+a);
	    }
	}
	
    public static void main(String[] args) {

    	VolatileVar v = new VolatileVar();

        Thread t1 = new Thread(v);
        Thread t2 = new Thread(v);
        Thread t3 = new Thread(v);
        Thread t4 = new Thread(v);
        t1.start();
        t2.start();
        t3.start();
        t4.start();
    }
}
```
#### ThreadLocal

  线程变量，每个线程有自己的变量副本。可以理解为是个map，类型 Map<Thread,Integer>，经常用于联接池中， 这样每个线程中保存自己的使用的联接。
```java
public class Test21_threadlocal2 {
    static ThreadLocal<Integer> arg = new ThreadLocal<>();

    public static void main(String[] args) {
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                //初始化参数
                arg.set(0);
                task1();
            }
        });
        t1.start();
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                //初始化参数
                arg.set(1);
                task1();
            }
        });
        t2.start();
    }

    public static void task1() {
        task2();
    }

    public static void task2() {
        System.out.println(arg.get());
    }
}
```

## 二、Lock接口

wait虚假唤醒

