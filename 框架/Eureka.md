# Eureka服务注册与发现

## 一、概念

  Eureka是Netflix公司开源的一款提供服务注册和发现的产品，是一种基于REST的服务，主要用于AWS云，Eureka提供了完整的Service Registry和Service Discovery实现，也是SpringCloud体系中最重要最核心的组件之一。

  Eureka由两个组件组成：**Eureka服务端**和**Eureka客户端**。Eureka服务端就是注册中心。Eureka客户端是一个java客户端，用来简化与服务端的交互、作为轮询负载均衡器，并提供服务的故障切换支持。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/1647951210909-03423fe2-3e7b-4c6f-8694-f7b2baeafeed.png" alt="image.png" width="50%" />

### Erueka Server：注册中心服务端

- **服务注册：**

  服务提供者启动时，会通过**Eureka Client向Eureka Server注册信息**，Eureka Server会**存储**该服务的信息，Eureka Server内部有**二层缓存机制**来维护整个注册表。

- **提供注册表**

  服务消费者在调用服务时，如果**Eureka Client没有缓存注册表的话，会从Eureka Server获取最新的注册表**。

- **同步状态**

  Eureka Client通过**注册、心跳机制和Eureka Server同步**当前客户端的状态。

### Eureka Client：注册中心客户端

  Eureka Client 是一个 Java 客户端，用于简化与 Eureka Server 的交互。Eureka Client 会**拉取、更新和缓存 Eureka Server 中的信息**。因此当所有的 Eureka Server 节点都宕掉，服务消费者依然可以使用**缓存**中的信息找到服务提供者，但是**当服务有更改的时候会出现信息不一致**。

### Register：服务注册

  服务的提供者，将自身注册到注册中心，服务提供者也是一个 Eureka Client。当 Eureka Client 向 Eureka Server 注册时，它提供自身的元数据，比如 IP 地址、端口，运行状况指示符 URL，主页等。

- Spring Cloud Eureka 在应用启动时，会在 EurekaAutoServiceRegistration 这个类初始化的时候，主动去 Eureka Server 端注册。
- Eureka 在启动完成之后会启动**一个 40 秒执行一次的定时任务**，该任务会去**监测自身的 IP 信息以及自身的配置信息**是否发生改变，如果发生改变，则会重新发起注册。
- 续约返回 404 状态码时，会去重新注册

### Renew：服务续约

  Eureka Client 会**每隔 30 秒发送一次心跳来续约**。 通过续约来告知 Eureka Server 该 Eureka Client 运行正常，没有出现问题。 默认情况下，如果 Eureka Server 在 **90 秒内没有收到 Eureka Client 的续约**，Server 端会将实例从其注册表中删除，此时间可配置，一般情况不建议更改。

### Remote Call：远程调用

  当Eureka Client从注册中心获取到服务提供者信息后，就可以通过http请求调用对应的服务；服务提供者有多个时，Eureka Client客户端会通过Ribbon自动进行负载均衡

### GetRegisty: 获取注册列表信息

  Eureka Client 从服务器获取注册表信息，并将其**缓存**在本地。客户端会使用该信息查找其他服务，从而进行远程调用。该注册列表信息定期（每30秒钟）更新一次。每次返回注册列表信息可能与 Eureka Client 的缓存信息不同，Eureka Client 自动处理。

  如果由于某种原因导致注册列表信息不能及时匹配，Eureka Client 则会重新获取整个注册表信息。 Eureka Server 缓存注册列表信息，整个注册表以及每个应用程序的信息进行了压缩，压缩内容和没有压缩的内容完全相同。Eureka Client 和 Eureka Server 可以使用 JSON/XML 格式进行通讯。在默认情况下 Eureka Client 使用压缩 JSON 格式来获取注册列表的信息。

### 自我保护机制

  默认情况下，如果 Eureka Server 在**一定的 90s 内没有接收**到某个微服务实例的心跳，会注销该实例。但是在微服务架构下服务之间通常都是跨进程调用，网络通信往往会面临着各种问题，比如微服务状态正常，网络分区故障，导致此实例被注销。

  固定时间内大量实例被注销，可能会严重威胁整个微服务架构的可用性。为了解决这个问题，Eureka 开发了自我保护机制，那么什么是自我保护机制呢？

  Eureka Server 在**运行期间**会去统计**心跳失败比例在 15 分钟之内是否低于 85%**，**如果低于 85%**，Eureka Server 即会进入自我保护机制。

### Eureka Server进入自我保护机制，会出现一下几种情况

- Eureka 不再从注册列表中移除因为长时间没收到心跳而应该过期的服务
- Eureka 仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上(即保证当前节点依然可用)
- 当网络稳定时，当前实例新的注册信息会被同步到其它节点中

### 注册中心高可用

  理论上来讲，服务消费者本地缓存了服务提供者的地址。即使 Eureka Server 宕机，也不会影响服务之间的调用，但是一旦涉及到服务的上下线，本地的缓存信息将会出现偏差，从而影响到了整个微服务架构的稳定性，因此搭建 Eureka Server 集群来提高整个架构的高可用性，是非常有必要的。这样就可以使注册中心高可用。

## 二、工作流程

1. Eureka Server 启动成功，等待服务端注册。在启动过程中如果配置了集群，集群之间定时通过 Replicate **同步**注册表，每个 Eureka Server 都存在**独立完整的服务注册表信息**
2. Eureka Client 启动时根据配置的 Eureka Server 地址去注册中心注册服务
3. Eureka Client 会**每** **30s** **向 Eureka Server 发送一次心跳请求，证明客户端服务正常**
4. 当 Eureka Server **90s** **内没有收到 Eureka Client 的心跳**，注册中心则认为该节点失效，会注销该实例
5. 单位时间内 Eureka Server 统计到有大量的 Eureka Client 没有上送心跳，则认为可能为网络异常，进入自我保护机制，不再剔除没有上送心跳的客户端（自我保护机制）
6. 当 Eureka Client 心跳请求恢复正常之后，Eureka Server 自动退出自我保护模式
7. Eureka Client **定时全量或者增量**从注册中心获取服务注册表，并且将获取到的信息**缓存**到本地
8. 服务调用时，Eureka Client 会先从本地缓存找寻调取的服务。如果获取不到，先从注册中心刷新注册表，再同步到本地缓存
9. Eureka Client 获取到目标服务器信息，发起服务调用
10. Eureka Client 程序关闭时向 Eureka Server 发送取消请求，Eureka Server 将实例从注册表中删除

## 三、Eureka一致性协议

### 消息广播

  Eureka Server管理了全部的服务器列表（PeerEurekaNodes）

  当Eureka Server收到客户端的注册、下线、心跳请求时，通过PeerEurekaNodes向其余的服务器进行消息广播，如果广播失败则重试，直到任务过期后取消任务，此时两台服务器之间数据会出现短暂的不一致

  如果网络恢复正常，收到了其他服务器广播的心跳任务，此时可以有三种情况：

- [脑裂](https://www.cnblogs.com/nicerblog/p/11232531.html)很快恢复，一切正常 
- 该实例已自动过期，则重新进行注册
- 数据冲突，出现不一致的情况，则需要发起同步请求，其实也就是重新注册一次，同时剔除老的实例

### Eureka保证AP

**CAP理论：A 高可用性 P 分区容错性 C一致性**

  Eureka在设计时**保证可用性**。Eureka各个节点都是平等的，几个节点挂掉不会影响正常节点的工作，**剩余的节点依然可以提供注册和查询服务**。而Eureka的客户端在向某个Eureka注册或如果发现连接失败，则会自动切换至其它节点，只要有一台Eureka还在，就能保证注册服务可用(保证可用性)，只不过查到的信息可能不是最新的(不保证强一致性)。

  除此之外，Eureka还有一种**自我保护机制**，如果在**15分钟内超过85%的节点**都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时会出现以下几种情况：

- Eureka不再从注册列表中移除因为**长时间没收到心跳**而应该**过期**的服务
- Eureka仍然能够接受**新服务的注册和查询请求**，但是不会被同步到其它节点上(即保证当前节点依然可用)
- 当网络稳定时，当前实例新的注册信息会被同步到其它节点中

  因此， Eureka可以很好的应对因网络故障导致部分节点失去联系的情况，而不会像zookeeper那样使整个注册服务瘫痪。

  **Eureka Server集群之间的状态是采用异步方式同步的，所以不保证节点间的状态一定是一致的，不过基本能保证最终状态是一致的。**

## 四、Eureka服务端安全设置

  通过spring-boot-starter-security将Spring Security添加到服务器的类路径中即可保护Eureka服务器。默认情况下，当Spring Security在类路径上时，它将要求在每次向应用程序发送请求时都发送有效的CSRF令牌。Eureka客户通常不会拥有有效的跨站点请求伪造（CSRF）令牌，您需要为/eureka/**端点禁用此要求。例如：

```java
@EnableWebSecurity
class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().ignoringAntMatchers("/eureka/**");
        super.configure(http);
    }
}
```

### CSRF(跨站请求伪造）

  一般来说，攻击者通过伪造用户的浏览器的请求，向访问一个用户自己曾经认证访问过的网站发送出去，使目标网站接收并误以为是用户的真实操作而去执行命令。常用于盗取账号、转账、发送虚假消息等。攻击者利用网站对请求的验证漏洞而实现这样的攻击行为，网站能够确认请求来源于用户的浏览器，却不能验证请求是否源于用户的真实意愿下的操作行为。

### 原理图：

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/1648202187464-e3f6465d-1f38-4cfd-a176-880c2526b706.png" alt="img" width="50%" />

**CSRF攻击原理及过程：**

1. 用户C打开浏览器，访问受信任网站A，输入用户名和密码请求登录网站A；
2. 在用户信息通过验证后，网站A产生Cookie信息并返回给浏览器，此时用户登录网站A成功，可以正常发送请求到网站A；
3. 用户未退出网站A之前，在同一浏览器中，打开一个TAB页访问网站B；
4. 网站B接收到用户请求后，返回一些攻击性代码，并发出一个请求要求访问第三方站点A；
5. 浏览器在接收到这些攻击性代码后，根据网站B的请求，在用户不知情的情况下携带Cookie信息，向网站A发出请求。网站A并不知道该请求其实是由B发起的，所以会根据用户C的Cookie信息以C的权限处理该请求，导致来自网站B的恶意代码被执行。

### 防御CSRF的手段

**目前防御CSRF攻击主要有三种策略**

- 验证HTTP Referer字段
- 在请求地址中添加token并验证
- 在HTTP头中自定义属性并验证

## 五、EurekaClient客户端源码分析

  EurekaClient 为了简化开发人员的工作量，将很多与EurekaServer交互的工作隐藏起来，自主完成。具体完成的工作分为三个阶段， 应用启动阶段,执行阶段, 销毁阶段.  各阶段的工作如下: 

**应用启动阶段：**

1. 读取与 **Eureka Server**交互的配置信息,封装成 **EurekaClientConfig**
2. 读取自身服务实例配置信息,封装成**EurekalnstanceConfig**
3. 从**Eureka server**拉取注册表信息并缓存到本地
4. 服务注册
5. 初始化发送心跳、缓存刷新(拉取注册表信息更新本地缓存)和按需注册(监控服务实例信息变化,决定是否重新发起注册,更新注册表中的服务实例元数据)定时任务

**应用执行阶段：**

1. 定时发送心跳到Eureka Server中维持在注册表的租约
2. 定时从 Eureka Server中拉取注册表信息,更新本地注册表缓存
3. 监控应用自身信息变化,若发生变化,需要重新发起服务注册

**应用销毁阶段：**

1. Eureka Server注销自身服务实例

### 应用启动阶段与运行阶段分析

  Eureka Client通过Starter的方式引人依赖, Spring Boot将会为项目使用以下的自动配置类。

1. EurekaClientAutoConfiguration：EurekeClient 自动配置类,负责Eureka关键Beans的配置和初始化,如AppplicationInfoManager和 EurekaClientConfig等。
2. RibbonEurekaAutoConfiguration: Ribbon负载均衡相关配置。
3. EurekaDiscoveryClientConfiguration:  配置自动注册和应用的健康检查器。

### 读取应用自身配置

  通过 **EurekaClientAutoConfiguration**配置类, Spring boot帮助 Eureka Client完成很多必要Bean的属性读取和配置。

1. EurekaClientConfig:      封装 Eureka Client与 Eureka Server交互所需要的配置信息。 Spring Cloud为其提供了一个默认配置类的EurekaClientConfigBean,可以在配置文件中通过前缀 eureka.client属性名进行属性覆盖
2. ApplicationInfoManager :  作为应用信息管理器,管理服务实例的信息类 InstanceInfo和服务实例的配置信息类 EurekaInstanceConfig
3. InstanceInfo:   封装将被发送到 Eureka Server进行服务注册的服务实例元数据。它在Eurek Server的注册表中代表一个服务实例,其他服务实例可以通过 Instancelnfo了解该服务的相关信息从而发起服务请求
4. EurekaInstanceConfig:   封装EurekaClient自身服务实例的配置信息,主要用于构建InstanceInfo通常这些信息在配置文件中的eureka.instance前缀下进行设置, SpringCloud通过EurekalnstanceConfigBean配置类提供了默认配置
5. DiscoveryClient:   Spring Cloud中定义用来服务发现的客户端接口,  对于DiscoveryClient可以具体查看 EurekaDiscoveryClient，EurekaDiscoveryClient又借助EurekaClient来实现

```java
@Bean  //由spring 托管
public DiscoveryClient discoveryClient(EurekaClient client,
			EurekaClientConfig clientConfig) {
	return new EurekaDiscoveryClient(client, clientConfig);
}
```

**客户端的类结构**

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/1648206960983-e1e20c20-60f8-4069-bac5-3f51c53267bc.png" alt="image.png" width="50%" />

```java
DiscoveryClient(ApplicationInfoManager applicationInfoManager, EurekaClientConfig config, AbstractDiscoveryClientOptionalArgs args, Provider<BackupRegistry> backupRegistryProvider, EndpointRandomizer endpointRandomizer) {
        this.RECONCILE_HASH_CODES_MISMATCH = Monitors.newCounter("DiscoveryClient_ReconcileHashCodeMismatch");
        this.FETCH_REGISTRY_TIMER = Monitors.newTimer("DiscoveryClient_FetchRegistry");
        this.REREGISTER_COUNTER = Monitors.newCounter("DiscoveryClient_Reregister");
        this.localRegionApps = new AtomicReference();
        this.fetchRegistryUpdateLock = new ReentrantLock();
        this.healthCheckHandlerRef = new AtomicReference();
        this.remoteRegionVsApps = new ConcurrentHashMap();
        this.lastRemoteInstanceStatus = InstanceStatus.UNKNOWN;
        this.eventListeners = new CopyOnWriteArraySet();
        this.registrySize = 0;
        this.lastSuccessfulRegistryFetchTimestamp = -1L;
        this.lastSuccessfulHeartbeatTimestamp = -1L;
        this.isShutdown = new AtomicBoolean(false);
        this.stats = new DiscoveryClient.Stats();
        if (args != null) {
            this.healthCheckHandlerProvider = args.healthCheckHandlerProvider;
            this.healthCheckCallbackProvider = args.healthCheckCallbackProvider;
            this.eventListeners.addAll(args.getEventListeners());
            this.preRegistrationHandler = args.preRegistrationHandler;
        } else {
            this.healthCheckCallbackProvider = null;
            this.healthCheckHandlerProvider = null;
            this.preRegistrationHandler = null;
        }

        this.applicationInfoManager = applicationInfoManager;
        InstanceInfo myInfo = applicationInfoManager.getInfo();
        this.clientConfig = config;
        staticClientConfig = this.clientConfig;
        this.transportConfig = config.getTransportConfig();
        this.instanceInfo = myInfo;
        if (myInfo != null) {
            this.appPathIdentifier = this.instanceInfo.getAppName() + "/" + this.instanceInfo.getId();
        } else {
            logger.warn("Setting instanceInfo to a passed in null value");
        }
    
        //传入BackupRegistry（NotImplementedRegistryImpl）备份注册中心
        this.backupRegistryProvider = backupRegistryProvider;
        this.endpointRandomizer = endpointRandomizer;
        this.urlRandomizer = new InstanceInfoBasedUrlRandomizer(this.instanceInfo);
        this.localRegionApps.set(new Applications());
        this.fetchRegistryGeneration = new AtomicLong(0L);
        this.remoteRegionsToFetch = new AtomicReference(this.clientConfig.fetchRegistryForRemoteRegions());
        this.remoteRegionsRef = new AtomicReference(this.remoteRegionsToFetch.get() == null ? null : ((String)this.remoteRegionsToFetch.get()).split(","));
        //从eureka server拉起注册表信息 对应配置项-> eureka.client.fetch-register
        if (config.shouldFetchRegistry()) {
            this.registryStalenessMonitor = new ThresholdLevelsMetric(this, "eurekaClient.registry.lastUpdateSec_", new long[]{15L, 30L, 60L, 120L, 240L, 480L});
        } else {
            this.registryStalenessMonitor = ThresholdLevelsMetric.NO_OP_METRIC;
        }
        // 当前的客户端是否应该注册到erueka中 对应配置项-> eureka.client.register-with-eureka 
        if (config.shouldRegisterWithEureka()) {
            this.heartbeatStalenessMonitor = new ThresholdLevelsMetric(this, "eurekaClient.registration.lastHeartbeatSec_", new long[]{15L, 30L, 60L, 120L, 240L, 480L});
        } else {
            this.heartbeatStalenessMonitor = ThresholdLevelsMetric.NO_OP_METRIC;
        }

        logger.info("Initializing Eureka in region {}", this.clientConfig.getRegion());
        //如果既不需要注册，也不需要拉去数据，直接返回,初始结束
        if (!config.shouldRegisterWithEureka() && !config.shouldFetchRegistry()) {
            logger.info("Client configured to neither register nor query for data.");
            this.scheduler = null;
            this.heartbeatExecutor = null;
            this.cacheRefreshExecutor = null;
            this.eurekaTransport = null;
            this.instanceRegionChecker = new InstanceRegionChecker(new PropertyBasedAzToRegionMapper(config), this.clientConfig.getRegion());
            DiscoveryManager.getInstance().setDiscoveryClient(this);
            DiscoveryManager.getInstance().setEurekaClientConfig(config);
            this.initTimestampMs = System.currentTimeMillis();
            this.initRegistrySize = this.getApplications().size();
            this.registrySize = this.initRegistrySize;
            logger.info("Discovery Client initialized at timestamp {} with initial instances count: {}", this.initTimestampMs, this.initRegistrySize);
        } else {
            try {
                //线程池大小为2，一个用户发送心跳，另外个缓存刷新
                // default size of 2 - 1 each for heartbeat and cacheRefresh
                this.scheduler = Executors.newScheduledThreadPool(2, (new ThreadFactoryBuilder()).setNameFormat("DiscoveryClient-%d").setDaemon(true).build());
                this.heartbeatExecutor = new ThreadPoolExecutor(1, this.clientConfig.getHeartbeatExecutorThreadPoolSize(), 0L, TimeUnit.SECONDS, new SynchronousQueue(), (new ThreadFactoryBuilder()).setNameFormat("DiscoveryClient-HeartbeatExecutor-%d").setDaemon(true).build());
                this.cacheRefreshExecutor = new ThreadPoolExecutor(1, this.clientConfig.getCacheRefreshExecutorThreadPoolSize(), 0L, TimeUnit.SECONDS, new SynchronousQueue(), (new ThreadFactoryBuilder()).setNameFormat("DiscoveryClient-CacheRefreshExecutor-%d").setDaemon(true).build());
                //初始化client与server交互的jersey客户端
                this.eurekaTransport = new DiscoveryClient.EurekaTransport();
                this.scheduleServerEndpointTask(this.eurekaTransport, args);
                Object azToRegionMapper;
                if (this.clientConfig.shouldUseDnsForFetchingServiceUrls()) {
                    azToRegionMapper = new DNSBasedAzToRegionMapper(this.clientConfig);
                } else {
                    azToRegionMapper = new PropertyBasedAzToRegionMapper(this.clientConfig);
                }

                if (null != this.remoteRegionsToFetch.get()) {
                    ((AzToRegionMapper)azToRegionMapper).setRegionsToFetch(((String)this.remoteRegionsToFetch.get()).split(","));
                }

                this.instanceRegionChecker = new InstanceRegionChecker((AzToRegionMapper)azToRegionMapper, this.clientConfig.getRegion());
            } catch (Throwable var12) {
                throw new RuntimeException("Failed to initialize DiscoveryClient!", var12);
            }
            //拉取注册表的信息
            if (this.clientConfig.shouldFetchRegistry()) {
                try {
                    boolean primaryFetchRegistryResult = this.fetchRegistry(false);
                    if (!primaryFetchRegistryResult) {
                        logger.info("Initial registry fetch from primary servers failed");
                    }

                    boolean backupFetchRegistryResult = true;
                    if (!primaryFetchRegistryResult && !this.fetchRegistryFromBackup()) {
                        backupFetchRegistryResult = false;
                        logger.info("Initial registry fetch from backup servers failed");
                    }

                    if (!primaryFetchRegistryResult && !backupFetchRegistryResult && this.clientConfig.shouldEnforceFetchRegistryAtInit()) {
                        throw new IllegalStateException("Fetch registry error at startup. Initial fetch failed.");
                    }
                } catch (Throwable var11) {
                    logger.error("Fetch registry error at startup: {}", var11.getMessage());
                    throw new IllegalStateException(var11);
                }
            }

            if (this.preRegistrationHandler != null) {
                this.preRegistrationHandler.beforeRegistration();
            }

            if (this.clientConfig.shouldRegisterWithEureka() && this.clientConfig.shouldEnforceRegistrationAtInit()) {
                try {
                    if (!this.register()) {
                        throw new IllegalStateException("Registration error at startup. Invalid server response.");
                    }
                } catch (Throwable var10) {
                    logger.error("Registration error at startup: {}", var10.getMessage());
                    throw new IllegalStateException(var10);
                }
            }
            //初始心跳定时任务，缓存刷新
            this.initScheduledTasks();

            try {
                Monitors.registerObject(this);
            } catch (Throwable var9) {
                logger.warn("Cannot register timers", var9);
            }

            DiscoveryManager.getInstance().setDiscoveryClient(this);
            DiscoveryManager.getInstance().setEurekaClientConfig(config);
            this.initTimestampMs = System.currentTimeMillis();
            this.initRegistrySize = this.getApplications().size();
            this.registrySize = this.initRegistrySize;
            logger.info("Discovery Client initialized at timestamp {} with initial instances count: {}", this.initTimestampMs, this.initRegistrySize);
        }
    }
```

**总结：**

1. 相关配置赋值
2. 备份注册中心的初始化，实际什么事都没做
3. **拉取Server注册表中的信息**
4. 注册前的预处理
5. **向Server注册自身**
6. **初始心跳定时任务，缓存刷新等定时任务**

### 拉取Server注册表中的信息

```java
//拉取注册信息，拉取的方法分为全量拉取和增量拉取.
private boolean fetchRegistry(boolean forceFullRegistryFetch) {
        Stopwatch tracer = this.FETCH_REGISTRY_TIMER.start();

        label122: {
            boolean var4;
            try {
                //如果增量拉取被禁止,则全量拉取
                Applications applications = this.getApplications();
                if (!this.clientConfig.shouldDisableDelta() && Strings.isNullOrEmpty(this.clientConfig.getRegistryRefreshSingleVipAddress()) && !forceFullRegistryFetch && applications != null && applications.getRegisteredApplications().size() != 0 && applications.getVersion() != -1L) {
                    //增量拉取
                    this.getAndUpdateDelta(applications);
                } else {
                    logger.info("Disable delta property : {}", this.clientConfig.shouldDisableDelta());
                    logger.info("Single vip registry refresh property : {}", this.clientConfig.getRegistryRefreshSingleVipAddress());
                    logger.info("Force full registry fetch : {}", forceFullRegistryFetch);
                    logger.info("Application is null : {}", applications == null);  //如果是第一次拉取，applications为null,则全量拉取
                    logger.info("Registered Applications size is zero : {}", applications.getRegisteredApplications().size() == 0);
                    logger.info("Application version is -1: {}", applications.getVersion() == -1L);
                    //全量拉取
                    this.getAndStoreFullRegistry();
                }
                applications.setAppsHashCode(applications.getReconcileHashCode());
                //打印注册表上所有服务实例信息
                this.logTotalInstances();
                break label122;
            } catch (Throwable var8) {
                logger.info("DiscoveryClient_{} - was unable to refresh its cache! This periodic background refresh will be retried in {} seconds. status = {} stacktrace = {}", new Object[]{this.appPathIdentifier, this.clientConfig.getRegistryFetchIntervalSeconds(), var8.getMessage(), ExceptionUtils.getStackTrace(var8)});
                var4 = false;
            } finally {
                if (tracer != null) {
                    tracer.stop();
                }
            }
            return var4;
        }
        this.onCacheRefreshed();
        this.updateInstanceRemoteStatus();
        return true;
    }
```

### 全量拉取

全量拉取一般只有发生在第一次拉去注册表信息的时候，全量拉取调用 getAndStoreFullRegistry()方法

```java
private void getAndStoreFullRegistry() throws Throwable {
        //拉取注册表的版本信息
        long currentUpdateGeneration = this.fetchRegistryGeneration.get();
        logger.info("Getting all instance registry info from the eureka server");
        Applications apps = null;
        EurekaHttpResponse<Applications> httpResponse = this.clientConfig.getRegistryRefreshSingleVipAddress() == null ? this.eurekaTransport.queryClient.getApplications((String[])this.remoteRegionsRef.get()) : this.eurekaTransport.queryClient.getVip(this.clientConfig.getRegistryRefreshSingleVipAddress(), (String[])this.remoteRegionsRef.get());
        //拉取成功,则保存信息到 apps中
        if (httpResponse.getStatusCode() == Status.OK.getStatusCode()) {
            apps = (Applications)httpResponse.getEntity();
        }

        logger.info("The response status is {}", httpResponse.getStatusCode());
        if (apps == null) {
            logger.error("The application is null for some reason. Not storing this information");
        } else if (this.fetchRegistryGeneration.compareAndSet(currentUpdateGeneration, currentUpdateGeneration + 1L)) {
            this.localRegionApps.set(this.filterAndShuffle(apps));
            logger.debug("Got full registry with apps hashcode {}", apps.getAppsHashCode());
        } else {
            logger.warn("Not updating applications as another thread is updating it already");
        }

    }
```

### 增量拉取

增量拉取对应的方法为: getAndUpdateDelta(applications)

```java
private void getAndUpdateDelta(Applications applications) throws Throwable {
        long currentUpdateGeneration = this.fetchRegistryGeneration.get();
        //拉取信息
        Applications delta = null;
        EurekaHttpResponse<Applications> httpResponse = this.eurekaTransport.queryClient.getDelta((String[])this.remoteRegionsRef.get());
        if (httpResponse.getStatusCode() == Status.OK.getStatusCode()) {
            delta = (Applications)httpResponse.getEntity();
        }
        //如果拉取失败进行全量拉取
        if (delta == null) {
            logger.warn("The server does not allow the delta revision to be applied because it is not safe. Hence got the full registry.");
            this.getAndStoreFullRegistry();
        } else if (this.fetchRegistryGeneration.compareAndSet(currentUpdateGeneration, currentUpdateGeneration + 1L)) {
            logger.debug("Got delta update with apps hashcode {}", delta.getAppsHashCode());
            String reconcileHashCode = "";
            if (this.fetchRegistryUpdateLock.tryLock()) {
                try {
                    //更新本地缓存
                    this.updateDelta(delta);
                    reconcileHashCode = this.getReconcileHashCode(applications);
                } finally {
                    this.fetchRegistryUpdateLock.unlock();
                }
            } else {
                logger.warn("Cannot acquire update lock, aborting getAndUpdateDelta");
            }

            if (!reconcileHashCode.equals(delta.getAppsHashCode()) || this.clientConfig.shouldLogDeltaDiff()) {
                this.reconcileAndLogDifference(delta, reconcileHashCode);
            }
        } else {
            logger.warn("Not updating application delta as another thread is updating it already");
            logger.debug("Ignoring delta update with apps hashcode {}, as another thread is updating it already", delta.getAppsHashCode());
        }

    }
```

### 向服务器注册

```java
boolean register() throws Throwable {
        logger.info("DiscoveryClient_{}: registering service...", this.appPathIdentifier);

        EurekaHttpResponse httpResponse;
        try {
            //把自身的实例发送给服务端
            httpResponse = this.eurekaTransport.registrationClient.register(this.instanceInfo);
        } catch (Exception var3) {
            logger.warn("DiscoveryClient_{} - registration failed {}", new Object[]{this.appPathIdentifier, var3.getMessage(), var3});
            throw var3;
        }

        if (logger.isInfoEnabled()) {
            logger.info("DiscoveryClient_{} - registration status: {}", this.appPathIdentifier, httpResponse.getStatusCode());
        }
        // 在http的响应状态码中, 204意思等同于请求执行成功，但是没有数据，浏览器不用刷新页面.也不用导向新的页面
        return httpResponse.getStatusCode() == Status.NO_CONTENT.getStatusCode();  //204
    }
```

### 定时任务: 

 initScheduledTasks()是负责定时任务的相关方法。

```java
private void initScheduledTasks() {
        int renewalIntervalInSecs;
        int expBackOffBound;
        if (this.clientConfig.shouldFetchRegistry()) {
            // 拉取服务默认30秒，eureka.client.register-fetch-interval-seconds
            renewalIntervalInSecs = this.clientConfig.getRegistryFetchIntervalSeconds();
            //更新缓存  
            expBackOffBound = this.clientConfig.getCacheRefreshExecutorExponentialBackOffBound();
            this.cacheRefreshTask = new TimedSupervisorTask("cacheRefresh", this.scheduler, this.cacheRefreshExecutor, renewalIntervalInSecs, TimeUnit.SECONDS, expBackOffBound, new DiscoveryClient.CacheRefreshThread());
            // 心跳服务，默认30秒
            this.scheduler.schedule(this.cacheRefreshTask, (long)renewalIntervalInSecs, TimeUnit.SECONDS);
        }

        if (this.clientConfig.shouldRegisterWithEureka()) {
            renewalIntervalInSecs = this.instanceInfo.getLeaseInfo().getRenewalIntervalInSecs();
            expBackOffBound = this.clientConfig.getHeartbeatExecutorExponentialBackOffBound();
            logger.info("Starting heartbeat executor: renew interval is: {}", renewalIntervalInSecs);
            this.heartbeatTask = new TimedSupervisorTask("heartbeat", this.scheduler, this.heartbeatExecutor, renewalIntervalInSecs, TimeUnit.SECONDS, expBackOffBound, new DiscoveryClient.HeartbeatThread());
            this.scheduler.schedule(this.heartbeatTask, (long)renewalIntervalInSecs, TimeUnit.SECONDS);
            this.instanceInfoReplicator = new InstanceInfoReplicator(this, this.instanceInfo, this.clientConfig.getInstanceInfoReplicationIntervalSeconds(), 2);
            this.statusChangeListener = new StatusChangeListener() {
                public String getId() {
                    return "statusChangeListener";
                }

                public void notify(StatusChangeEvent statusChangeEvent) {
                    DiscoveryClient.logger.info("Saw local status change event {}", statusChangeEvent);
                    DiscoveryClient.this.instanceInfoReplicator.onDemandUpdate();
                }
            };
            if (this.clientConfig.shouldOnDemandUpdateStatusChange()) {
                this.applicationInfoManager.registerStatusChangeListener(this.statusChangeListener);
            }

            this.instanceInfoReplicator.start(this.clientConfig.getInitialInstanceInfoReplicationIntervalSeconds());
        } else {
            logger.info("Not registering with Eureka server per configuration");
        }

    }
```

### 服务下线

```java
public synchronized void shutdown() {
        if (this.isShutdown.compareAndSet(false, true)) {
            logger.info("Shutting down DiscoveryClient ...");
            if (this.statusChangeListener != null && this.applicationInfoManager != null) {
                //注销状态监听器
                this.applicationInfoManager.unregisterStatusChangeListener(this.statusChangeListener.getId());
            }
            //取消定时任务
            this.cancelScheduledTasks();
            if (this.applicationInfoManager != null && this.clientConfig.shouldRegisterWithEureka() && this.clientConfig.shouldUnregisterOnShutdown()) {
                this.applicationInfoManager.setInstanceStatus(InstanceStatus.DOWN);
                this.unregister();
            }
            //关闭与server连接的客户端
            if (this.eurekaTransport != null) {
                this.eurekaTransport.shutdown();
            }
            //关闭相关监控
            this.heartbeatStalenessMonitor.shutdown();
            this.registryStalenessMonitor.shutdown();
            Monitors.unregisterObject(this);
            logger.info("Completed shut down of DiscoveryClient");
        }

    }
```

## 六、EurekaServer服务端源码分析

  EurekaServerAutoConfiguration 是通过配置文件注册（@Bean)。EurekaServer 是服务的注册中心，负责Eureka Client的相关信息注册，主要职责：

- 服务注册
- 接受心跳服务
- 服务剔除:  由eurekaServer主动剔除 心跳超时的服务
- 服务下线:  由 eurekaClient主动发送下线通知后，由eurekaServer接收后，删除服务
- 集群同步

  EurekaServerAutoConfiguration 是通过配置文件注册,并由spring boot完成加载. 在这个配置类中，它创建了 PeerAwareInstanceRegistry的对象. 这个对象主要是基于 节点感知的实例信息注册. 

```java
	@Bean
	public PeerAwareInstanceRegistry peerAwareInstanceRegistry(
			ServerCodecs serverCodecs) {
		this.eurekaClient.getApplications(); // force initialization
		return new InstanceRegistry(this.eurekaServerConfig, this.eurekaClientConfig,
				serverCodecs, this.eurekaClient,
				this.instanceRegistryProperties.getExpectedNumberOfClientsSendingRenews(),
				this.instanceRegistryProperties.getDefaultOpenForTrafficCount());
	}
```

PeerWareInstanceRegistry的类层次结构如下:

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/1648209049715-a3b5e826-4fa2-4a15-b260-ec50c8c655c0.png" alt="image.png" width="30%" />

### 最上层的接口LeaseManager

```java
public interface LeaseManager<T> {
    //注册
    void register(T var1, int var2, boolean var3);
    //下线
    boolean cancel(String var1, String var2, boolean var3);
    //更新
    boolean renew(String var1, String var2, boolean var3);
    //服务剔除
    void evict();
}
```

### 应用实例注册表接口

   com.netflix.eureka.registry.InstanceRegistry，应用实例注册表接口。它继承了 LookupService 、LeaseManager 接口，提供应用实例的注册与发现服务。另外，它结合实际业务场景，定义了更加丰富的接口方法

```java
public interface InstanceRegistry extends LeaseManager<InstanceInfo>, LookupService<String> {
     // ====== 开启与关闭相关 ======
    void openForTraffic(ApplicationInfoManager var1, int var2);	
	void shutdown();

    /** @deprecated */
    @Deprecated //已弃用
    void storeOverriddenStatusIfRequired(String var1, InstanceStatus var2);
    // ====== 应用实例状态变更相关 ======
    void storeOverriddenStatusIfRequired(String var1, String var2, InstanceStatus var3);

    boolean statusUpdate(String var1, String var2, InstanceStatus var3, String var4, boolean var5);

    boolean deleteStatusOverride(String var1, String var2, InstanceStatus var3, String var4, boolean var5);

    Map<String, InstanceStatus> overriddenInstanceStatusesSnapshot();

    Applications getApplicationsFromLocalRegionOnly();

    List<Application> getSortedApplications();

    Application getApplication(String var1, boolean var2);

    InstanceInfo getInstanceByAppAndId(String var1, String var2);

    InstanceInfo getInstanceByAppAndId(String var1, String var2, boolean var3);

    void clearRegistry();
    // ====== 响应缓存相关 ======
    void initializedResponseCache();

    ResponseCache getResponseCache();

    long getNumOfRenewsInLastMin();
    // ====== 自我保护模式相关 ======
    int getNumOfRenewsPerMinThreshold();

    int isBelowRenewThresold();
    // ====== 调试/监控相关 ======
    List<Pair<Long, String>> getLastNRegisteredInstances();

    List<Pair<Long, String>> getLastNCanceledInstances();

    boolean isLeaseExpirationEnabled();

    boolean isSelfPreservationModeEnabled();
}
```
  而PeerAwareInstanceRegistryImpl是一个子类的实现，在上面的基础上扩展对集群的同步操作，使Eureaka Server集群信息保持一致.

```java
public interface PeerAwareInstanceRegistry extends InstanceRegistry {
    void init(PeerEurekaNodes var1) throws Exception;

    int syncUp();  //同步

    boolean shouldAllowAccess(boolean var1);

    void register(InstanceInfo var1, boolean var2); //注册

    void statusUpdate(String var1, ASGStatus var2, boolean var3);
}
```

### 服务注册

  . com.netflix.eureka.registry.AbstractInstanceRegistry#register 这方法是负责服务的注册的

```java
public void register(InstanceInfo registrant, int leaseDuration, boolean isReplication) {
        //获取读锁
        this.read.lock();

        try {
            //   gMap  其实可以发现，这里注册中心其实是个ConcurrentHashMap
            Map<String, Lease<InstanceInfo>> gMap = (Map)this.registry.get(registrant.getAppName());
            EurekaMonitors.REGISTER.increment(isReplication);
            if (gMap == null) {
                ConcurrentHashMap<String, Lease<InstanceInfo>> gNewMap = new ConcurrentHashMap();
                //key 为appName, 如果存在，返回存在的值，否则添加，返回null
                gMap = (Map)this.registry.putIfAbsent(registrant.getAppName(), gNewMap);
                if (gMap == null) {
                    gMap = gNewMap;
                }
            }
            //根据instanceId获取实例的租约
            Lease<InstanceInfo> existingLease = (Lease)((Map)gMap).get(registrant.getId());
            if (existingLease != null && existingLease.getHolder() != null) {
                Long existingLastDirtyTimestamp = ((InstanceInfo)existingLease.getHolder()).getLastDirtyTimestamp();
                Long registrationLastDirtyTimestamp = registrant.getLastDirtyTimestamp();
                logger.debug("Existing lease found (existing={}, provided={}", existingLastDirtyTimestamp, registrationLastDirtyTimestamp);
                //如果该实例的租约已经存在，比较最后的更新时间戳大小，取最大值的注册信息信息
                if (existingLastDirtyTimestamp > registrationLastDirtyTimestamp) {
                    logger.warn("There is an existing lease and the existing lease's dirty timestamp {} is greater than the one that is being registered {}", existingLastDirtyTimestamp, registrationLastDirtyTimestamp);
                    logger.warn("Using the existing instanceInfo instead of the new instanceInfo as the registrant");
                    registrant = (InstanceInfo)existingLease.getHolder();
                }
            } else {
                //如果租约不存在，注册一个新的实例
                synchronized(this.lock) {
                    if (this.expectedNumberOfClientsSendingRenews > 0) {
                        ++this.expectedNumberOfClientsSendingRenews;
                        this.updateRenewsPerMinThreshold();
                    }
                }

                logger.debug("No previous lease information found; it is new registration");
            }
            //创建新的租约
            Lease<InstanceInfo> lease = new Lease(registrant, leaseDuration);
            if (existingLease != null) {
                lease.setServiceUpTimestamp(existingLease.getServiceUpTimestamp());
            }
            //保存租约到map中
            ((Map)gMap).put(registrant.getId(), lease);
            //获得最近注册队列
            this.recentRegisteredQueue.add(new Pair(System.currentTimeMillis(), registrant.getAppName() + "(" + registrant.getId() + ")"));
            if (!InstanceStatus.UNKNOWN.equals(registrant.getOverriddenStatus())) {
                logger.debug("Found overridden status {} for instance {}. Checking to see if needs to be add to the overrides", registrant.getOverriddenStatus(), registrant.getId());
                if (!this.overriddenInstanceStatusMap.containsKey(registrant.getId())) {
                    logger.info("Not found overridden id {} and hence adding it", registrant.getId());
                    this.overriddenInstanceStatusMap.put(registrant.getId(), registrant.getOverriddenStatus());
                }
            }

            InstanceStatus overriddenStatusFromMap = (InstanceStatus)this.overriddenInstanceStatusMap.get(registrant.getId());
            if (overriddenStatusFromMap != null) {
                logger.info("Storing overridden status {} from map", overriddenStatusFromMap);
                registrant.setOverriddenStatus(overriddenStatusFromMap);
            }

            InstanceStatus overriddenInstanceStatus = this.getOverriddenInstanceStatus(registrant, existingLease, isReplication);
            registrant.setStatusWithoutDirty(overriddenInstanceStatus);
            if (InstanceStatus.UP.equals(registrant.getStatus())) {
                lease.serviceUp();
            }

            registrant.setActionType(ActionType.ADDED);
            this.recentlyChangedQueue.add(new AbstractInstanceRegistry.RecentlyChangedItem(lease));
            registrant.setLastUpdatedTimestamp();
            this.invalidateCache(registrant.getAppName(), registrant.getVIPAddress(), registrant.getSecureVipAddress());
            logger.info("Registered instance {}/{} with status {} (replication={})", new Object[]{registrant.getAppName(), registrant.getId(), registrant.getStatus(), isReplication});
        } finally {
            //释放锁
            this.read.unlock();
        }

    }
```

### 接受心跳服务:

  在Eureka Client完成服务的注册后，需要定时向Eureka Server发送心跳请求（默认30s）,维持自己在EurekaServer的租约有效性

```java
public boolean renew(String appName, String id, boolean isReplication) {
        EurekaMonitors.RENEW.increment(isReplication);
        //根据appName获取服务集群租约集合
        Map<String, Lease<InstanceInfo>> gMap = (Map)this.registry.get(appName);
        Lease<InstanceInfo> leaseToRenew = null;
        if (gMap != null) {
            leaseToRenew = (Lease)gMap.get(id);
        }
        //如果租约不存在，直接返回false
        if (leaseToRenew == null) {
            EurekaMonitors.RENEW_NOT_FOUND.increment(isReplication);
            logger.warn("DS: Registry: lease doesn't exist, registering resource: {} - {}", appName, id);
            return false;
        } else {
            InstanceInfo instanceInfo = (InstanceInfo)leaseToRenew.getHolder();
            if (instanceInfo != null) {
                //得到服务的最终状态
                InstanceStatus overriddenInstanceStatus = this.getOverriddenInstanceStatus(instanceInfo, leaseToRenew, isReplication);
                if (overriddenInstanceStatus == InstanceStatus.UNKNOWN) {
                    //如果状态为UNKNOWN，取消续约
                    logger.info("Instance status UNKNOWN possibly due to deleted override for instance {}; re-register required", instanceInfo.getId());
                    EurekaMonitors.RENEW_NOT_FOUND.increment(isReplication);
                    return false;
                }

                if (!instanceInfo.getStatus().equals(overriddenInstanceStatus)) {
                    logger.info("The instance status {} is different from overridden instance status {} for instance {}. Hence setting the status to overridden status", new Object[]{instanceInfo.getStatus().name(), overriddenInstanceStatus.name(), instanceInfo.getId()});
                    instanceInfo.setStatusWithoutDirty(overriddenInstanceStatus);
                }
            }

            this.renewsLastMin.increment();
            //跟新续约有效时间
            leaseToRenew.renew();
            return true;
        }
    }
```

### 服务剔除:  

  如果Eureka Client在注册后，由于服务的崩溃或网络异常导致既没有续约，也没有下线，那么服务就处于不可知的状态，需要剔除这些服务 , 通过com.netflix.eureka.registry.AbstractInstanceRegistry#evict(long)完成,  这是个定时任务调用的方法.  在com.netflix.eureka.registry.AbstractInstanceRegistry#postInit 中使用 AbstractInstanceRegistry.EvictionTask 负责调用(默认60s)

```java
public void evict(long additionalLeaseMs) {
        logger.debug("Running the evict task");
        //如果自我保护状态，不允许剔除服务 配置项:   enable-self-preservation: true
        if (!this.isLeaseExpirationEnabled()) {
            logger.debug("DS: lease expiration is currently disabled.");
        } else {
            List<Lease<InstanceInfo>> expiredLeases = new ArrayList();
            //遍历注册表registry，获取所有过期的租约
            Iterator var4 = this.registry.entrySet().iterator();

            while(true) {
                Map leaseMap;
                do {
                    if (!var4.hasNext()) {
                        //获取注册表租约总数
                        int registrySize = (int)this.getLocalRegistrySize();
                        int registrySizeThreshold = (int)((double)registrySize * this.serverConfig.getRenewalPercentThreshold());
                        //计算最多允许剔除的阈值
                        int evictionLimit = registrySize - registrySizeThreshold;
                        //两者中取小的值，为本次剔除的数量
                        int toEvict = Math.min(expiredLeases.size(), evictionLimit);
                        if (toEvict > 0) {
                            logger.info("Evicting {} items (expired={}, evictionLimit={})", new Object[]{toEvict, expiredLeases.size(), evictionLimit});
                            Random random = new Random(System.currentTimeMillis());
                            //逐个剔除
                            for(int i = 0; i < toEvict; ++i) {
                                int next = i + random.nextInt(expiredLeases.size() - i);
                                Collections.swap(expiredLeases, i, next);
                                Lease<InstanceInfo> lease = (Lease)expiredLeases.get(i);
                                String appName = ((InstanceInfo)lease.getHolder()).getAppName();
                                String id = ((InstanceInfo)lease.getHolder()).getId();
                                EurekaMonitors.EXPIRED.increment();
                                logger.warn("DS: Registry: expired lease for {}/{}", appName, id);
                                //剔除
                                this.internalCancel(appName, id, false);
                            }
                        }

                        return;
                    }

                    Entry<String, Map<String, Lease<InstanceInfo>>> groupEntry = (Entry)var4.next();
                    leaseMap = (Map)groupEntry.getValue();
                } while(leaseMap == null);

                Iterator var7 = leaseMap.entrySet().iterator();

                while(var7.hasNext()) {
                    Entry<String, Lease<InstanceInfo>> leaseEntry = (Entry)var7.next();
                    Lease<InstanceInfo> lease = (Lease)leaseEntry.getValue();
                    if (lease.isExpired(additionalLeaseMs) && lease.getHolder() != null) {
                        expiredLeases.add(lease);
                    }
                }
            }
        }
    }
```

### 服务下线:   

  EurekaClient在应用销毁时候，会向Eureka Server发送下线请求对于服务端的服务下线，其主要代码对应在com.netflix.eureka.registry.AbstractInstanceRegistry#cancel.

```java
public boolean cancel(String appName, String id, boolean isReplication) {
        return this.internalCancel(appName, id, isReplication);
    }
 
    protected boolean internalCancel(String appName, String id, boolean isReplication) {
        boolean var10;
        try {
            //读锁，防止被其他线程进行修改
            this.read.lock();
            EurekaMonitors.CANCEL.increment(isReplication);
            //根据appName获取服务实例集群
            Map<String, Lease<InstanceInfo>> gMap = (Map)this.registry.get(appName);
            Lease<InstanceInfo> leaseToCancel = null;
            //移除服务实例租约
            if(gMap != null) {
                leaseToCancel = (Lease)gMap.remove(id);
            }
 
            AbstractInstanceRegistry.CircularQueue var6 = this.recentCanceledQueue;
            synchronized(this.recentCanceledQueue) {
                this.recentCanceledQueue.add(new Pair(Long.valueOf(System.currentTimeMillis()), appName + "(" + id + ")"));
            }
 
            InstanceStatus instanceStatus = (InstanceStatus)this.overriddenInstanceStatusMap.remove(id);
            if(instanceStatus != null) {
                logger.debug("Removed instance id {} from the overridden map which has value {}", id, instanceStatus.name());
            }
            //租约不存在，返回false
            if(leaseToCancel == null) {
                EurekaMonitors.CANCEL_NOT_FOUND.increment(isReplication);
                logger.warn("DS: Registry: cancel failed because Lease is not registered for: {}/{}", appName, id);
                boolean var17 = false;
                return var17;
            }
            //设置租约的下线时间
            leaseToCancel.cancel();
            InstanceInfo instanceInfo = (InstanceInfo)leaseToCancel.getHolder();
            String vip = null;
            String svip = null;
            if(instanceInfo != null) {
                instanceInfo.setActionType(ActionType.DELETED);
                this.recentlyChangedQueue.add(new AbstractInstanceRegistry.RecentlyChangedItem(leaseToCancel));
                instanceInfo.setLastUpdatedTimestamp();
                vip = instanceInfo.getVIPAddress();
                svip = instanceInfo.getSecureVipAddress();
            }
            //设置缓存过期
            this.invalidateCache(appName, vip, svip);
            logger.info("Cancelled instance {}/{} (replication={})", new Object[]{appName, id, Boolean.valueOf(isReplication)});
            var10 = true;
        } finally {
            //释放锁
            this.read.unlock();
        }
 
        return var10;
    }
```

### 集群同步

  如果Eureka Server是通过集群方式进行部署，为了为维护整个集群中注册表数据一致性所以集群同步也是非常重要得事情。

集群同步分为两部分: 

1. EurekaServer在启动过程中从他的peer节点中拉取注册表信息，并将这些服务实例注册到本地注册表中；
2. 另外eureka server每次对本地注册表进行操作时，同时会将操作同步到他的peer节点中，达到数据一致；

### Eureka Server初始化本地注册表信息:

  在eureka server启动过程中，会从它的peer节点中拉取注册表来初始化本地注册表，这部分主要通过com.netflix.eureka.registry.PeerAwareInstanceRegistryImpl#syncUp ,他从可能存在的peer节点中，拉取peer节点中的注册表信息，并将其中的服务实例的信息注册到本地注册表中。

```java
public int syncUp() {
        // Copy entire entry from neighboring DS node
        int count = 0;
        // serverConfig指的就是服务器配置信息,即要同步的eureka服务器列表
        for (int i = 0; ((i < serverConfig.getRegistrySyncRetries()) && (count == 0)); i++) {
            if (i > 0) {
                try {
                    //根据配置停 一下
                    Thread.sleep(serverConfig.getRegistrySyncRetryWaitMs());
                } catch (InterruptedException e) {
                    logger.warn("Interrupted during registry transfer..");
                    break;
                }
            }
            //获取每个eureka服务器中所有应用的所有的服务实例
            Applications apps = eurekaClient.getApplications();
            for (Application app : apps.getRegisteredApplications()) {
                for (InstanceInfo instance : app.getInstances()) {
                    try {
                         //判断是否可以注册
                        if (isRegisterable(instance)) {
                            //注册到自身的注册表中
                            register(instance, instance.getLeaseInfo().getDurationInSecs(), true);
                            count++;
                        }
                    } catch (Throwable t) {
                        logger.error("During DS init copy", t);
                    }
                }
            }
        }
        return count;
    }
```

通过这一步保证了eureka启动时的数据一致性(第一次数据拉取).

### Eureka Server之间注册表信息同步复制

  为了保证Eureka Server集群运行时候注册表的信息一致性，每个eureka server在对本地注册表进行管理操作时，会将相应的信息同步到peer节点中。

以下的各个方法中都会调用同步复制.

- com.netflix.eureka.registry.PeerAwareInstanceRegistryImpl#cancel
- com.netflix.eureka.registry.PeerAwareInstanceRegistryImpl#register
- com.netflix.eureka.registry.PeerAwareInstanceRegistryImpl#renew等方法中，都回调用方法
- com.netflix.eureka.registry.PeerAwareInstanceRegistryImpl#replicateToPeers

```java
 private void replicateToPeers(Action action, String appName, String id,
                                  InstanceInfo info /* optional */,
                                  InstanceStatus newStatus /* optional */, boolean isReplication) {
        Stopwatch tracer = action.getTimer().start();
        try {
            if (isReplication) {
                numberOfReplicationsLastMin.increment();
            }
            // If it is a replication already, do not replicate again as this will create a poison replication
            if (peerEurekaNodes == Collections.EMPTY_LIST || isReplication) {
                return;
            }
//向peer集群中的每一个peer进行同步
            for (final PeerEurekaNode node : peerEurekaNodes.getPeerEurekaNodes()) {
                // If the url represents this host, do not replicate to yourself.
//是自已就不同步了
                if (peerEurekaNodes.isThisMyUrl(node.getServiceUrl())) {
                    continue;
                }
                //根据action调用不同的同步请求
                replicateInstanceActionsToPeers(action, appName, id, info, newStatus, node);
            }
        } finally {
            tracer.stop();
        }
    }
```

方法中会通过for循环遍历所有的PeerEurekaNode，调用replicateInstanceActionsToPeers方法，把信息复制给其他的Eureka Server节点. 

```java
 /**
     * Replicates all instance changes to peer eureka nodes except for
     * replication traffic to this node.
     *
     */
    private void replicateInstanceActionsToPeers(Action action, String appName,
                                                 String id, InstanceInfo info, InstanceStatus newStatus,
                                                 PeerEurekaNode node) {
        try {
            InstanceInfo infoFromRegistry = null;
            CurrentRequestVersion.set(Version.V2);
            switch (action) {
                case Cancel:
                    node.cancel(appName, id);
                    break;
                case Heartbeat:
                    InstanceStatus overriddenStatus = overriddenInstanceStatusMap.get(id);
                    infoFromRegistry = getInstanceByAppAndId(appName, id, false);
                    node.heartbeat(appName, id, infoFromRegistry, overriddenStatus, false);
                    break;
                case Register:
                    node.register(info);
                    break;
                case StatusUpdate:
                    infoFromRegistry = getInstanceByAppAndId(appName, id, false);
                    node.statusUpdate(appName, id, newStatus, infoFromRegistry);
                    break;
                case DeleteStatusOverride:
                    infoFromRegistry = getInstanceByAppAndId(appName, id, false);
                    node.deleteStatusOverride(appName, id, infoFromRegistry);
                    break;
            }
        } catch (Throwable t) {
            logger.error("Cannot replicate information to {} for action {}", node.getServiceUrl(), action.name(), t);
        }
}
```

方法中，会判断action具体的动作,调用节点不同的操作. 以注册为例，查看一下注册的操作.

```java
	/**
     * Sends the registration information of {@link InstanceInfo} receiving by
     * this node to the peer node represented by this class.
     *
     * @param info
     *            the instance information {@link InstanceInfo} of any instance
     *            that is send to this instance.
     * @throws Exception
     */
    public void register(final InstanceInfo info) throws Exception {
        long expiryTime = System.currentTimeMillis() + getLeaseRenewalOf(info);
        batchingDispatcher.process(
                taskId("register", info),
                new InstanceReplicationTask(targetHost, Action.Register, info, null, true) {
                    public EurekaHttpResponse<Void> execute() {
                        return replicationClient.register(info);
                    }
                },
                expiryTime
        );
    }
```

## 七、SpringBoot整合Eureka

### EurekaServer服务端

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
	<version>3.1.2</version>
</dependency>
```

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServer {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer.class, args);
    }
}
```

```yml
server:
  port: 7001

eureka:
  instance:
    hostname: eureka7001.com
  client:
    #false 表示不向注册中心注册自己
    register-with-eureka: false
    #false 表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    #覆盖默认ip地址
    service-url:
      defaultZone: http://eureka.com:7001/eureka

spring:
  application:
    name: eureka_server
```

### EurekaClient客户端

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableEurekaClient
public class ZkRedisMain {
    public static void main(String[] args) {
        SpringApplication.run(ZkRedisMain.class, args);
    }
}
```

```yml
server:
  port: 8081
eureka:
  instance:
  	instance-id: zk-redis_8081
 	 prefer-ip-address: true
  client:
  	register-with-eureka: true
  	fetch-registry: true
  	serviceUrl:
  		defaultZone: http://eureka.com:7001/eureka	
ribbon:
  #  指定是建立连接所用的时间，适用于网络状况正常的情况下，两端连接所有的时间
  ReadTimeout: 5000
  #指定是建立连接后从服务器读取到可用资源所用的时间
  ConnectionTimeout: 5000
logging:
  level:
    root: warn
```

