# 并发编程JUC

## 一.线程基础、线程之间的共享和协作

### 什么是进程和线程

**进程是程序运行资源分配的最小单位**

- 进程是操作系统进行资源分配的最小单位,其中资源包括:CPU、内存空间、磁盘IO等,同一进程中的多条线程共享该进程中的全部系统资源,而进程和进程之间是相互独立的。进程是具有一定独立功能的程序关于某个数据集合上的一次运行活动,进程是系统进行资源分配和调度的一个独立单位。
- 进程是程序在计算机上的一次执行活动。当你运行一个程序,你就启动了一个进程。显然,程序是死的、静态的,进程是活的、动态的。进程可以分为系统进程和用户进程。凡是用于完成操作系统的各种功能的进程就是系统进程,它们就是处于运行状态下的操作系统本身,用户进程就是所有由你启动的进程。

**线程是CPU调度的最小单位，必须依赖于进程而存在**

- 线程是进程的一个实体,是CPU调度和分派的基本单位,它是比进程更小的、能独立运行的基本单位。线程自己基本上不拥有系统资源,只拥有一点在运行中必不可少的资源(如程序计数器,一组寄存器和栈),但是它可与同属一个进程的其他的线程共享进程所拥有的全部资源。

**线程无处不在**

- 任何一个程序都必须要创建线程,特别是Java不管任何程序都必须启动一个main函数的主线程; Java Web开发里面的定时任务、定时器、JSP和 Servlet、异步消息处理机制,远程访问接口RM等,任何一个监听事件, onclick的触发事件等都离不开线程和并发的知识。

### CPU核心是和线程数的关系

  多核心:也指单芯片多处理器( Chip Multiprocessors,简称CMP),CMP是由美国斯坦福大学提出的,其思想是将大规模并行处理器中的SMP(对称多处理器)集成到同一芯片内,各个处理器并行执行不同的进程。这种依靠多个CPU同时并行地运行程序是实现超高速计算的一个重要方向,称为并行处理。

  多线程: Simultaneous Multithreading.简称SMT.让同一个处理器上的多个线程同步执行并共享处理器的执行资源。

  核心数、线程数:目前主流CPU都是多核的。增加核心数目就是为了增加线程数,因为操作系统是通过线程来执行任务的,一般情况下它们是1:1对应关系,也就是说四核CPU一般拥有四个线程。但 Intel引入超线程技术后,使核心数与线程数形成1:2的关系。

### CPU时间片轮转机制

  时间片轮转调度是一种最古老、最简单、最公平且使用最广的算法,又称RR调度。每个进程被分配一个时间段,称作它的时间片,即该进程允许运行的时间。

  上下文切换 (context switch) , 其实际含义是任务切换, 或者CPU寄存器切换。当多任务内核决定运行另外的任务时, 它保存正在运行任务的当前状态, 也就是CPU寄存器中的全部内容。这些内容被保存在任务自己的堆栈中, 入栈工作完成后就把下一个将要运行的任务的当前状况从该任务的栈中重新装入CPU寄存器, 并开始下一个任务的运行, 这一过程就是context switch。

如图: 每个任务都是整个应用的一部分, 都被赋予一定的优先级, 有自己的一套CPU寄存器和栈空间

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220524200937130.png" width="30%"/>

### 什么是并行和并发

- 并发:指应用能够交替执行不同的任务,比如单CPU核心下执行多线程并非是同时执行多个任务,如果你开两个线程执行,就是在你几乎不可能察觉到的速度不断去切换这两个任务,已达到"同时执行效果",其实并不是的,只是计算机的速度太快,我们无法察觉到而已.
- 并行:指应用能够同时执行不同的任务,例:吃饭的时候可以边吃饭边打电话,这两件事情可以同时执行

  两者区别:一个是交替执行,一个是同时执行.

### 高并发编程的意义、好处和注意事项

**(1)充分利用CPU的资源**

- 从上面的CPU的介绍,可以看的出来,现在市面上没有CPU的内核不使用多线程并发机制的,特别是服务器还不止一个CPU,如果还是使用单线程的技术做思路,明显就out了。因为程序的基本调度单元是线程,并且一个线程也只能在一个CPU的一个核的一个线程跑,如果你是个i3的CPU的话,最差也是双核心4线程的运算能力:如果是一个线程的程序的话,那是要浪费3/4的CPU性能:如果设计一个多线程的程序的话,那它就可以同时在多个CPU的多个核的多个线程上跑,可以充分地利用CPU,减少CPU的空闲时间,发挥它的运算能力,提高并发量。
- 就像我们平时坐地铁一样,很多人坐长线地铁的时候都在认真看书,而不是为了坐地铁而坐地铁,到家了再去看书,这样你的时间就相当于有了两倍。这就是为什么有些人时间很充裕,而有些人老是说没时间的一个原因,工作也是这样,有的时候可以并发地去做几件事情,充分利用我们的时间,CPU也是一样,也要充分利用。

**(2)加快响应用户的时间**

- 比如我们经常用的迅雷下载,都喜欢多开几个线程去下载,谁都不愿意用一个线程去下载,为什么呢?答案很简单,就是多个线程下载快啊。
- 我们在做程序开发的时候更应该如此,特别是我们做互联网项目,网页的响应时间若提升1s,如果流量大的话,就能增加不少转换量。做过高性能web前端调优的都知道,要将静态资源地址用两三个子域名去加载,为什么?因为每多一个子域名,浏览器在加载你的页面的时候就会多开几个线程去加载你的页面资源,提升网站的响应速度。多线程,高并发真的是无处不在。

**(3)可以使你的代码模块化,异步化,简单化**

- 例如我们实现电商系统，下订单和给用户发送短信、邮件就可以进行拆分，将给用户发送短信、邮件这两个步骤独立为单独的模块，并交给其他线程去执行。这样既增加了异步的操作，提升了系统性能，又使程序模块化,清晰化和简单化。
- 多线程应用开发的好处还有很多,大家在日后的代码编写过程中可以慢慢体会它的魅力。

### 多线程的注意事项

**(1)线程之间的安全性**

- 从前面的章节中我们都知道,在同一个进程里面的多线程是资源共享的,也就是都可以访问同一个内存地址当中的一个变量。例如:若每个线程中对全局变量、静态变量只有读操作,而无写操作,一般来说,这个全局变量是线程安全的:若有多个线程同时执行写操作,一般都需要考虑线程同步,否则就可能影响线程安全。

**(2)线程之间的死锁**

- 为了解决线程之间的安全性引入了Java的锁机制,而一不小心就会产生Java线程死锁的多线程问题,因为不同的线程都在等待那些根本不可能被释放的锁,从而导致所有的工作都无法完成。假设有两个线程,分别代表两个饥饿的人,他们必须共享刀叉并轮流吃饭。他们都需要获得两个锁:共享刀和共享叉的锁。
- 假如线程A获得了刀,而线程B获得了叉。线程A就会进入阻塞状态来等待获得叉,而线程B则阻塞来等待线程A所拥有的刀。这只是人为设计的例子,但尽管在运行时很难探测到,这类情况却时常发生

**(3)线程太多了会将服务器资源耗尽形成死机当机**

- 线程数太多有可能造成系统创建大量线程而导致消耗完系统内存以及CPU的“过渡切换”,造成系统的死机,那么我们该如何解决这类问题呢?
- 某些系统资源是有限的,如文件描述符。多线程程序可能耗尽资源,因为每个线程都可能希望有一个这样的资源。如果线程数相当大,或者某个资源的侯选线程数远远超过了可用的资源数则最好使用资源池。一个最好的示例是数据库连接池。只要线程需要使用一个数据库连接,它就从池中取出一个,使用以后再将它返回池中。资源池也称为资源库。

### Java中线程基础

#### 三种线程的启动方式

##### 1.继承Thread类创建线程类

- 定义[Thread类](https://so.csdn.net/so/search?q=Thread%E7%B1%BB&spm=1001.2101.3001.7020)的子类，并重写该类的run方法，该run方法的方法体就代表了线程要完成的任务。因此把run()方法称为执行体。
- 创建Thread[子类](https://so.csdn.net/so/search?q=%E5%AD%90%E7%B1%BB&spm=1001.2101.3001.7020)的实例，即创建了线程对象。
- 调用线程对象的start()方法来启动该线程。
```java
public class Test1_thread {
    
    public static void main(String[] args) throws InterruptedException {
        System.out.println("主方法的开头...");
        //外部类线程启动
        MyThread mt = new MyThread();
        mt.start();     //子程序 运行
        //内部类线程启动
        InnerThread it = new InnerThread();
        it.start();     //启动线程要用start(); -->jvm会自动的调用线程中的run()

        //匿名内部类
        Thread nmThread = new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i <= 100; i++) {
                    System.out.println("匿名内部类中j的值为：" + i);
                    try {
                        Thread.sleep(5000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        });
        nmThread.start();

        for (int i = 0; i < 5; i++) {
            Thread.sleep(1000);
            System.out.println("主方法在运行...");
        }

    }

    //内部类 只有Test1_thread会用到
    static class InnerThread extends Thread {
        @Override
        public void run() {
            for (int i = 0; i <= 100; i++) {
                System.out.println("内部类中j的值为：" + i);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

//方案一：外部类 写一个类继承自Thread， 重写run()方法。在这个方法加入耗时的操作或阻塞操作
class MyThread extends Thread {
    @Override
    public void run() {
        for (int i = 0; i <= 100; i++) {
            System.out.println("i的值为：" + i);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
//缺点：java是单继承，以上方法影响类的扩展性
```
##### 2.通过Runnable接口创建线程类

- 定义runnable接口的实现类，并重写该接口的run()方法，该run()方法的方法体同样是该线程的线程执行体。
- 创建 Runnable实现类的实例，并依此实例作为Thread的target来创建Thread对象，该Thread对象才是真正的线程对象。
- 调用线程对象的start()方法来启动该线程。
```java
package com.zhulin.thread;

import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * @program:大数据工作区
 * @description:
 * @author:ZHULIN
 * @create:2022-01-10 20:44
 */
public class Test2_Thread_runnable {
    public static void main(String[] args) {
        //方法一：继承Thread类
        ShowTimeThread stt = new ShowTimeThread();
        stt.setName("线程1--显示时间");     //设置线程名
        stt.setPriority(1);     //可以设置优先级（理论上） 1-10
        stt.start();

        //实现二：实现runnable接口  任务对象
        ShowTimeThread2 task = new ShowTimeThread2();
        //创建线程对象        任务      线程名
        Thread t = new Thread(task, "线程二--继承Runnable接口");
        t.setPriority(10);
        t.start();      //t启动，jvm就会自动回调它配置 task中的run()

        //实现二：换成匿名内部类写法
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日 hh:mm:ss");
                Date d = null;
                while (true) {
                    d = new Date();
                    System.out.println(Thread.currentThread().getName() + "当前的时间为：" + sdf.format(d));
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }, "线程3--匿名内部类");
        t2.start();

        //写法4：函数式编程  -> lambda写法
        Thread t3 = new Thread(() -> {
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日 hh:mm:ss");
            Date d = null;
            while (true) {
                d = new Date();
                System.out.println(Thread.currentThread().getName() + "当前的时间为：" + sdf.format(d));
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "线程4--函数式编程");
        t3.start();
    }
}

/**
 * 显示时间  线程类
 */
class ShowTimeThread extends Thread {
    @Override
    public void run() {
        //耗时操作
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日 hh:mm:ss");
        Date d = null;
        //死循环
        while (true) {
            d = new Date();
            //Thread.currentThread().getName()  获取线程的名字
            System.out.println(Thread.currentThread().getName() + "当前时间为：" + sdf.format(d));
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

/**
 * 方案二：写一个类(任务类) 实现Runnable接口，重写run()
 */
class ShowTimeThread2 implements Runnable {
    @Override
    //run加入在线程中完成的操作
    public void run() {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日 hh:mm:ss");
        Date d = null;
        while (true) {
            d = new Date();
            System.out.println(Thread.currentThread().getName() + "当前时间为：" + sdf.format(d));
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
##### 3.通过Callable和Future创建线程

- 创建Callable接口的实现类，并实现call()方法，该call()方法将作为线程执行体，并且有返回值。
- 创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该FutureTask对象封装了该Callable对象的call()方法的返回值。
- 使用FutureTask对象作为Thread对象的target创建并启动新线程。
- 调用FutureTask对象的get()方法来获得子线程执行结束后的返回值
```java
public class Test03_callable {
    public static void main(String[] args) {
        //FutureTask对象
        //方式一：内部类
        FutureTask<Integer> task = new FutureTask<Integer>(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                int count = 0;
                for (int i = 0; i <= 100; i++) {
                    Thread.sleep(100);
                    count += i;
                }
                return count;
            }
        });

        //方式二：Lambda表达式
        //FutureTask<Integer> task = new FutureTask<Integer>(() -> {
        //    int count = 0;
        //    for (int i = 0; i <= 100; i++) {
        //        Thread.sleep(100);
        //        count += i;
        //    }
        //    return count;
        //});

        //创建线程 与一个FutureTask任务绑定
        Thread thread = new Thread(task);
        //启动线程
        thread.start();
        try {
            //获取线程返回值
            //System.out.println("1+2+3+...+100=" + task.get());  //等到两种情况跳出  1.任务执行出异常  2.任务执行完
            System.out.println("1+2+3+...+100=" + task.get(20, TimeUnit.SECONDS)); //超时
        } catch (InterruptedException | ExecutionException | TimeoutException e) {
            e.printStackTrace();
        }
        //因为调用了get() 这是阻塞式的方法，它要等出结果后，主线程才会继续
        System.out.println("主程序中其他的代码......");
    }
}
```
#### 创建线程的三种方式的对比：

##### （1）采用实现Runnable、Callable接口的方式创建多线程时，

**优势是：**
  线程类只是实现了Runnable接口或Callable接口，还可以继承其他类。在这种方式下，多个线程可以共享同一个target对象，所以非常适合多个相同线程来处理同一份资源的情况，从而可以将CPU、代码和数据分开，形成清晰的模型，较好地体现了面向对象的思想。

**劣势是：**
  编程稍微复杂，如果要访问当前线程，则必须使用Thread.currentThread()方法。

##### （2）使用继承Thread类的方式创建多线程时

**优势是：**
  编写简单，如果需要访问当前线程，则无需使用Thread.currentThread()方法，直接使用this即可获得当前线程。

**劣势是：**
  线程类已经继承了Thread类，所以不能再继承其他父类

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

### 线程的常用方法

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

  和主线程共死，finally不能保证一定执行。Daemon（守护）线程是一种支持型线程，因为它主要被用作程序中后台调度以及支持性工作。这意味着，当一个Java虚拟机中不存在非Daemon线程的时候，Java虚拟机将会退出。可以通过调用Thread.setDaemon(true)将线程设置为Daemon线程。我们一般用不上，比如垃圾回收线程就是Daemon线程。

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
### 线程间的共享

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
