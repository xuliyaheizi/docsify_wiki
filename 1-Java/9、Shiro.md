## 定义
shiro是一个功能强大且易于使用的Java安全框架，它执行身份验证、授权、加密和会话管理。
## Shiro核心架构
![image.png](https://cdn.nlark.com/yuque/0/2022/png/22980297/1646796622378-67b770f2-a9ad-419f-a3ee-aa4e096180b2.png#clientId=ua7d078d8-e1e0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=230&id=u3ab18f86&name=image.png&originHeight=307&originWidth=385&originalType=binary&ratio=1&rotation=0&showTitle=false&size=116313&status=done&style=shadow&taskId=u54cde148-857f-4c79-904e-c4379dca628&title=&width=289)
## Subject
**subject即主体**，外部应用与subject进行交互，subject记录了当前的操作用户，将用户的概率理解为当前操作的主体。外部程序通过subject进行认证授权，而subject是通过**SecurityManager**安全管理器进行认证授权
## SecurityManager
SecyurityManager即安全管理器，对全部的subject进行安全管理，它是shiro的核心，负责对所有的subject进行安全管理。通过SecurityManager可以完成subject的认证、授权等，实质上SecurityManager是通过Authenticator进行认证，通过Authorizer进行授权，通过SessionManager进行会话管理
**SecurityManager**是一个接口，继承了Authenticator，Authorizer，SessionManager这三个接口
## Authenticator
Authenticator即认证器，对用户身份进行认证，Authenticator是一个接口，shiro提供ModularRealmAuthenticator实现类，通过ModularRealmAuthenticator基本上可以满足大多数需求,也可以自定义认证器
## Authorizer
Authorizer即授权器，用户通过认证器认证通过，在访问功能时需要通过授权器判断用户是否有此功能的操作权限
## Realm
Realm即领域，相当于datasource数据源，securityManager进行安全认证需要通过Realm获取用户权限数据，比如：如果用户身份数据在数据库那么Realm就需要从数据库获取用户身份信息
**注：**不要把realm理解成只是从数据源取数据，在realm中还有认证授权校验的相关的代码
## SessionManager
sessionManager即会话管理，shiro框架定义了一套会话管理，它不依赖web容器的[session](https://so.csdn.net/so/search?q=session&spm=1001.2101.3001.7020)，所以shiro可以使用在非web应用上，也可以将分布式应用的会话集中在一点管理，此特性可使它实现单点登录
## SessionDAO
SessionDAO即会话dao，是对session会话操作的一套接口，比如要将session存储到数据库，可以通过jdbc将会话存储到数据库
## CacheManager
CacheManager即缓存管理，将用户权限数据存储在缓存，这样可以提高性能
### Cryptography
Cryptography即密码管理，shiro提供了一套加密/解密的组件，方便开发。比如提供常用的散列、加/解密等功能。
# Shiro中的认证
身份认证，就是判断一个用户是否为合法用户的处理过程。最常用的简单身份认证方式是系统通过核对用户输入的用户名和口令，看其是否与系统中存储的该用户的用户名和口令一致，来判断用户身份是否正确
## Subject
访问系统的用户，主体可以是用户、程序等，进行认证的都称为主体
## Principal
身份信息，是主体（subject）进行身份认证的标识，表示必须具有唯一性，如用户名、手机号、邮箱地址等，一个主体可以有多个身份，但是必须有一个主身份(Primary Principal)
## Credential
凭证信息，是只有主体自己知道的安全信息，如密码、证书等
# Shiro自定义认证
用户的认证是在 SimpleAccountRealm 的 doGetAuthenticationInfo 的方法中完成的，而 SimpleAccountRealm 继承自 AuthorizingRealm, 而在 AuthorizingRealm 中有一个抽象方法 doGetAuthenticationInfo自定义认证的时候可以自定义一个realm继承自 AuthorizingRealm 来复写 doGetAuthenticationInfo，在这个方法里面实现认证逻辑,  AuthorizingRealm是继承自 AuthenticatingRealm有个实现用户授权的方法 doGetAuthorizationInfo 
**注：通过继承 **AuthorizingRealm 可重写授权方法 doGetAuthorizationInfo 和认证方法 doGetAuthenticationInfo
![image.png](https://cdn.nlark.com/yuque/0/2022/png/22980297/1647049159104-f4b46313-801d-4a4c-a764-122c3445ff5d.png#clientId=ubc6f0d2f-a17e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=530&id=u5d4ce71c&name=image.png&originHeight=707&originWidth=1496&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1353300&status=done&style=shadow&taskId=ubbdbd998-3094-4fb1-918e-0efabb0a591&title=&width=1122)
### 测试代码
![image.png](https://cdn.nlark.com/yuque/0/2022/png/22980297/1647049226959-6d3099c5-1b83-4182-9a28-1c57b274c216.png#clientId=ubc6f0d2f-a17e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=534&id=u0f988fdb&name=image.png&originHeight=712&originWidth=1098&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1089854&status=done&style=shadow&taskId=u7a6df634-d3d2-4ee6-84ec-90a4cc18f27&title=&width=824)

# Shiro的加密和随机盐
## Shrio中密码的加密策略
实际应用中用户的密码并不是明文存储在数据库中的，而是采用一种加密算法将密码加密后存储在数据库中。Shiro中提供了一整套的加密算法，并且提供了随机盐。Shiro使用指定的加密算法将用户密码和随机盐进行加密，并按照指定的散列次数将散列后的密码存储在数据库中。
### Shiro中的加密方式
![image.png](https://cdn.nlark.com/yuque/0/2022/png/22980297/1647051927319-45f6c62e-710e-425e-97fb-27be137989c8.png#clientId=ubc6f0d2f-a17e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=435&id=u849b220e&name=image.png&originHeight=580&originWidth=1040&originalType=binary&ratio=1&rotation=0&showTitle=false&size=822766&status=done&style=shadow&taskId=uaa4c519f-2b2c-4bb5-a4c4-b086aaaf436&title=&width=780)
### 自定义Realm加密
![image.png](https://cdn.nlark.com/yuque/0/2022/png/22980297/1647051992127-cbc8c88a-8e7d-4baf-9ab0-708868d0a3bb.png#clientId=ubc6f0d2f-a17e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=569&id=u50e4657f&name=image.png&originHeight=758&originWidth=1337&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1300314&status=done&style=shadow&taskId=uf3ffb5aa-f046-449a-a1d9-7def7cc02f4&title=&width=1003)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/22980297/1647052792230-cb50c656-706f-4815-aa78-59610ea4a780.png#clientId=ubc6f0d2f-a17e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=627&id=ubbd81b82&name=image.png&originHeight=835&originWidth=1150&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1277323&status=done&style=shadow&taskId=u0a0cc914-b634-4c89-82da-0761f907c09&title=&width=863)














