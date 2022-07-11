# JVM常见面试题

## 常见面试题一

>**jdk、jre、jvm的关系**

jdk是java开发工具包，包括了jre和jvm；jre是java运行时环境，为java程序运行的最小单元；jvm是java字节码运行的环境

>**未来jdk发展的新技术有哪些**

`Graal VM虚拟机`，号称`Run Programs Faster Anywhere`，是一款跨语言全栈虚拟机。如果有虚拟机能够取代`HotSpot`成为新的武林盟主，那么一定是`Graal VM`，并且虚拟机的切换对使用者是完全兼容的

`Substrate VM`，Substrate VM的出现，算是满足了人们心中对Java提前编译的全部期待，运行在Substrate VM上的小规模应用， 其内存占用和启动时间与运行在HotSpot上相比有**5倍到50倍**的下降

>**jvm的种类，我们常用的是哪一种，特点为**

不同的厂商，有不同的JVM的实现。我们采用的是`HotSpot`，名称中的HotSpot指的就是它的**热点代码探测技术**

- 通过计数器找到最具编译价值代码，触发即时编译或栈上替换
- 通过编译器与解释器协同工作，在最优化的程序响应时间与最佳执行性能中取得平衡

>**为什么叫java虚拟机，它和vmware的区别**

**虚拟机**

所谓虚拟机（Virtual Machine），就是一台虚拟的计算机。它是一款软件，用来执行一系列虚拟计算机指令。大体上，虚拟机可以分为系统虚拟机和程序虚拟机。

- 大名鼎鼎的Visual Box，VMware就属于系统虚拟机，它们完全是对物理计算机的仿真，提供了一个可运行完整操作系统的软件平台
- 程序虚拟机的典型代表就是Java虚拟机，它专门为执行单个计算机程序而设计，在Java虚拟机中执行的指令我们称为Java字节码指令

无论是系统虚拟机还是程序虚拟机，在上面运行的软件都被限制于虚拟机提供的资源中。

**Java虚拟机**

Java虚拟机是一台执行Java字节码的虚拟计算机，它拥有独立的运行机制，其运行的Java字节码也未必由Java语言编译而成。

JVM平台的各种语言可以共享Java虚拟机带来的跨平台性、优秀的垃圾回器，以及可靠的即时编译器。

Java技术的核心就是Java虚拟机（JVM，Java Virtual Machine），因为所有的Java程序都运行在Java虚拟机内部。

Java虚拟机就是二进制字节码的运行环境，负责装载字节码到其内部，解释/编译为对应平台上的机器指令执行。每一条Java指令，Java虚拟机规范中都有详细定义，如怎么取操作数，怎么处理操作数，处理结果放在哪里。

特点：

- 一次编译，到处运行
- 自动内存管理
- 自动垃圾回收功能

>**java虚拟机的整体架构**

主要包括三个部分：

- 类加载子系统
- JVM内存结构
- 执行引擎

![image-20220531143854437](https://cdn.fengxianhub.top/resources-master/202205311438599.png)

>**字节码的加载流程**

加载、链接、初始化

>**java的编译器输入的指令流是一种基于栈的指令集架构，它有什么优点**

Java编译器输入的指令流基本上是一种基于栈的指令集架构，另外一种指令集架构则是基于寄存器的指令集架构。具体来说：这两种架构之间的区别：

**基于栈式架构的特点**

- 设计和实现更简单，适用于资源受限的系统；
- 避开了寄存器的分配难题：使用零地址指令方式分配。
- 指令流中的指令大部分是零地址指令，其执行过程依赖于操作栈。指令集更小，编译器容易实现。
- 不需要硬件支持，可移植性更好，更好实现跨平台

**基于寄存器架构的特点**

- 典型的应用是x86的二进制指令集：比如传统的PC以及Android的Davlik虚拟机。
- 指令集架构则完全依赖硬件，可移植性差
- 性能优秀和执行更高效
- 花费更少的指令去完成一项操作。
- 在大部分情况下，基于寄存器架构的指令集往往都以一地址指令、二地址指令和三地址指令为主，而基于栈式架构的指令集却是以零地址指令为主方水洋

**栈的特点**：

- 跨平台性
- 指令集小
- 指令多
- 执行性能比寄存器差

>**jvm可以编译其他语言的字节码吗**

可以，凡是能够编译成字节码文件的语言都可以使用JVM

>**java虚拟机规范有哪些？**

**java编译器方面**



>**jvm在什么情况下会加载一个类?**

在JVM中所有的类都是懒加载的

>**类加载到jvm中的过程 ? 每个阶段的工作 ?**



>**jvm中的类加载器的类型及它加载的目标路径?如何自定义一个类加载器加载一个指定目录下的class文件?**



>**什么是双亲委派模型，有什么作用 ?**



>**类加载器是如何确定一个类在jvm中的唯一性的? 两个类来源于同一个Class文件，被同一个虚拟机加载,这两个类一定相等吗?**

通过魔术字来确定是不是class文件。被同一个JVM加载两个类不一定相等，在JVM规范中是这样定义的，两个类及其类加载器相同才能说这两个类相同

```java
ClassFile { // u4表示字节数
    u4 magic; // 魔术字
    u2 minor_version; // 小版本号
    u2 major_version; // 主版本号
    u2 constant_pool_count; // 常量池信息
    cp_info constant_pool[constant_pool_count-1];
    u2 access_flags; // 类的访问权限
    u2 this_class; // 类名信息
    u2 super_class; // 父类信息
    u2 interfaces_count; // 接口数量
    u2 interfaces[interfaces_count]; // 接口信息
    u2 fields_count; // 属性数量
    field_info fields[fields_count];  // 属性信息
    u2 methods_count; // 方法数量
    method_info methods[methods_count]; // 方法信息
    u2 attributes_count; // 属性信息
    attribute_info attributes[attributes_count];
}
```

0~3 字节，表示它是否是【class】类型的文件

0000000 ***ca fe ba be*** 00 00 00 34 00 23 0a 00 06 00 15 09

>**Tomcat的类加载器有哪些?**

Tomcat的[类加载](https://so.csdn.net/so/search?q=类加载&spm=1001.2101.3001.7020)器在Java的类加载器上 新增了5个类加载器，3个基础类，每个web应用类加载器+jsp加载器

- commonClassLoader    
- CatalinaClassLoader   
- shareClassLoader   
- webappClassLoader   
- JspClassLoader

>**Tomcat为什么打破了双亲委派机制**

1. 部署在用一个Tomcat上的两个web应用所使用的类库要相互隔离

2. 部署在同一个Tomcat上的两个web应用所使用的java类库要相互共享

3. 保证Tomcat服务器自身的安全不受部署的web应用程序影响

4. 需要支持jsp页面的热部署和热加载

>**双亲委派模型最大问题是什么**

底层的类加载器无法加载底层的类, 比如如下情况:
`javax.xml.parsers`包中定义了xml解析的类接口, Service Provider Interface SPI 位于rt.jar 
即接口在启动ClassLoader中,  而SPI的实现类，通常是由用户实现的， 由AppLoader加载。 

以下是`javax.xmlparsers.FactoryFinder`中的解决代码:  

   ```java
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
      // https://blog.csdn.net/syh121/article/details/120274044
  ClassLoader cl = Thread.currentThread().getContextClassLoader();
  return ServiceLoader.load(service, cl);
   ```

>双亲委派模式是默认的模式，但并非必须. 还有以下几个例 子，它实际上是破坏了双亲委派模式的. 
>
>a. Tomcat的WebappClassLoader 就会先加载自己的Class，找不到再委托parent
>b. OSGi的ClassLoader形成网状结构，根据需要自由加载Class



>请完成一个热替换的例子，并解释什么是热替换?





>`<clinit>()` 是类构造器方法,它与类的构造方法有什么区别?

对于一个类，编译器会按从上至下的顺序，收集所有 static 静态代码块和静态成员赋值的代码，合并为一个特殊的方法`<cinit>()V`

```java
public class Demo {
    static int i = 10;
    static {
        i = 20;
    }
    static {
        i = 30;
    }
}
```

```java
0: bipush 10
2: putstatic #2 // Field i:I
5: bipush 20
7: putstatic #2 // Field i:I
10: bipush 30
12: putstatic #2 // Field i:I
15: return
```

**编译器会按从上至下的顺序，收集所有 {} 代码块和成员变量赋值的代码，形成新的构造方法，但原始构造方法内的代码总是在最后！**











