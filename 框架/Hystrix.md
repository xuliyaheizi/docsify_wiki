# Hystrix断路器-服务降级

## 一.服务雪崩

多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其它的微服务，这就是所谓的“扇出”。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的“雪崩效应”.

对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的所有资源都在几秒钟内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障。这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统。

所以，通常当你发现一个模块下的某个实例失败后，这时候这个模块依然还会接收流量，然后这个有问题的模块还调用了其他的模块，这样就会发生级联故障，或者叫雪崩。

### 引起雪崩的原因和服务雪崩的三个阶段

**原因：**

- 硬件故障
- 程序Bug
- 缓存击穿(用户大量访问缓存中没有的键值，导致大量请求查询数据库，使数据库压力过大）
- 用户大量请求

**三阶段：**

1. 服务不可用
2. 调用端重试加大流量（用户重试/代码逻辑重试）
3. 服务调用者不可用（同步等待造成的资源耗尽）

### 解决方案

#### 应用扩容（扩大服务器承受力）

- 加机器
- 升级硬件

#### 流量控制

- 限流
- 关闭重试

#### 缓存

将用户可能访问的数据大量的放入缓存中，减少访问数据库的请求

#### 服务降级

- 页面接口拒绝服务
- 页面拒绝服务
- 延迟持久化
- 随机拒绝服务

#### 服务熔断

## 二.熔断器

熔断器的原理很简单，如同电力过载保护器。它可以实现快速失败，如果它在一段时间内侦测到许多类似的错误，会强迫其以后的多个调用快速失败，不再访问远程服务器，从而防止应用程序不断地尝试执行可能会失败的操作，使得应用程序继续执行而不用等待修正错误，或者浪费CPU时间去等到长时间的超时产生。熔断器也可以使应用程序能够诊断错误是否已经修正，如果已经修正，应用程序会再次尝试调用操作。

熔断器模式就像是那些容易导致错误的操作的一种代理。这种代理能够记录最近调用发生错误的次数，然后决定使用允许操作继续，或者立即返回错误。 熔断器开关相互转换的逻辑如下图：

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/1648986685788-5233a374-3dc0-4e94-8982-c5031d1ebf37.png" alt="image.png" style="zoom: 80%;" />

## 三.Hystrix

**Hystrix**是一个用于处理**分布式系统**的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等

**Hystrix**能够保证在一个依赖出问题的情况下，**不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性**。

“断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个符合预期的、可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的**蔓延，乃至雪崩**。

### 断路器机制

断路器很好理解, 当**Hystrix Command**请求后端服务失败数量**超过一定比例(默认50%)**, 断路器会切换到**开路状态(Open)**. 这时所有请求会直接失败而不会发送到后端服务. 断路器保持在开路状态一段时间后(默认5秒), 自动切换到**半开路状态(HALF-OPEN)**. 这时会判断下一次请求的返回情况, 如果请求成功, 断路器切回**闭路状态(CLOSED)**, 否则重新切换到**开路状态(OPEN)**. Hystrix的断路器就像我们家庭电路中的保险丝, 一旦后端服务不可用, 断路器会直接切断请求链, 避免发送大量无效请求影响系统吞吐量, 并且**断路器有自我检测并恢复的能力**.

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/1648997839743-ee7eb326-ff79-43f6-b309-96baf614f2ed.png" alt="dawda.png" style="zoom:67%;" />

- 最开始处于closed状态，一旦检测到错误到达一定阈值，便转为open状态；
- 这时候会有个 reset timeout，到了这个时间了，会转移到half open状态；
- 尝试放行一部分请求到后端，一旦检测成功便回归到closed状态，即恢复服务；

### 资源隔离

在Hystrix中, 主要通过线程池来实现资源隔离. 通常在使用的时候我们会根据调用的远程服务划分出多个线程池. 例如调用产品服务的Command放入A线程池, 调用账户服务的Command放入B线程池. 这样做的主要优点是运行环境被隔离开了. 这样就算调用服务的代码存在bug或者由于其他原因导致自己所在线程池被耗尽时, 不会对系统的其他服务造成影响. 但是带来的代价就是维护多个线程池会对系统带来额外的性能开销. 如果是对性能有严格要求而且确信自己调用服务的客户端代码不会出问题的话, 可以使用Hystrix的信号模式(Semaphores)来隔离资源.

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/1648998195703-7bfb33bf-bfe0-49c8-9fbb-ab99b3ae05ce.png" alt="dawdad.png" style="zoom:67%;" />

**配置多个线程池代码**

- threadPoolKey，这对于Hystrix来说是新建一个线程池的信号，threadPoolKey的值则是线程池的标识。
- 如果只是配置了threadPoolKey，那么Hystrix会使用默认配置来初始化该线程池
- threadPoolProperties。该属性接收一个@HystrixProperty数组，这些HystrixProperty就是用来配置新建的线程池。比如，可以使用coreSize来配置线程池的容量。
- 当然也可以设置一个队列，来应对当线程池繁忙的情况。通过maxQueueSize来设置该队列的容量。当请求的数量超过队列的容量，其它的请求会迅速失败返回，直到队列又有空闲的“位置”。

```java
@HystrixCommand(
    fallbackMethod = "buildFallbackLicenseList",
    threadPoolKey = "licenseByOrgThreadPool",
    threadPoolProperties = {
        @HystrixProperty(name = "coreSize",value="30"),
        @HystrixProperty(name="maxQueueSize", value="10")
    }
)
public List<License> getLicensesByOrg(String organizationId){
    randomlyRunLong();
    return licenseRepository.findByOrganizationId(organizationId);
}
```

### Fallback服务降级

Fallback相当于是降级操作. 对于查询操作, 我们可以实现一个fallback方法, 当请求后端服务出现异常的时候, 可以使用fallback方法返回的值. fallback方法的返回值一般是设置的默认值或者来自缓存.

**降级的情况**

- 程序运行异常
- 超时
- 服务熔断触发服务降级
- 线程池/信号量打满也会导致服务降级

#### 使用

**引入依赖**

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

**启动注解**

```java
@EnableCircuitBreaker // 启用断路器
```

**业务代码加入断路逻辑**

**threadPoolKey**属性添加线程池，实现资源隔离

```java
@RequestMapping("/product/get/{pid}")
@HystrixCommand(fallbackMethod = "getMethodExceptionCallBack", 
    threadPoolKey = "productGetIdServiceThreadPool",
    threadPoolProperties = {@HystrixProperty(name = "coreSize", value = "10"),
        @HystrixProperty(name = "maxQueueSize", value = "10")})
public JsonModel<Product> get(@PathVariable("pid") Integer pid) {
    try {
        TimeUnit.SECONDS.sleep(3);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return this.iProductFeignInterface.get(pid);
}

public JsonModel getMethodExceptionCallBack(Integer id) {
    JsonModel jm = new JsonModel();
    jm.setCode(500);
    jm.setMsg("服务调用异常");
    return jm;
}
```

### Feign Hystrix

**修改消费端配置**

```yaml
feign:
  hystrix:
    enabled: true  #feign默认关闭hystrix
```

**创建回调类**

```java
@Component
public class ProductFallBack implements IProductFeignInterface {
    @Override
    public JsonModel<Product> get(Integer pid) {
        return new JsonModel<>(500, "服务器内部错误，导致接口异常", null);
    }

    @Override
    public JsonModel add(Product product) {
        return new JsonModel(500, "服务器内部错误，导致接口异常", null);
    }

    @Override
    public JsonModel list() {
        return new JsonModel(500, "服务器内部错误，导致接口异常", null);
    }

    @Override
    public JsonModel delete(Integer pid) {
        return new JsonModel(500, "服务器内部错误，导致接口异常", null);
    }
}
```

**添加fallback属性**

```java
@FeignClient(name = "MICROCLOUD-PROVIDER-PRODUCT", configuration = FeignClientAuthConfig.class,
    fallback = ProductFallBack.class)
public interface IProductFeignInterface {}
```

### 服务熔断

**定义：**当下游的服务因为某种原因突然变得不可用或响应过慢，上游服务为了保证自己整体服务的可用性，***不再继续调用目标服务，直接返回，快速释放资源。如果目标服务情况好转则恢复调用

### Hystrix技术点

#### 设计目标

1. 对来自依赖的延迟和故障进行防护和控制——这些依赖通常都是通过网络访问的
2. 阻止故障的连锁反应
3. 快速失败并迅速恢复
4. 回退并优雅降级
5. 提供近实时的监控与告警

#### 设计原则

1. 防止任何单独的依赖耗尽资源（线程）
2. 过载立即切断并快速失败，防止排队
3. 尽可能提供回退以保护用户免受故障
4. 使用隔离技术（例如隔板，泳道和断路器模式）来限制任何一个依赖的影响
5. 通过近实时的指标，监控和告警，确保故障被及时发现
6. 通过动态修改配置属性，确保故障及时恢复
7. 防止整个依赖客户端执行失败，而不仅仅是网络通信

#### 实现原理

1. 使用命令模式将所有对外部服务（或依赖关系）的调用包装在HystrixCommand或HystrixObservableCommand对象中，并将该对象放在单独的线程中执行；
2. 每个依赖都维护着一个线程池（或信号量），线程池被耗尽则拒绝请求（而不是让请求排队）。
3. 记录请求成功，失败，超时和线程拒绝。
4. 服务错误百分比超过了阈值，熔断器开关自动打开，一段时间内停止对该服务的所有请求。
5. 请求失败，被拒绝，超时或熔断时执行降级逻辑。
6. 近实时地监控指标和配置的修改。

## 四.服务监控HystrixDashboard的使用

- 默认的集群监控： http://turbine-hostname:port/turbine.stream
- 指定的集群监控： http://turbine-hostname:port/turbine.stream?cluster=[clusterName]
- 单体应用的监控： http://hystrix-app:port/actuator/hystrix.stream

### 使用

**新建一个监控模块，引入依赖**

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

**添加注解**@EnableHystrixDashboard

```java
@SpringBootApplication
@EnableHystrixDashboard
public class DashboardApp {
    public static void main(String[] args) {
        SpringApplication.run(DashboardApp.class, args);
    }
}
```

**配置文件**

```yaml
server:
  port: 9001
hystrix:
  dashboard:
    proxy-stream-allow-list: "localhost"  #添加代理名称
```

**带hystrix消费端配置**

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yaml
#打开actuator中的端口，默认情况 springboot2之后，这些端口都没有开放
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

