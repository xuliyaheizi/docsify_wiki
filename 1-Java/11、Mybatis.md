Mybatis就是一个持久层的的框架。对JDBC做了轻量级封装

它是基于Java编写的持久层框架，使开发者不必关心传统jdbc的api，只关心sql语句本身。

mybatis 通过 xml或注解的方式将要执行的各种 statement 配置起来，并通过 java 对象和 statement 中 sql的动态参数进行映射生成最终执行的 sql 语句，最后由 mybatis 框架执行 sql 并将结果映射为 java 对象并返回。采用 ORM 思想解决了实体和数据库映射的问题，对 jdbc 进行了封装，屏蔽了 jdbc api 底层访问细节，使我们不用与 jdbc api 打交道，就可以完成对数据库的持久化操作。

ORM思想对应的框架有：mybatis（半自动），hibernate（全自动），spring data jpa（全自动，底层封装的就是hibernate）
![image.png](https://cdn.nlark.com/yuque/0/2022/png/22980297/1647693188862-fdcaa1e4-55bc-4ce0-853f-f3ec4c1e2c08.png#clientId=u74c954c6-ddf8-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u7d6a717c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=397&originWidth=709&originalType=url&ratio=1&rotation=0&showTitle=false&size=144233&status=done&style=none&taskId=u1b0f6877-9615-4ab8-a0f1-b387744ef0a&title=)
明确：它是一个持久层框架，解决项目对数据库的CRUD操作，只要会方法名、sql语句，学mybatis框架很轻松。

JDBC缺陷总结

1：数据库链接创建、释放频繁造成系统资源浪费从而影响系统性能，如果使用数据库连接池可解决此问题。

2：Sql 语句在代码中硬编码，造成代码不易维护，实际应用 sql 变化的可能较大， sql 变动需要改变java 代码。

3：使用 preparedStatement 向占有位符号传参数存在硬编码，因为 sql 语句的 where 条件不一定，可能多也可能少，修改 sql 还要修改代码，系统不易维护。

4：对结果集解析存在硬编码（查询列名）， sql 变化导致解析代码变化，系统不易维护，如果能将数据库记录封装成 pojo 对象解析比较方便。（通过映射，让查询的结果集直接封装到实体类的属性中）

传统模式接口实现类进行开发和代理模式它们做的事情是一样的，区别是，mybatis通过代理模式将我们传统模式要做的实现类操作封装起来了，我们就不需要去实现dao接口。
