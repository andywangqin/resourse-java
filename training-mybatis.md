### 1、mybatis
---
#### 1.1、入门
1. 每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为中心的。SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder 获得。而 SqlSessionFactoryBuilder 则可以从 XML 配置文件或一个预先定制的 Configuration 的实例构建出 SqlSessionFactory 的实例。
2. 既然有了 SqlSessionFactory ，顾名思义，我们就可以从中获得 SqlSession 的实例了。SqlSession 完全包含了面向数据库执行 SQL 命令所需的所有方法。你可以通过 SqlSession 实例来直接执行已映射的 SQL 语句
3. 线程安全性
依赖注入框架可以创建线程安全的、基于事务的 SqlSession 和映射器（mapper）并将它们直接注入到你的 bean 中，因此可以直接忽略它们的生命周期。如果对如何通过依赖注入框架来使用 MyBatis 感兴趣可以研究一下 MyBatis-Spring 或 MyBatis-Guice 两个子项目。
SqlSessionFactoryBuilder: SqlSessionFactoryBuilder 实例的最佳范围是方法范围（也就是局部方法变量）;SqlSessionFactory:线程安全的类;SqlSession不是线程安全的

#### 1.2、XML 映射配置文件
##### 1.2.1、settings
##### 1.2.2、处理枚举类型
若想映射枚举类型 Enum，则需要从 EnumTypeHandler 或者 EnumOrdinalTypeHandler 中选一个来使用。

##### 1.2.3、Mapper XML 文件

##### 1.2.4、动态 SQL

#### 1.3、mybatis-spring

参考:

- [mybatis](http://www.mybatis.org/mybatis-3/zh/getting-started.html)
- [mybatis-spring](http://www.mybatis.org/spring/zh/)