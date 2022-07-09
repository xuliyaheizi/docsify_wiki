# JVM虚拟机原理与调优

## 一、JVM概念

### 什么是Java虚拟机？

 虚拟机是一种抽象化的计算机，通过在实际的计算机上仿真模拟各种计算机功能来实现的。Java虚拟机有自己完善的硬体架构，如处理器、堆栈、寄存器等，还具有相应的指令系统。Java虚拟机屏蔽了与具体操作系统平台相关的信息，使得Java程序只需生成在Java虚拟机上运行的目标代码（字节码），就可以在多种平台上不加修改地运行。

[JVM](https://so.csdn.net/so/search?q=JVM&spm=1001.2101.3001.7020)（Java虚拟机）是一个抽象的计算模型。就如同一台真实的机器，它有自己的指令集和执行引擎，可以在运行时操控内存区域。目的是为构建在其上运行的应用程序提供一个运行环境。JVM可以解读指令代码并与底层进行交互：包括操作系统平台和执行指令并管理资源的硬件体系结构。

Java 虚拟机（JVM）是运行 Java 字节码的虚拟机。JVM 有针对不同系统的特定实现（Windows，Linux，macOS），目的是使用相同的字节码，它们都会给出相同的结果。字节码和不同系统的 JVM 实现是 Java 语言“一次编译，随处可以运行”的关键所在。**JVM 并不是只有一种！只要满足 JVM 规范，每个公司、组织或者个人都可以开发自己的专属 JVM。**

### JVM的内存模型

JVM 运行时内存共分为虚拟机栈、堆、元空间、程序计数器、本地方法栈五个部分。还有一部分内存叫直接内存，属于操作系统的本地内存，也是可以直接操作的。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207091520404.png" alt="image-20220709152016354" style="width:40%;" />

- 元空间（方法区）：本质和永久代类似，都是对JVM规范中方法区的实现。元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。在jdk1.8以前，称为永久代，在1.8以后，称为元空间MateSpace，不由具名管理它的内存结构，而是交给操作系统内存，元空间使用的系统内存，元空间的串池StringTable被移到了堆内存中。线程共享，方法区逻辑上是堆的一部分，方法存储了更类结构相关的一些信息，比如常量，类变量，类的构造器，方法的信息，成员方法和构造方法，编译器编译后的代码等等，方法区如果内存不足也会报内存溢出
- **虚拟机栈**：每个线程都有一个私有的栈，随着线程的创建而创建。栈里面存着的是一种叫“栈帧”的东西，每个方法都会创建一个栈帧，栈帧中存放了`局部变量表（基本数据类型和对象引用）、操作数栈、方法出口`等信息。栈的大小可以固定也可以动态扩展。
  - `局部变量表`：用来临时存储8个基本数据类型、对象引用地址、returnAddress类型，就是一些操作完成以后的数据，是一个数组结构。
  - `操作数栈`：操作数栈就是用来操作的，例如代码中有个 i = 6*6，他在一开始的时候就会进行操作，读取我们的代码，进行计算后再放入局部变量表中去，临时来存放数据的。
  - `动态链接`：假如方法中有一个service.add()方法，要链接到别的地方去，这就是动态链接，存储链接的地方。
- **本地方法栈**：与虚拟机栈类似，区别是虚拟机栈执行Java方法，本地方法执行native方法。在虚拟机规范中对本地方法栈中方法使用的语言、使用方法与数据结构没有强制规定，因此虚拟机可以自由实现它。
- **程序计数器**：程序计数器可以看成是当前线程所执行的字节码的行号指示器。在任何一个确定的时刻，一个处理器（对于多内核来说是一个内核）都只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要一个独立的程序计数器，我们称这类内存区域为“线程私有”内存。
- **堆内存**：堆内存是 JVM 所有`线程共享的部分`，在虚拟机启动的时候就已经创建。`所有的对象和数组`都在堆上进行分配。这部分空间可通过`GC（垃圾回收器）`进行回收。当申请不到空间时会抛出 `OutOfMemoryError`。堆是JVM内存占用最大，管理最复杂的一个区域。其唯一的用途就是存放对象实例：所有的对象实例及数组都在对上进行分配。jdk1.8后，字符串常量池从永久代中剥离出来，存放在队中。
- **直接内存**：直接内存并不是虚拟机运行时数据区的一部分，也不是Java 虚拟机规范中农定义的内存区域。在JDK1.4 中新加入了`NIO(New Input/Output)`类，引入了一种基于`通道(Channel)与缓冲区（Buffer）的I/O `方式，它可以使用native 函数库直接分配堆外内存，然后通脱一个存储在Java堆中的DirectByteBuffer 对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在Java堆和Native堆中来回复制数据。

### 垃圾回收

#### 1、如何判断垃圾可以回收？

- 引用计数：给对象一个计数器，但是难以解决对象之间循环引用的问题，会造成内存泄露。
- 可达性分析：java虚拟机中的垃圾回收器采用的是这种算法，判断GC ROOT是否有相连的引用链，如果没有就回收。

##Q&A

### 未来JDK的新技术发展

#### 新一代即时编译器

Graal编译器比C2编译器晚了足足二十年面世，有着极其充沛的后发优势，在保持输出相近质量的编译代码的同时，开发效率和扩展性上都要显著优于C2编译器，这决定了C2编译器中优秀的代码优化技术可以轻易地移植到Graal编译器上，但是反过来Graal编译器中行之有效的优化在C2编译器里实现起来则异常艰难。这种情况下，Graal的编译效果短短几年间迅速追平了C2，甚至某些测试项中开始逐渐反超C2编译器。Graal能够做比C2更加复杂的优化，如“部分逃逸分析”（Partial Escape Analysis），也拥有比C2更容易使用激进预测性优化（Aggressive Speculative Optimization）的策略，支持自定义的预测性假设等。 

#### 向Native迈进

经陆续推出了跨进程的、可以面向用户程序的类型信息共享（Application Class Data Sharing，AppCDS，允许把加载解析后的类型信息缓存起来，从而提升下次启动速度，原本CDS只支持Java标准库，在JDK 10时的AppCDS开始支持用户的程序代码）、无操作的垃圾收集器（Epsilon，只做内存分配而不做回收的收集器，对于运行完就退出的应用十分合适）等改善措施。而酝酿中的一个更彻底的解决方案，是逐步开始对提前编译（Ahead of Time Compilation，AOT）提供支持。 

提前编译是相对于即时编译的概念，提前编译能带来的最大好处是Java虚拟机加载这些已经预编译成二进制库之后就能够直接调用，而无须再等待即时编译器在运行时将其编译成二进制机器码。理论上，提前编译可以减少即时编译带来的预热时间，减少Java应用长期给人带来的“第一次运行慢”的不良体验，可以放心地进行很多全程序的分析行为，可以使用时间压力更大的优化措施。

提前编译的缺点：降低了Java链接过程的动态性，必须要求加载的代码在编译期是全部已知的，而不能在运行期才确定。

### JVM种类，目前使用的哪种，特点？

**种类**：Sun Classic/Exact VM、Exact VM、HotSpot VM、Mobile/Embedded VM 、BEA JRockit/IBM J9 VM 、BEA Liquid VM/Azul VM、



### 为什么叫java虚拟机，与VMware的区别



### java虚拟机的整体架构

### 字节码的加载流程？

### java的编译器输入的指令流是一种基于栈的指令集架构，有什么优点？

### 能运行在虚拟机上的字节码只能由javac编译而来的java源代码产生吗，其他语言？





## 类加载相关
\1. jvm在什么情况下会加载一个类?
\2. 类加载到jvm中的过程?每个阶段的工作?
\3. jvm中的类加载器的类型及它加载的目标路径?如何自定义一个类加载器加载一个指定目录下的class文件?
\4. 什么是双亲委派模型，有什么作用?
\5. 类加载器是如何确定一个类在jvm中的唯一性的? 两个类来源于同一个Class文件，被同一个虚拟机加载,这两个类一定相等吗?
\6.  tomcat的类加载器有哪些?
\8. 双亲委派模型最大问题：底层的类加载器无法加载底层的类, 比如如下情况:
     javax.xml.parsers包中定义了xml解析的类接口, Service Provider Interface SPI 位于rt.jar 
  即接口在启动ClassLoader中,  而SPI的实现类，通常是由用户实现的， 由AppLoader加载。 

   以下是javax.xmlparsers.FactoryFinder中的解决代码:  
   static private Class getProviderClass(String className, ClassLoader cl,
    boolean doFallback, boolean useBSClsLoader) throws ClassNotFoundException
{
  try {
    if (cl == null) {
      if (useBSClsLoader) {
        return Class.forName(className, true, FactoryFinder.class.getClassLoader());
      } else {
        cl = ss.getContextClassLoader();      //获取上下文加载器
        if (cl == null) {
          throw new ClassNotFoundException();
        }
        else {
          return cl.loadClass(className);   //使用上下文ClassLoader
        }
      }
    }
    else {
      return cl.loadClass(className);
    }
  }
  catch (ClassNotFoundException e1) {
    if (doFallback) {
      // Use current class loader - should always be bootstrap CL
      return Class.forName(className, true, FactoryFinder.class.getClassLoader());
    }

   



  更多可以参考理解:  jdbc的SPI 加载方式.  ![img](file:///C:\Users\zhulin\AppData\Roaming\Tencent\QQTempSys\%W@GJ$ACOF(TYDYECOKVDYB.png)https://blog.csdn.net/syh121/article/details/120274044

  ClassLoader cl = Thread.currentThread().getContextClassLoader();
  return ServiceLoader.load(service, cl);

\9. 双亲委派模式是默认的模式，但并非必须. 还有以下几个例 子，它实际上是破坏了双亲委派模式的. 
  a. Tomcat的WebappClassLoader 就会先加载自己的Class，找不到再委托parent
  b. OSGi的ClassLoader形成网状结构，根据需要自由加载Class

\10. 请完成一个热替换的例子，并解释什么是热替换?