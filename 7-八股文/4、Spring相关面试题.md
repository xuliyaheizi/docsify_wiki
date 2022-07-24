# Spring相关面试题

## 一、Spring

### 1.1、什么是Spring？

Spring 是个java企业级应用的开源开发框架。Spring主要用来开发Java应用，但是有些扩展是针对构建J2EE平台的web应用。Spring 框架目标是简化Java企业级应用开发，并通过POJO为基础的编程模型促进良好的编程习惯。

Spring是一个**轻量的IOC和AOP容器框架**。是为Java应用程序提供基础性服务的一套框架，目的用于简化企业应用程序的开发，它使得开发者只需要关心业务需求。常见的配置方式有三种：`基于XML的配置`、`基于注解的配置`、`基于Java的配置`。

主要模块有：

- Spring Core：核心类库，提供IOC服务；
- Spring Context：提供框架式的Bean访问方式，以及企业级功能（JNDI、定时任务等）；
- Spring AOP：AOP服务；
- Spring DAO：对JDBC的抽象，简化了数据访问异常的处理；
- Spring ORM：对现有的ORM框架的支持；
- Spring Web：提供了基本的面向Web的综合特性，例如多方文件上传；
- Spring MVC：提供面向Web应用的Model-View-Controller实现。

### 1.2、Spring框架的好处？

- 轻量：Spring是轻量的，基本的版本大约2MB。
- 控制反转：Spring通过控制反转实现了基本耦合，对象们给出它们的依赖，而不是创建或查找依赖的对象们。
- 面向切面的编程（AOP）：Spring支持面向切面的编程，并且把应用业务逻辑和系统服务分开。
- 容器：Spring包含并管理应用中对象的生命周期和配置。
- MVC框架：Spring的WEB框架是个精心设计的框架，是WEB框架的一个很好的替代品。
- 事务管理：Spring提供一个持续的事务管理接口，可用扩展到上至本地事务下至全局事务（JTA）。
- 异常处理：Spring提供方便的API把具体技术相关的异常（比如由JDBC，Hibernate or JDO抛出的）转化为一致的unchecked异常。

### 1.3、Autowired和Resource关键字的区别？

@Resource和@Autowired都是做bean的注入时使用，且@Resource并不是Spring的注解，它的包是javax.annotation.Resource，需要导入，但是Spring支持该注解的注入。

1. 共同点：两者都可以写在字段和setter上。两者如果都写在字段上，那么就不需要再写setter方法。

2. 不同点：

   - @Autowired为Spring提供的注解，需要导入包org.springframework.beans.factory.annotation.Autowired;只按照byType注入。

     ```java
     public class TestServiceImpl {
         // 下面两种@Autowired只要使用一种即可
         @Autowired
         private UserDao userDao; // 用于字段上
     
         @Autowired
         public void setUserDao(UserDao userDao) { // 用于属性的方法上
             this.userDao = userDao;
         }
     }
     ```

   - @Autowired注解是按照类型（byType）装配依赖对象，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它的required属性为false。如果我们想使用按照名称（byName）来装配，可以结合@Qualififier注解一起使用。如下：

     ```java
     public class TestServiceImpl {
         @Autowired
         @Qualifier("userDao")
         private UserDao userDao; 
     }
     ```

   - @Resource默认按照ByName自动注入，由J2EE提供，需要导入包javax.annotation.Resource。@Resource有两个重要的属性：name和type，而Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。所以，如果使用name属性，则使用byName的自动注入策略。而使用type属性时则使用byType自动注入策略。如果即不制定name也不制定type属性，这时将通过反射机制使用byName自动注入策略。

     ```java
     public class TestServiceImpl {
         // 下面两种@Resource只要使用一种即可
         @Resource(name="userDao")
         private UserDao userDao; // 用于字段上
     
         @Resource(name="userDao")
         public void setUserDao(UserDao userDao) { // 用于属性的setter方法上
             this.userDao = userDao;
         }
     }
     ```

     注：最好是将@Resource放在setter方法上，因为这样更符合面向对象的思想，通过set、get去操作属性，而不是直接去操作属性。

     @Resource装配顺序：

     ①如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常。

     ②如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常。

     ③如果指定了type，则从上下文中找到类似匹配的唯一bean进行装配，找不到或是找到多个，都会抛出异常。

     ④如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配。@Resource的作用相当于@Autowired，只不过@Autowired按照byType自动注入。

### 1.4、依赖注入的方式有几种？

1. 构造器注入：将被依赖对象通过构造函数的参数注入给依赖对象，并且在初始化对象的时候注入。

   优点：对象初始化完成后便可获得可使用的对象。

   缺点：当需要注入的对象很多时，构造器参数列表将会很长；不够灵活。若有多种注入方式，每种方式只需注入指定几个依赖，那么就需要提供多个重载的构造函数，有点麻烦。

2. setter方式注入：IOC Service Provider通过调用成员变量提供的setter函数将被依赖对象注入给依赖类。

   优点：灵活。可用选择性的注入需要的对象。

   缺点：依赖对象初始化完成后由于尚未注入被依赖对象，因此还不能使用。

3. 接口注入：依赖类必须要实现指定的接口，然后实现该接口中的一个函数，该函数就是用于依赖注入。该函数的参数就是要注入的对象。

   优点：接口注入中，接口的名字、函数的名字都不重要，只要保证函数的参数是要注入的对象类型即可。

   缺点：侵入行（如果类A要使用别人提供的一个功能，若为了使用这个功能，需要在自己的类中增加额外的代码）太强，不建议使用。

### 1.5、说说SpringMVC的理解？

**MVC是一种设计模式**

![image-20220722213930324](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207222139753.png)

- M-Model 模型（完成业务逻辑：有javaBean构成，service+dao+entity）
- V-View 视图（做界面的展示 jsp，html……）
- C-Controller 控制器（接收请求—>调用模型—>根据结果派发页面）

springMVC是一个MVC的开源框架，springMVC=struts2+spring，springMVC就相当于是Struts2加上sring的整合，但是这里有一个疑惑就是，springMVC和spring是什么样的关系呢？这个在百度百科上有一个很好的解释：意思是说，springMVC是spring的一个后续产品，其实就是spring在原有基础上，又提供了web应用的MVC模块，可以简单的把springMVC理解为是spring的一个模块（类似AOP，IOC这样的模块），网络上经常会说springMVC和spring无缝集成，其实springMVC就是spring的一个子模块，所以根本不需要同spring进行整合。

**工作原理**

![image-20220722214050977](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207222140229.png)

1. 用户发送请求至前端控制器DispatcherServlet。 

2. DispatcherServlet收到请求调用HandlerMapping处理器映射器。

3. 处理器映射器找到具体的处理器(可以根据xml配置、注解进行查找)，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。 

4. DispatcherServlet调用HandlerAdapter处理器适配器。

5. HandlerAdapter经过适配调用具体的处理器(Controller，也叫后端控制器)。 

6. Controller执行完成返回ModelAndView。 

7. HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet。 

8. DispatcherServlet将ModelAndView传给ViewReslover视图解析器。

9. ViewReslover解析后返回具体View。

10. DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）。

11. DispatcherServlet响应用户。

**组件说明：**

以下组件通常使用框架提供实现：

- DispatcherServlet：作为前端控制器，整个流程控制的中心，控制其它组件执行，统一调度，降低组件之间的耦合性，提高每个组件的扩展性。HandlerMapping：通过扩展处理器映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。
- HandlAdapter：通过扩展处理器适配器，支持更多类型的处理器。
- ViewResolver：通过扩展视图解析器，支持更多类型的视图解析，例如：jsp、freemarker、pdf、 excel等。

**组件：** 

1. 前端控制器DispatcherServlet（不需要工程师开发）,由框架提供 作用：接收请求，响应结果，相当于转发器，中央处理器。有了dispatcherServlet减少了其它组件之间的耦合度。 用户请求到达前端控制器，它就相当于mvc模式中的c，dispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请求，dispatcherServlet的存在降低了组件之间的耦合性。

2. 处理器映射器HandlerMapping(不需要工程师开发),由框架提供 作用：根据请求的url查找Handler HandlerMapping负责根据用户请求找到Handler即处理器，springmvc提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。

3. 处理器适配器HandlerAdapter 作用：按照特定规则（HandlerAdapter要求的规则）去执行Handler 通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。

4. 处理器Handler(需要工程师开发) 注意：编写Handler时按照HandlerAdapter的要求去做，这样适配器才可以去正确执行Handler Handler 是继DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的用户请求进行处理。 由于Handler涉及到具体的用户业务请求，所以一般情况需要工程师根据业务需求开发Handler。 

5. 视图解析器View resolver(不需要工程师开发),由框架提供 作用：进行视图解析，根据逻辑视图名解析成真正的视图（view） View Resolver负责将处理结果生成View视图，View Resolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。 springmvc框架提供了很多的View视图类型，包括：jstlView、freemarkerView、pdfView等。 一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由工程师根据业务需求开发具体的页面。

6. 视图View(需要工程师开发jsp...) View是一个接口，实现类支持不同的View类型（jsp、freemarker、pdf...）

**核心架构的具体流程步骤如下：** 

1. 首先用户发送请求——>DispatcherServlet，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器进行处理，作为统一访问点，进行全局的流程控制；

2.  DispatcherServlet——>HandlerMapping， HandlerMapping 将会把请求映射为HandlerExecutionChain 对象（包含一个Handler 处理器（页面控制器）对象、多个HandlerInterceptor 拦截器）对象，通过这种策略模式，很容易添加新的映射策略； 

3. DispatcherServlet——>HandlerAdapter，HandlerAdapter 将会把处理器包装为适配器，从而支持多种类型的处理器，即适配器设计模式的应用，从而很容易支持很多类型的处理器； 

4. HandlerAdapter——>处理器功能处理方法的调用，HandlerAdapter 将会根据适配的结果调用真正的处理器的功能处理方法，完成功能处理；并返回一个ModelAndView 对象（包含模型数据、逻辑视图名）； 

5. ModelAndView的逻辑视图名——> ViewResolver， ViewResolver 将把逻辑视图名解析为具体的View，通过这种策略模式，很容易更换其他视图技术； 6、View——>渲染，View会根据传进来的Model模型数据进行渲染，此处的Model实际是一个Map数据结构，因此很容易支持其他视图技术； 

6. 返回控制权给DispatcherServlet，由DispatcherServlet返回响应给用户，到此一个流程结束。

   

看到这些步骤我相信大家很感觉非常的乱，这是正常的，但是这里主要是要大家理解springMVC中的几个组件：

- 前端控制器（DispatcherServlet）：接收请求，响应结果，相当于电脑的CPU。
- 处理器映射器（HandlerMapping）：根据URL去查找处理器。
- 处理器（Handler）：需要程序员去写代码处理逻辑的。
- 处理器适配器（HandlerAdapter）：会把处理器包装成适配器，这样就可以支持多种类型的处理器，类比笔记本的适配器（适配器模式的应用）。
- 视图解析器（ViewResovler）：进行视图解析，多返回的字符串，进行处理，可以解析成对应的页面。

### 1.6、Spring常用的注解有哪些？

- @RequestMapping：用于处理请求url映射的注解，可用于类或方法上。用于类上，则表示类中的所有响应请求的方法都是以该地址作为父路径。
- @RequestBody：注解实现接收http请求的json数据，将json转换为java对象。
- @ResponseBody：注解实现将controller方法返回对象转化为json对象响应给客户。

### 1.7、说说对Spring的AOP的理解？

AOP（面向切面编程）能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，减低模块间的耦合度，并有利于未来的可扩展性和可维护性。

SpringAOP是基于动态代理的，如果代理的对象实现了某个接口，那么Spring AOP就会使用JDK动态代理去创建代理对象。而对于没有实现接口的对象，就无法使用JDK动态代理，转而使用CGlib动态代理生成一个被代理对象的子类作为代理。

![image-20220722215532314](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207222155107.png)

当然也可以使用AspectJ，Spring AOP中已经集成了AspectJ，AspectJ应该算得上是Java生态系统中最完整的AOP框架了。使用AOP之后我们可以把一些通用功能抽象出来，在需要用到的地方直接使用即可，这样可以大大简化代码量。我们需要增加新功能也方便，提高了系统的扩展性。日志功能、事务管理和权限管理等场景都用到了AOP。

### 1.8、在SpringAOP中，关注点和横切关注的区别是什么？

关注点是应用中一个模块的行为，一个关注点可能会被定义成一个我们想实现的一个功能。横切关注点是一个关注点，此关注点是整个应用都会使用的功能，并影响整个应用，比如日志、安全和数据传输，几乎应用的每个模块都需要的功能。因此这些都属于横切关注点。

那什么是连接点呢？连接点代表一个应用程序的某个位置，在这个位置我们可以插入一个AOP切面，它实际上是个应用程序执行Spring AOP的位置。切入点是什么？切入点是一个或一组连接点，通知将在这些位置执行。可以通过表达式或匹配的方式指明切入点。

### 1.9、什么是通知呢？有哪些类型呢？

通知是个在方法执行前或执行后要做的动作，实际上是程序执行时要通过SpringAOP框架触发的代码段。

Spring切面可以应用五种类型的通知：

- **before**：前置通知，在一个方法执行前被调用。
- **after:** 在方法执行之后调用的通知，无论方法执行是否成功。
- **after-returning:** 仅当方法成功完成后执行的通知。
- **after-throwing:** 在方法抛出异常退出时执行的通知。
- **around:** 在方法执行之前和之后调用的通知。

### 1.10、说说对SpringIOC的理解？

IOC就是控制反转，是指创建对象的控制权的转移。以前创建对象的主动权和时机是由自己把控的，而现在这种权力转移到Spring容器中，并由容器根据配置文件去创建实例和管理各个实例之间的依赖关系。对象与对象之间松散耦合，也利于功能的复用。DI依赖注入，和控制反转是同一个概率的不同角度的描述，即应用程序在运行时依赖IOC容器来动态注入对象需要的外部资源。

最直观的表达就是，IOC让对象的创建不用去new了，可以由spring自动生产，使用java的反射机制，根据配置文件在运行动态的去创建对象以及管理对象，并调用对象的方法的。

SpringIOC有三种注入方式：构造器注入、setter方式注入、根据注解注入。

> IOC让相互合作的组件保持松散的耦合，而AOP编程允许把遍布于各层的功能分离出来，形成可重用的功能组件。

### 1.11、解释一下Spring Bean的生命周期？

Servlet的生命周期：实例化、初始化init、接收请求service、销毁destroy；

Spring上下文的Bean生命周期也类似，如下：

1. 实例化Bean：对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean进行实例化。对于ApplicationContext容器，当容器启动结束后，通过获取BeanDefinition对象中的信息，实例化所有的bean。
2. 设置对象属性（依赖注入）：实例化后的对象被封装在BeanWrapper对象中，紧接着，Spring根据BeanDefinition中的信息以及通过BeanWrapper提供的设置属性的接口完成依赖注入。
3. 处理Aware接口：Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给Bean
   1. 如果这个Bean已经实现了BeanNameAware接口，会调用它实现的setBeanName（String beanld）方法，此处传递的就是Spring配置文件中Bean的id值。
   2. 如果这个Bean已经实现了BeanFactoryAware接口，会调用它实现的setBeanFactory()方法，传递的是Spring工厂自身。
   3. 如果这个Bean已经实现了ApplicationContextAware接口，会调用setApplicationContext（ApplicationContext）方法，传入Spring上下文。
4. BeanPostProcessor：如果想对Bean进行一些自定义的处理，那么可以让Bean实现了BeanPostProcessor接口，那将会调用postProcessBeforeInitalization（Object obj，String s）方法。
5. InitializingBean与init-method：如果Bean在Spring配置文件中配置了init-method属性，则会自动调用其配置的初始化方法。
6. 如果这个Bean实现了BeanPostProcessor接口，将会调用postProcessAfterInitialization(Object obj, String s)方法；由于这个方法是在Bean初始化结束时调用的，所以可以被应用于内存或缓存技术；
7. DisposableBean：当Bean不再需要时，会经过清洗阶段，如果Bean实现了DisposableBean这个接口，会调用其实现的destroy()方法。
8. destroy-method：最后，如果这个Bean的Spring配置中配置了destroy-method属性，会自动调用其配置的销毁方法。

![image-20220722222148405](https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/202207222221768.png)

### 1.12、解释Spring支持的几种Bean的作用域？

Spring容器中的bean可以分为5个范围

- singleton：默认，每个容器中只有一个bean的实例，单例的模式由BeanFactory自身来维护。
- prototype：为每一个bean请求提供一个实例。
- request：为每一个网络请求创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收。
- session：与request范围类似，确保每个session中有一个bean的实例，在session过期后，bean会随之失效。
- global-session：全局作用域，global-session和Porlet应用相关。当你的应用部署在Portlet容器中工作时，它包含很多portlet。如果你想要声明让所有的portlet共用全局的存储变量的话，那么这全局变量需要存储在global-session中。全局作用域与Servlet中的session作用域效果相同。

### 1.13、Spring基于XML注入bean的几种方式？

1. set方法注入
2. 构造器注入：通过index设置参数的位置；通过type设置参数类型
3. 静态工厂注入
4. 实例工厂

### 1.14、Spring框架中都用到了哪些设计模式？

- 简单工厂模式：Spring中的BeanFactory就是简单工厂模式的体现。根据传入一个唯一的标识来获得Bean对象，但是在传入参数后创建还是传入参数前创建，要根据具体情况来定。
- 工厂模式：Spring中的FactoryBean就是典型的工厂模式。实现了FactoryBean接口的bean是一类叫做factory的bean。其特点是，spring在使用getBean()调用获得该bean时，会自动调用该bean的getObject()方法，所有返回的不是factory这个bean，而bean.getObject()方法的返回值。
- 单例模式：在Spring中用到的单例模式有：`scope="singleton"`，注册式单例模式，bean存放于Map中，bean name当做key，bean当做value。
- 原型模式：在Spring中用到的原型模式有：`scope="prototype"`，每次获取的是通过克隆生成的新实例，对其进行修改时对原有实例对象不造成任何影响。
- 迭代器模式：在 Spring 中有个 CompositeIterator 实现了 Iterator，Iterable 接口和 Iterator 接口，这两个都是迭代相关的接口。可以这么认为，实现了 Iterable 接口，则表示某个对象是可被迭代的。Iterator 接口相当于是一个迭代器，实现了 Iterator 接口，等于具体定义了这个可被迭代的对象时如何进行迭代的。
- 代理模式：Spring中经典的AOP，就是使用动态代理实现的，分JDK和CGlib动态代理。
- 适配器模式：Spring 中的 AOP 中 AdvisorAdapter 类，它有三个实现：MethodBeforAdviceAdapter、AfterReturnningAdviceAdapter、ThrowsAdviceAdapter。Spring会根据不同的 AOP 配置来使用对应的 Advice，与策略模式不同的是，一个方法可以同时拥有多个Advice。Spring 存在很多以 Adapter 结尾的，大多数都是适配器模式。
- 观察者模式：Spring 中的 Event 和 Listener。spring 事件：ApplicationEvent，该抽象类继承了EventObject 类，JDK 建议所有的事件都应该继承自 EventObject。spring 事件监听器：ApplicationListener，该接口继承了 EventListener 接口，JDK 建议所有的事件监听器都应该继承EventListener。
- 责任链模式：DispatcherServlet 中的 doDispatch() 方法中获取与请求匹配的处理器HandlerExecutionChain，this.getHandler() 方法的处理使用到了责任链模式。

### 1.15、Spring框架中的单例Bean是线程安全的吗？

Spring框架并没有对单例Bean进行任何多线程的封装处理

- 关于单例Bean的线程安全和并发问题，需要开发者自行去搞定。
- 单例的线程安全问题，并不是Spring应该去关心的。Spring应该做的是，提供根据配置，创建单例Bean或多例Bean的功能。

实际上，大部分的Spring Bean并没有可变的状态，所以在某种程度上说Spring的单例Bean是线程安全的。如果Bean有多种状态的话，就需要自行保证线程。最浅显的解决办法，就是将多态Bean的作用域（Scope）由Singleton变更为Prototype。

## 二、SpringBoot



