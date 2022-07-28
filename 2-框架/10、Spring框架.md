# Spring框架

## Spring的常用注解

### 1、给容器中注入插件

- @Componet：反之各种组件；@Controller（控制层）、@Service（业务层）、@Repository（数据访问层）都可以称为@Componet。
- @Bean：导入第三方包里面的注解
- @Import（要导入到容器中的组件）
- @ImportSelector：返回需要导入的组件的全类名数组
- @ImportBeanDefinitionRegistrar：手动注册bean到容器中
- 使用spring提供的FactoryBean（工厂Bean）

### 2、注入bean的注解

- @Autowired：由bean提供。可以作用在变量、setter方法、构造函数上；有个属性为required，可以配置为false。
- @Inject：由JSR-330提供；用法与@Autowired一样
- @Resource：由JSR-250提供
- @Primary：让spring进行自动装配的时候，默认使用首选的bean，和@Qualifier一个效果

@Autowired、@Inject是默认按照类型匹配的，@Resource是按照名称匹配的，@Autowired如果需要按照名称匹配需要和@Qualifier一起使用，@Inject和@Name一起使用。

### 3、JsonIgnore

1. 作用：在json序列化时将java bean中的一些属性忽略掉，序列化和反序列化都受到影响。
2. 使用方法：一般标记在属性或方法上，返回的json数据即不包含该属性
3. 注解失效：如果注解失效，可能是使用的是fastJson，尝试使用对应的注解来忽略字段，注解为：@JSONField(serialize = false)。

### 4、初始化方法和销毁方法

1. 通过@Bean(initMethod="init",destoryMethod="destory")方法

2. 通过bean实现InitializingBean来定义初始化逻辑，DisposableBean定义销毁逻辑

3. 可以使用JSR250：@PostConstruct：初始化方法；@PreDestory：销毁方法。

4. BeanPostProcessor：bean的后置处理器，在bean初始化前后进行一些处理工作

5. postProcessBeforeInitialization：在初始化之前工作；

6. postProcessAfterInitialization：在初始化工作之后工作；

### 5、Java配置类相关注解

- @Configuration：声明当前类为配置类。
- @Bean：注解在方法上，声明当前方法的返回值为一个bean，替代xml中的方式。
- @ComponentScan：用于对Component进行扫描。

### 6、切面（AOP）相关注解

- @Aspect：声明一个切面。
- @After：在方法执行之后执行（方法上）
- @Before：在方法执行之前执行（方法上）
- @Around：在方法执行前与之后执行（方法上）
- @PointCut：声明切点

### 7、@Bean的属性支持

- @Scope：设置Spring容器如何新建Bean实例（方法上，得有@Bean）
  - Singleton：单例，一个Spring容器中只有一个bean实例，默认模式。
  - Protetype：每次调用新建一个bean。
  - Request：web项目中，给每个http request新建一个bean。
  - Session：web项目中，给每个http session新建一个bean。
  - GlobalSession：给每一个 global http session新建一个Bean实例。

### 8、@Value注解

**支持如下方式的注入**

- 注入普通字符
- 注入操作系统属性
- 注入表达式结果
- 注入其他bean属性
- 注入文件资源
- 注入网站资源
- 注入配置文件

**@Value三种情况的用法**

1. `${}`是去找外部配置的参数，将值赋过来。
2. `#{}`是SpEL表达式，去寻找对应变量的内容。
3. `#{}`直接写字符串就是将字符串的值注入进去。