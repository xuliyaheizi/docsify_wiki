# 面试题Q&A

> jvm在什么情况下会加载一个类?

> 类加载到jvm中的过程?每个阶段的工作?

> jvm中的类加载器的类型及它加载的目标路径?如何自定义一个类加载器加载一个指定目录下的class文件?

**启动类/引导类（BootStrap ClassLoader）**

这个类加载器使用C/C++语言实现，嵌套在JVM内部，java程序无法直接操作这个类，是用来加载Java核心类库。如：`JAVA_HOME/jre/lib/rt.jar`、`resources.jar`、`sun.boot.class.path`路径下的包，用于提供jvm运行所需的包。没有继承`java.lang.ClassLoader`，也没有父类加载器。它加载`扩展类加载器`和`应用程序类加载器`，并成为他们的父类加载器。出于安全考虑，启动类只加载包名为：java、javax、sun开头的类。

**扩展类加载器（Extension ClassLoader）**

由`sun.misc.Launcher$ExtClassLoader`实现，派生继承自java.lang.ClassLoader，父类加载器为`启动类加载器`。加载从`java.ext.dirs`目录中加载类库，或者从JDK安装目录下`jre/lib/ext`加载类库。将自定义的包放在以上目录下，也会自动加载进去。

**应用程序类加载器（Application ClassLoader）**

由`sun.misc.Launcher$AppClassLoader`实现。派生继承自java.lang.ClassLoader，父类加载器为`启动类加载器`。负责加载`环境变量classpath`和`系统属性java.class.path`指定路径下的类库。是程序中默认的类加载器，Java程序中的类都由该加载器加载完成。通过`ClassLoader#getSystemClassLoader()`获取并操作这个加载器。

**自定义类加载器**

实现步骤：继承`java.lang.ClassLoader`类，重写`findClass()`方法，如果没有太复杂的需求，可以直接继承`URLClassLoader`类，重写`loadClass`方法(打破双亲委派机制)，具体可参考`AppClassLoader`和`ExtClassLoader`。

```java
public class TestClassLoader extends ClassLoader {

    private String codePath;

    public TestClassLoader(ClassLoader parent, String codePath) {
        super(parent);
        this.codePath = codePath;
    }

    public TestClassLoader(String codePath) {
        this.codePath = codePath;
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        BufferedInputStream bufferedInputStream = null;
        ByteArrayOutputStream byteArrayOutputStream = null;

        //完整的类名
        String file = codePath + name + ".class";
        try {

            //初始化输入流
            bufferedInputStream = new BufferedInputStream(new FileInputStream(file));
            //获取输出流
            byteArrayOutputStream = new ByteArrayOutputStream();

            int len;
            byte[] data = new byte[1024];
            while ((len = bufferedInputStream.read(data)) != -1) {
                byteArrayOutputStream.write(data, 0, len);
            }

            //获取内存中的字节数组
            byte[] bytes = byteArrayOutputStream.toByteArray();

            //调用defineClass将字节数组转换成class实例
            Class<?> clazz = defineClass(null, bytes, 0, bytes.length);
            return clazz;
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                bufferedInputStream.close();
                byteArrayOutputStream.close();
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }
        return null;
    }
}
```

```java
public class MyClassTest {
    public static void main(String[] args) {
        TestClassLoader testClassLoader = new TestClassLoader("D:/IDEA工作区/JVM/test1/src/main/java/com/zhulin" +
                "/myclass/");
        try {
            Class<?> demoTest = testClassLoader.loadClass("DemoTest");
            System.out.println(demoTest.getClassLoader().getClass().getName());
            Object o = demoTest.newInstance();
            Method say = demoTest.getMethod("say");
            say.invoke(o);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}

//输出
com.zhulin.myclass.TestClassLoader
hello world
```

>  什么是双亲委派模型，有什么作用?

> 类加载器是如何确定一个类在jvm中的唯一性的?  两个类来源于同一个Class文件，被同一个虚拟机加载,这两个类一定相等吗?
>
> 对于任意一个类，都需要由加载它的类加载器和这个类本身一同确立其在Java虚拟机中的唯一性，每一个类加载器，都拥有一个独立的类名称空间。这句话可以表达得更通俗一些：比较两个类是否“相等”，只有在这两个类是由同一个类加载器加载的前提下才有意义，否则，即使这两个类来源于同一个Class文件，被同一个虚拟机加载，只要`加载它们的类加载器`不同，那这两个类就必定不相等。

> tomcat的类加载器有哪些?

Tomcat的类加载器在Java的类加载器上 新增了5个类加载器.  3个基础类,每个web应用类加载器,+jsp加载器

commonClassLoader        CatalinaClassLoader     shareClassLoader      webappClassLoader     JspClassLoader
<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207121905218.png" alt="QQ图片20220712152131" style="width:40%;" />

- **Common**：以应用类加载器为父类，是tomcat顶层的公用类加载器，其路径由conf/catalina.properties中的common.loader指定，默认指向${catalina.home}/lib下的包。
- **Catalina**：以Common类加载器为父类，是用于加载Tomcat应用服务器的类加载器，其路径由server.loader指定，默认为空，此时tomcat使用Common类加载器加载应用服务器。
- **Shared**：以Common类加载器为父类，是所有Web应用的父类加载器，其路径由shared.loader指定，默认为空，此时tomcat使用Common类加载器作为Web应用的父加载器。
- **Web应用**：以Shared类加载器为父类，加载/WEB-INF/classes目录下的未压缩的Class和资源文件以及/WEB-INF/lib目录下的jar包，该类加载器只对当前Web应用可见，对其他Web应用均不可见。

> 双亲委派模型最大问题：底层的类加载器无法加载底层的类, 比如如下情况:
> javax.xml.parsers包中定义了xml解析的类接口,  Service Provider Interface SPI 位于rt.jar 
> 即接口在启动ClassLoader中,    而SPI的实现类，通常是由用户实现的， 由AppLoader加载。 

  以下是javax.xmlparsers.FactoryFinder中的解决代码:   

```java
static private Class getProviderClass(String className, ClassLoader cl,
                                      boolean doFallback, boolean useBSClsLoader) throws ClassNotFoundException{
    try {
        if (cl == null) {
            if (useBSClsLoader) {
                return Class.forName(className, true, FactoryFinder.class.getClassLoader());
            } else {
                cl = ss.getContextClassLoader();           //获取上下文加载器
                if (cl == null) {
                    throw new ClassNotFoundException();
                }
                else {
                    return cl.loadClass(className);      //使用上下文ClassLoader
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
```

   更多可以参考理解:   jdbc的SPI 加载方式.    https://blog.csdn.net/syh121/article/details/120274044

    ClassLoader cl = Thread.currentThread().getContextClassLoader();
    return ServiceLoader.load(service, cl);

> 双亲委派模式是默认的模式，但并非必须. 还有以下几个例 子，它实际上是破坏了双亲委派模式的. 
> a. Tomcat的WebappClassLoader 就会先加载自己的Class，找不到再委托parent
> b. OSGi的ClassLoader形成网状结构，根据需要自由加载Class

>  请完成一个热替换的例子，并解释什么是热替换?

热替换：在不停止正在运行的系统的情况下进行类（对象）的升级替换，

> 从虚拟机层面来看，以下代码运行时
>  number值的变化历程?????  0->10->20

```java
public class ClassInitTest {
    private static int number = 10;      //linking之prepare: number = 0 --> initial: 10 --> 20
    static {
        number = 20;
        System.out.println(num);
    }

    public static void main(String[] args) {
        System.out.println(ClassInitTest.num);//2
        System.out.println(ClassInitTest.number);//10
    }
}
```

> 以下代码为输出类加载器的名字, 请给出你的结果:


```java
public class ClassLoaderTest {
    public static void main(String[] args) {
        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
        System.out.println(systemClassLoader);//sun.misc.Launcher$AppClassLoader@18b4aac2
        ClassLoader extClassLoader = systemClassLoader.getParent();
        System.out.println(extClassLoader);//sun.misc.Launcher$ExtClassLoader@1540e19d
        ClassLoader bootstrapClassLoader = extClassLoader.getParent();
        System.out.println(bootstrapClassLoader);//null
        ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
        System.out.println(classLoader);//sun.misc.Launcher$AppClassLoader@18b4aac2
        ClassLoader classLoader1 = String.class.getClassLoader();
        System.out.println(classLoader1);//null
    }
}
```

> 给出以下代码输出的各个类加载器的路径( 注意：分别用idea测试和 直接在控制台下输出  测试两次，查看结果的不同，给出结论)

```java
public class ClassLoaderTest1 {
    public static void main(String[] args) {
        System.out.println("**********启动类加载器**************");
        //获取BootstrapClassLoader能够加载的api的路径
        URL[] urLs = sun.misc.Launcher.getBootstrapClassPath().getURLs();
        for (URL element : urLs) {
            System.out.println(element.toExternalForm());
        }
        //从上面的路径中随意选择一个类,来看看他的类加载器是什么:引导类加载器
        ClassLoader classLoader = Provider.class.getClassLoader();
        System.out.println(classLoader);//null

        System.out.println("***********扩展类加载器*************");
        String extDirs = System.getProperty("java.ext.dirs");
        for (String path : extDirs.split(";")) {
            System.out.println(path);
        }

        //从上面的路径中随意选择一个类,来看看他的类加载器是什么:扩展类加载器
        ClassLoader classLoader1 = CurveDB.class.getClassLoader();
        System.out.println(classLoader1);//sun.misc.Launcher$ExtClassLoader@1540e19d
    }
}
```

```java
//idea
**********启动类加载器**************
file:/D:/Java/jdk1.8/jre/lib/resources.jar
file:/D:/Java/jdk1.8/jre/lib/rt.jar
file:/D:/Java/jdk1.8/jre/lib/sunrsasign.jar
file:/D:/Java/jdk1.8/jre/lib/jsse.jar
file:/D:/Java/jdk1.8/jre/lib/jce.jar
file:/D:/Java/jdk1.8/jre/lib/charsets.jar
file:/D:/Java/jdk1.8/jre/lib/jfr.jar
file:/D:/Java/jdk1.8/jre/classes
null
***********扩展类加载器*************
D:\Java\jdk1.8\jre\lib\ext
C:\Windows\Sun\Java\lib\ext
null

Process finished with exit code 0

//控制台
**********start class loader**************
file:/D:/Java/jdk1.8/jre/lib/resources.jar
file:/D:/Java/jdk1.8/jre/lib/rt.jar
file:/D:/Java/jdk1.8/jre/lib/sunrsasign.jar
file:/D:/Java/jdk1.8/jre/lib/jsse.jar
file:/D:/Java/jdk1.8/jre/lib/jce.jar
file:/D:/Java/jdk1.8/jre/lib/charsets.jar
file:/D:/Java/jdk1.8/jre/lib/jfr.jar
file:/D:/Java/jdk1.8/jre/classes
null
***********Extend the class loader*************
D:\Java\jdk1.8\jre\lib\ext
C:\Windows\Sun\Java\lib\ext
null
```

> 获取ClassLoader 实例的途径
>
>   1. 获取当前类的 ClassLoader ：clazz.getClassLoader()
>   2. 获取当前线程上下文的ClassLoader：Thread.currentThread().getContextClassLoader()
>   3. 获取系统的ClassLoader ： ClassLoader.getSystemClassLoader();
>   4. 获取调用者的ClassLoader ： DriverManager.getCallerClassLoader()
>
>  那么以下代码输出的类加载器将是什么：

```java
public class ClassLoaderTest2 {
    public static void main(String[] args) {
        try {
            //1.Class.forName().getClassLoader()
            ClassLoader classLoader = Class.forName("java.lang.String").getClassLoader();
            System.out.println(classLoader); // String 类由启动类加载器加载，我们无法获取

            //2.Thread.currentThread().getContextClassLoader()
            ClassLoader classLoader1 = Thread.currentThread().getContextClassLoader();
            System.out.println(classLoader1);

            //3.ClassLoader.getSystemClassLoader().getParent()
            ClassLoader classLoader2 = ClassLoader.getSystemClassLoader();
            System.out.println(classLoader2);

        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}

//输出
null
sun.misc.Launcher$AppClassLoader@18b4aac2
sun.misc.Launcher$AppClassLoader@18b4aac2
```

> 测试一下双亲委派模型在保证系统模型统一方面的作用
>
> ​	保证JVM提供的核心类不被篡改，保证class执行安全。

```java
//创建一个类：   java.lang.String
package java.lang;

public class String {
    static{
        System.out.println("我是自定义的String类的静态代码块");
    }
}
public class StringTest {
    public static void main(String[] args) {
        java.lang.String str = new java.lang.String();
        System.out.println("hello");
        StringTest test = new StringTest();
        System.out.println(test.getClass().getClassLoader());
    }
}
```

以上代码的输出结果是什么?

```java
Error: A JNI error has occurred, please check your installation and try again
Exception in thread "main" java.lang.SecurityException: Prohibited package name: java.lang
	at java.lang.ClassLoader.preDefineClass(ClassLoader.java:655)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:754)
	at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:142)
	at java.net.URLClassLoader.defineClass(URLClassLoader.java:473)
	at java.net.URLClassLoader.access$100(URLClassLoader.java:74)
	at java.net.URLClassLoader$1.run(URLClassLoader.java:369)
	at java.net.URLClassLoader$1.run(URLClassLoader.java:363)
	at java.security.AccessController.doPrivileged(Native Method)
	at java.net.URLClassLoader.findClass(URLClassLoader.java:362)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:418)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:355)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:351)
	at sun.launcher.LauncherHelper.checkAndLoadMain(LauncherHelper.java:601)

Process finished with exit code 1
```

如果修改一下代码如下: 

```java
package java.lang;

public class String {
    static{
        System.out.println("我是自定义的String类的静态代码块");
    }
    public static void main(String[] args) {
        System.out.println("hello,String");
    }
}
```

 输出会是什么? 为什么?   这种机制就叫沙箱安全机制，请试着解释此术语及作用?

> 什么是沙箱？沙箱是一个限制程序运行的环境。沙箱机制就是将Java代码限定在虚拟机JVM)特定的运行范围中，并且严格限制代码对本地系统资源访问，通过这样的措施来保证对代码的有效隔离，防止对本地系统造成破坏。

```java
错误: 在类 java.lang.String 中找不到 main 方法, 请将 main 方法定义为:
   public static void main(String[] args)
否则 JavaFX 应用程序类必须扩展javafx.application.Application

Process finished with exit code 1
```

如果在自定义的java.lang包下定义自己的类，代码如下，会发生什么，请试分析为什么?

```java
package java.lang;

public class ShkStart {
    public static void main(String[] args) {
        System.out.println("hello!");
    }
}
```

```java
Error: A JNI error has occurred, please check your installation and try again
Exception in thread "main" java.lang.SecurityException: Prohibited package name: java.lang
	at java.lang.ClassLoader.preDefineClass(ClassLoader.java:655)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:754)
	at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:142)
	at java.net.URLClassLoader.defineClass(URLClassLoader.java:473)
	at java.net.URLClassLoader.access$100(URLClassLoader.java:74)
	at java.net.URLClassLoader$1.run(URLClassLoader.java:369)
	at java.net.URLClassLoader$1.run(URLClassLoader.java:363)
	at java.security.AccessController.doPrivileged(Native Method)
	at java.net.URLClassLoader.findClass(URLClassLoader.java:362)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:418)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:355)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:351)
	at sun.launcher.LauncherHelper.checkAndLoadMain(LauncherHelper.java:601)

Process finished with exit code 1
```

> 试分析基于SPI机制的  jdbc 驱动的加载流程.  着重分析，为什么jdbc的接口是通过Bootstrap ClassLoader加载  rt.jar包获取，但 jdbc驱动却无法通过Bootstrap classload加载，那么它是怎么加载进来的?

> jvm中只有主动使用类，才会加载类，那么加载类的七种情况有哪些?

只有当对类的主动使用的时候才会导致类的初始化，类的主动使用有以下：

- 创建类的实例，new。
- 访问某个类或接口的静态变量、或者对该静态变量赋值。
- 调用类的静态方法。
- 反射（如Class.forName("com.yz.Test")）。
- 初始化某个类的子类，则其父类也会被初始化。
- Java虚拟机启动时被标明为启动类的类（Java Test），直接使用java.exec命令来运行某个主类。
- jdk1.7

> 类加载器作为jvm运行的第一阶段的组件的总结???



> jvm运行时数据区的划分?

JVM运行时将数据区划分为程序计数器、虚拟机栈、本地方法栈、方法区和堆。

**方法区中永久代到元空间的转变：**在JDK6的时候开始放弃永久代，逐渐采用本地方法内存来实现方法区。在JDK7的时候，已经将永久代中的字符串常量池、静态变量等移出。JDK8完全废除永久代概念，改为在本地内存实现的元空间代替，并将JDK7中永久代剩余的内容全部移到元空间中。

> 根据jvm规范，这些数据区中哪些会出现 内存溢出异常，分别是什么场景下出现?

内存溢出分为两大类：`OutOfMemoryError`和`StackOverflowError`。类加载器加载所需的字节码文件完毕后交由执行引擎执行，在执行过程中需要一段空间来存储数据（类比CPU和主存），关于内存溢出我们需要关系这个段内存空间的分配与释放过程看是否会发生溢出。

- **java堆内存溢出**

  ```java
  public class JavaHeap {
      public static void main(String[] args) {
          List<byte[]> list = new ArrayList<>();
          int i = 0;
          while (true) {
              list.add(new byte[5 * 1024 * 1024]);
              System.out.println("count is:" + (++i));
          }
      }
  }
  
  //输出
  count is:627
  count is:628
  count is:629
  count is:630
  count is:631
  Exception in thread "main" java.lang.OutOfMemoryError: Java heap space //堆内存溢出
  	at com.zhulin.memory.JavaHeap.main(JavaHeap.java:16)
  
  Process finished with exit code 1
  ```

  **问题：**

  1. 设置的jvm内存太小，对象所需内存太大，创建对象时分配空间，就会抛出这个异常。
  2. 流量/数据峰值，应用程序自身的处理存在一定的限额，例如一定数量的用户或一定数量的数据。当用户量或数据量突然激增并超过预期的阈值时，就会峰值停止前正常运行的操作将停止并触发堆空间异常。

- **Java堆内存泄露**

  Java中的内存泄漏是一些对象不再被应用程序使用但垃圾收集无法识别的情况。因此，这些未使用的对象仍然在Java堆空间中无限期地存在。不停的堆积最终会出现溢出。

- **垃圾回收超时溢出**

  当应用程序耗尽所有可用内存时，GC开销限制超过了错误，而GC多次未能清除它，这时便会引发java.lang.OutOfMemoryError。当JVM花费大量的时间执行GC，而收效甚微，而一旦整个GC的过程超过限制便会触发错误(默认的jvm配置GC的时间超过98%，回收堆内存低于2%)。

  要减少对象生命周期，尽量能快速的进行垃圾回收。

- **Metaspace内存溢出**

  元空间的溢出，系统会抛出java.lang.OutOfMemoryError: Metaspace。出现这个异常的问题的原因是系统的代码非常多或引用的第三方包非常多或者通过动态代码生成类加载等方法，导致元空间的内存占用很大。

  默认情况下，元空间的大小仅受本地内存限制。但是为了整机的性能，尽量还是要对该项进行设置，以免造成整机的服务停机。

  1）优化参数配置，避免影响其他JVM进程

  -XX:MetaspaceSize，初始空间大小，达到该值就会触发垃圾收集进行类型卸载，同时GC会对该值进行调整：如果释放了大量的空间，就适当降低该值；如果释放了很少的空间，那么在不超过MaxMetaspaceSize时，适当提高该值。

  -XX:MaxMetaspaceSize，最大空间，默认是没有限制的。

  除了上面两个指定大小的选项以外，还有两个与 GC 相关的属性：
  -XX:MinMetaspaceFreeRatio，在GC之后，最小的Metaspace剩余空间容量的百分比，减少为分配空间所导致的垃圾收集 。
  -XX:MaxMetaspaceFreeRatio，在GC之后，最大的Metaspace剩余空间容量的百分比，减少为释放空间所导致的垃圾收集。

  2）慎重引用第三方包

  对第三方包，一定要慎重选择，不需要的包就去掉。这样既有助于提高编译打包的速度，也有助于提高远程部署的速度。

  3）关注动态生成类的框架

  对于使用大量动态生成类的框架，要做好压力测试，验证动态生成的类是否超出内存的需求会抛出异常。

- **直接内存内存溢出**

  在使用ByteBuffer中的allocateDirect()的时候会用到，很多javaNIO(像netty)的框架中被封装为其他的方法，出现该问题时会抛出java.lang.OutOfMemoryError: Direct buffer memory异常。

  如果经常有类似的操作，可以考虑设置参数：-XX:MaxDirectMemorySize，并及时clear内存。

- **栈内存溢出**

  当一个线程执行一个Java方法时，JVM将创建一个新的栈帧并且把它push到栈顶。此时新的栈帧就变成了当前栈帧，方法执行时，使用栈帧来存储参数、局部变量、中间指令以及其他数据。

  当一个方法递归调用自己时，新的方法所产生的数据(也可以理解为新的栈帧)将会被push到栈顶，方法每次调用自己时，会拷贝一份当前方法的数据并push到栈中。因此，递归的每层调用都需要创建一个新的栈帧。这样的结果是，栈中越来越多的内存将随着递归调用而被消耗，如果递归调用自己一百万次，那么将会产生一百万个栈帧。这样就会造成栈的内存溢出。

  ```java
  public class StackOverflow {
      private static int count;
  
      public static void main(String[] args) {
          try {
              method1();
          } catch (Throwable e) {
              e.printStackTrace();
              System.out.println(count);
          }
      }
  
      private static void method1() {
          count++;
          method1();
      }
  }
  //输出
  java.lang.StackOverflowError 	//栈内存溢出错误
  	at com.zhulin.memory.StackOverflow.method1(StackOverflow.java:22)
  	at com.zhulin.memory.StackOverflow.method1(StackOverflow.java:22)
  	at com.zhulin.memory.StackOverflow.method1(StackOverflow.java:22)
  .......
      at com.zhulin.memory.StackOverflow.method1(StackOverflow.java:22)
  	at com.zhulin.memory.StackOverflow.method1(StackOverflow.java:22)
  20958  	//调用了20958次导致了内存溢出
      
  //设置 -Xss256k
      at com.zhulin.memory.StackOverflow.method1(StackOverflow.java:22)
  	at com.zhulin.memory.StackOverflow.method1(StackOverflow.java:22)
  2721	//由于栈内存调小，只调用了2721次就导致内存溢出
  ```

  如果程序中确实有递归调用，出现栈溢出时，可以调高-Xss大小，就可以解决栈内存溢出的问题了。递归调用防止形成死循环，否则就会出现栈内存溢出。

- **创建本地线程内存溢出**

  线程基本只占用heap以外的内存区域，也就是这个错误说明除了heap以外的区域，无法为线程分配一块内存区域了，这个要么是内存本身就不够，要么heap的空间设置得太大了，导致了剩余的内存已经不多了，而由于线程本身要占用内存，所以就不够用了。

  首先检查操作系统是否有线程数的限制，使用shell也无法创建线程，如果是这个问题就需要调整系统的最大可支持的文件数。

  日常开发中尽量保证线程最大数的可控制的，不要随意使用线程池。不能无限制的增长下去。

- **超出交换区内存溢出**

  在Java应用程序启动过程中，可以通过-Xmx和其他类似的启动参数限制指定的所需的内存。而当JVM所请求的总内存大于可用物理内存的情况下，操作系统开始将内容从内存转换为硬盘。

  一般来说JVM会抛出Out of swap space错误，代表应用程序向JVM native heap请求分配内存失败并且native heap也即将耗尽时，错误消息中包含分配失败的大小（以字节为单位）和请求失败的原因。

  增加系统交换区的大小，我个人认为，如果使用了交换区，性能会大大降低，不建议采用这种方式，生产环境尽量避免最大内存超过系统的物理内存。其次，去掉系统交换区，只使用系统的内存，保证应用的性能。

- **数组超限内存溢出**

  有的时候会碰到这种内存溢出的描述Requested array size exceeds VM limit，一般来说java对应用程序所能分配数组最大大小是有限制的，只不过不同的平台限制有所不同，但通常在1到21亿个元素之间。当Requested array size exceeds VM limit错误出现时，意味着应用程序试图分配大于Java虚拟机可以支持的数组。JVM在为数组分配内存之前，会执行特定平台的检查：分配的数据结构是否在此平台是可寻址的。

  因此数组长度要在平台允许的长度范围之内。不过这个错误一般少见的，主要是由于Java数组的索引是int类型。 Java中的最大正整数为2 ^ 31 - 1 = 2,147,483,647。 并且平台特定的限制可以非常接近这个数字，例如：我的环境上(64位macOS，运行Jdk1.8)可以初始化数组的长度高达2,147,483,645（Integer.MAX_VALUE-2）。若是在将数组的长度再增加1达到nteger.MAX_VALUE-1会出现的OutOfMemoryError。

- **系统杀死进程内存溢出**

  在描述该问题之前，先熟悉一点操作系统的知识：操作系统是建立在进程的概念之上，这些进程在内核中作业，其中有一个非常特殊的进程，称为“内存杀手（Out of memory killer）”。当内核检测到系统内存不足时，OOM killer被激活，检查当前谁占用内存最多然后将该进程杀掉。

  一般Out of memory:Kill process or sacrifice child错会在当可用虚拟虚拟内存(包括交换空间)消耗到让整个操作系统面临风险时，会被触发。在这种情况下，OOM Killer会选择“流氓进程”并杀死它。

  虽然增加交换空间的方式可以缓解Java heap space异常，还是建议最好的方案就是升级系统内存，让java应用有足够的内存可用，就不会出现这种问题。

> 这些数据区哪些是线程独有的，哪些是线程共享区?

**线程私有：**程序计数器、虚拟机栈、本地方法栈

**线程共享：**方法区、堆、

> 每个区存储的数据的特点?

- 程序计数器：记录正在执行的虚拟机字节码指令的地址
- 虚拟机栈：每个线程都会创建一个栈，栈里存栈帧，栈帧中存储局部变量表、操作数栈、动态链接
- 本地方法栈：
- 方法区：存放已经被Java虚拟机加载的类型信息、常量、静态变量，即时编译器编译后的代码缓存等数据。运行时常量池存放各种字面量和符号引用。
- 堆：存放对象实例

> 程序计数器是什么，它是线程独有的吗? 它是否有内存溢出问题.

- 程序计数器可以看成是当前线程所执行的字节码的行号指示器。在任何一个确定的时刻，一个处理器（对于多内核来说是一个内核）都只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要一个独立的程序计数器，我们称这类内存区域为“线程私有”内存。

- 不存在内存溢出的问题，是`JVM数据区中唯一一个不会内存溢出的区`。

  程序计数器仅仅只是一个运行指示器，所需要存储的内容仅仅就是下一个需要待执行的命令的地址，无论代码有多少，在最坏情况下死循环也不会让这块内存区超限。只存下一个字节码指令的地址，消耗内存小且固定，无论方法多深，他只存一条。只针对一个线程，随着线程的结束而销毁。

> 虚拟机栈上保存哪些数据?怎么放?虚拟机栈是线程独有的吗，它是否有内存溢出问题?虚拟机栈的优点?

虚拟机栈是线程运行时需要的一个内存空间，每个线程都有一个私有的栈，随着线程的创建而创建。栈里面存着的是一种叫“栈帧”的东西，每个`方法`都会创建一个栈帧，栈帧中存放了`局部变量表（基本数据类型和对象引用）、操作数栈、方法出口`等信息。栈的大小可以固定也可以动态扩展。

- `局部变量表`：用来临时存储8个基本数据类型、对象引用地址、returnAddress类型，就是一些操作完成以后的数据，是一个数组结构。
- `操作数栈`：操作数栈就是用来操作的，例如代码中有个 i = 6*6，他在一开始的时候就会进行操作，读取我们的代码，进行计算后再放入局部变量表中去，临时来存放数据的。
- `动态链接`：假如方法中有一个service.add()方法，要链接到别的地方去，这就是动态链接，存储链接的地方。

虚拟机栈是线程独有的

- 存在内存溢出问题；栈内存溢出

  - 栈的大小是固定的，一直入栈没有出栈，最后导致栈帧的内存超过了整个栈的内存，使得无法分配新的栈帧内存，就会产生栈内存溢出。

    栈帧过多的原因：方法的递归调用，在方法的递归中没有设置一个正确的结束条件，以至于一直递归调用导致栈帧内存超过栈内存。

  - 栈帧过大导致栈内存溢出；栈内存大小一般为1M，但是方法内部的int类型的变量才4个字节，所以这种情况不常见

- 虚拟机栈的优点：跨平台、指令集小、编译器容易实现、对于栈来说不存在垃圾回收问题、栈是一种快速有效的分配存储方式，访问速度仅次于程序计数器

> 虚拟机栈的大小是否可动?是否会有异常出现?

- java虚拟机规范允许java栈的大小是动态的或者是固定不变的

  - 如果采用固定大小的Java虚拟机栈，那每一个线程的Java虚拟机栈容量可以在线程创建的时候独立选定。如果线程请求分配的栈容量超过Java虚拟机栈允许的最大容量，Java虚拟机将会抛出一个stackoverflowError异常。
  - 如果Java虚拟机栈可以动态扩展，并且在尝试扩展的时候无法中请到足够的内存，或者在创建新的线程时没有足够的内存去创建对应的虚拟机栈，那Java虚拟机将会抛出一个OutOfMemoryError异常。

  ```java
  public class StackOverflow {
      public static void main(String[] args) {
          main(args);
      }
  }
  //输出
  Exception in thread "main" java.lang.StackOverflowError		//栈内存溢出
  	at com.zhulin.memory.StackOverflow.main(StackOverflow.java:25)
  	at com.zhulin.memory.StackOverflow.main(StackOverflow.java:25)
  ```

> 如何设置虚拟机栈大小?

- 添加VM参数：-Xss256k

> 什么叫本地方法? 是否可以写一个例子来实现本地方法，以输出一个hello world?

- 本地方法是有非Java代码实现的Java方法。
- 在定义一个本地方法时，并不提供实现体，实现体是由非Java语言在外面实现的。

com_zhulin_nativeclass_MyNative

gcc  MyNative.c com_zhulin_nativeclass_MyNative.h jni.h -fPIC -shared -o mynative.dll

> 什么叫本地方法栈?有什么作用?它是线程私有的吗? 它是否有可能抛出异常?

- 虚拟机栈用于管理Java方法的调用，本地方法栈用于管理本地方法的调用
- 是线程私有的
- 会抛出异常，与虚拟机栈类似；允许被实现成固定或者是可动态扩展的内存大小。
  - 如果线程请求分配的栈容量超过本地方法栈允许的最大容量，Java虚拟机将会抛出一个StackOverflowError异常
  - 如果本地方法栈可以动态扩展，并且尝试扩展的时候无法申请到足够的内存，或者在创建新的线程时没有足够的内存去创建对应的本地方法栈，那么Java虚拟机将会抛出一个OutOfMemoryError异常。

> jvm规范一定强制要求实现本地方法栈吗?

- JVM规范并没有强制要求实现本地方法栈
- 在Hotspot JVM中，直接将本地方法栈和虚拟机栈合二为一

> 方法区是线程独有的吗?它是否有异常?它的作用?

- 方法区不是线程独有的
- 
- 方法的作用：当类加载器加载完成类之后，会将类信息、运行时常量池、静态变量（此处指的是指针，如果是一个对象对象的分配还是在堆中）等存储在方法区；但在JDK不同版本对字符串常量和静态变量的存储有所不同，这部分内容后续列出

> 方法区的演进, jdk7及以前，它叫什么? jdk8开始，这又叫什么.

- JDK6：在JDK6以前方法区也就是HotSpot虚拟机中的永久代，此时类信息、运行时常量池、静态变量等存储在方法区
- JDK7：在JDK7中法区也是HotSpot虚拟机中的永久代，此时类信息以及其他信息存储在永久代，但是静态变量以及字符串常量已经存储在堆中
- JDK8: 在这个版本中HotSpot中已经不存在永久代了，相对应的是MetaSpace也就是元数据空间，此时类信息以及其他信息存储在元数据空间，但是静态变量以及字符串常量已经存储在堆中； 并且此时元数据空间不再使用的虚拟机内存而是使用的直接内存

> 方法区或永久代的大小如何设置?

- JDK7以前：
  - -XX:PermSize, 默认20.75M
  - -XX：MaxPermSize， 32位系统默认64M， 64位默认82M
- jdk8以后
  - -XX：MetaspaceSize, 默认21M
  - -XX：MaxMetaspaceSize，默认-1没有限制