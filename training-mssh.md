### 1、maven
---
#### 1.1、maven安装
略

#### 1.2、POM基础
##### 1.2.1、Simple POM
![](\\image\\QQ20160906113324.png)

|节点 |	描述 |
|groupId	|这是工程组的标识。它在一个组织或者项目中通常是唯一的。例如，一个银行组织 com.company.bank 拥有所有的和银行相关的项目。|
|artifactId	|这是工程的标识。它通常是工程的名称。例如，消费者银行。groupId 和 artifactId 一起定义了 artifact 在仓库中的位置。|
|version	|这是工程的版本号。在 artifact 的仓库中，它用来区分不同的版本。例如：
com.company.bank:consumer-banking:1.0
com.company.bank:consumer-banking:1.1.|
|packaging	||
|name	||
|url	||

##### 1.2.2、Super POM
所有的 POM 都继承自一个父 POM（无论是否显式定义了这个父 POM）。父 POM 也被称作 Super POM，它包含了一些可以被继承的默认设置。

effective pom:Super pom 加上工程自己的配置,命令：mvn help:effective-pom

#### 1.3、构建生命周期（Build Lifecycle）
##### 1.3.1、概念:构建生命周期（Build Lifecycle）、阶段（Phase）和目标（Goal）
其中生命周期由多个有序的构建阶段组成，一个构建阶段可以绑定一个或者多个的目标。构建生命周期和阶段只是抽象的概念，不涉及具体的功能。 具体的功能由插件（Plugin）实现。一个插件可以实现多个目标。
而maven默认将某些目标自动绑定到某些阶段，只要进行了绑定，执行阶段时，自动就会执行该目标。

例如：
mvn package
手擀面这条命令执行的是default生命周期中的package阶段，而这个阶段默认绑定了maven-jar-plugin的jar目标（即jar:jar），所以执行这个阶段就会调用该插件jar目标所做的事情，也就是创建项目的jar包。

当然也可以直接执行目标，比如：
mvn dependency:copy-dependencies
这里dependency是插件的名称，copy-denpendencies是插件的目标。
如果需要将某个目标绑定到某个阶段，在POM文件中配置插件即可。

Maven 有以下三个标准的生命周期

- clean: 主要目的是清理项目
- default(or build)：定义了真正构建时所需要执行的所有步骤，它是生命周期中最核心的部分
- site: 生成项目站点文档

##### 1.3.2、Clean 生命周期

##### 1.3.3、Default (or Build) 生命周期
|阶段	|处理	|描述|
|prepare-resources|	资源拷贝	|本阶段可以自定义需要拷贝的资源|
|compile	|编译	|本阶段完成源代码编译|
|package	|打包	|本阶段根据 pom.xml 中描述的打包配置创建 JAR / WAR 包|
|install	|安装	|本阶段在本地 / 远程仓库中安装工程包|

##### 1.3.4、Site 生命周期

#### 1.4、dependencies 
##### 1.4.1、仓库
Maven 仓库有三种类型：

- 本地（local）
- 中央（central）
- 远程（remote）

##### 1.4.2、依赖管理
Maven 提供一些功能来控制可传递的依赖的程度。

|功能	|功能描述|
|依赖调节	|决定当多个手动创建的版本同时出现时，哪个依赖版本将会被使用。 如果两个依赖版本在依赖树里的深度是一样的时候，第一个被声明的依赖将会被使用。|
|依赖管理	|直接的指定手动创建的某个版本被使用。例如当一个工程 C 在自己的以来管理模块包含工程 B，即 B 依赖于 A， 那么A 即可指定在 B 被引用时所使用的版本。|
|依赖范围	|包含在构建过程每个阶段的依赖。|
|依赖排除	|任何可传递的依赖都可以通过 "exclusion" 元素被排除在外。举例说明，A 依赖 B， B 依赖 C，因此 A 可以标记 C 为 “被排除的”。|
|依赖可选	|任何可传递的依赖可以被标记为可选的，通过使用 "optional" 元素。例如：A 依赖 B， B 依赖 C。因此，B 可以标记 C 为可选的， 这样 A 就可以不再使用 C。|

##### 1.4.3、外部依赖
通常情况下它会包含一些任何仓库无法使用，并且 maven 也无法下载的 jar 文件。如果你的代码正在使用这个库，那么 Maven 的构建过程将会失败，因为在编译阶段它不能下载或者引用这个库。

##### 1.4.4、快照（snapshot）
快照是一个特殊的版本，它表示当前开发的一个副本。与常规版本不同，Maven 为每一次构建从远程仓库中检出一份新的快照版本。

#### 1.5、plugin
mvn [plugin-name]:[goal-name]

#### 1.6、build
##### 1.6.1、build


##### 1.6.2、构建自动化
在 bus-core-api 的 pom 文件里添加一个编译目标来提醒 app-web-ui 工程和 app-desktop-ui 工程启动创建。
使用一个持续集成（CI）的服务器，比如 Hudson，来实现自动化创建。

##### 1.6.3、自动化部署

一般情况下，在一个工程开发进程里，一次部署的过程包含需如下步骤：

- 合入每个子工程下的代码到 SVN 或者源代码库，并标记它。
- 从 SVN 下载完整的源代码。
- 构建应用程序。
- 保存构建结果为 WAR 或者 EAR 类型文件并存放到一个共同的指定的网络位置上。
- 从网络上获得该文件并且部署该文件到产品线上。
- 更新文档日期和应用程序的版本号。

#### 1.7、profile
构建配置文件是一组配置的集合，用来设置或者覆盖 Maven 构建的默认配置,使用构建配置文件，可以为不同的环境定制构建过程，例如 Producation 和 Development 环境，在profile里几乎可以定义所有在pom里的定义的内容（<dependencies>，<properties>，插件配置等等，不过不能再定义他自己了）。当一个profile被激活时，它定义的<dependencies>，<properties>等就会覆盖掉原pom里定义的相同内容，从而可以通过激活不同的profile来使用不同的配置。

#### 1.9、demo
##### 1.9.1、create project
Maven 使用原型（archetype）插件创建web工程，执行:

mvn archetype:generate -DgroupId=com.qianmo.demo -DartifactId=demo-server -Dversion=1.0.0-SNAPSHOT -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false

##### 1.9.2、import maven project

1. 通过idea new菜单，import maven项目
2. idea 设置
Import Maven projects automatically; add java package
3. 集成tomcat

参考:
[Maven教程](http://wiki.jikexueyuan.com/project/maven/repositories.html)

[POM Reference](https://maven.apache.org/pom.html#Build_Settings)




### 2、spring
---
#### 2.1、spring核心
- 依赖注入（DI）
- 面向方面的程序设计（AOP）

#### 2.2、IoC 容器
- BeanFactory 容器
- ApplicationContext 容器


在资源宝贵的移动设备或者基于 applet 的应用当中， BeanFactory 会被优先选择。否则，一般使用的是 ApplicationContext，除非你有更好的理由选择 BeanFactory。

#### 2.3、Bean
被称作 bean 的对象是构成应用程序的支柱也是由 Spring IoC 容器管理的。bean 是一个被实例化，组装，并通过 Spring IoC 容器所管理的对象

Bean 的作用域

Bean 的生命周期

Bean 后置处理器


#### 2.6、Spring mvc


#### 2.9、demo


参考:
[Spring教程](http://wiki.jikexueyuan.com/project/spring/hello-world-example.html)


### 3、mybatis
---
#### 3.1、入门
1. 每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为中心的。SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder 获得。而 SqlSessionFactoryBuilder 则可以从 XML 配置文件或一个预先定制的 Configuration 的实例构建出 SqlSessionFactory 的实例。
2. 既然有了 SqlSessionFactory ，顾名思义，我们就可以从中获得 SqlSession 的实例了。SqlSession 完全包含了面向数据库执行 SQL 命令所需的所有方法。你可以通过 SqlSession 实例来直接执行已映射的 SQL 语句
3. 线程安全性
依赖注入框架可以创建线程安全的、基于事务的 SqlSession 和映射器（mapper）并将它们直接注入到你的 bean 中，因此可以直接忽略它们的生命周期。如果对如何通过依赖注入框架来使用 MyBatis 感兴趣可以研究一下 MyBatis-Spring 或 MyBatis-Guice 两个子项目。
SqlSessionFactoryBuilder: SqlSessionFactoryBuilder 实例的最佳范围是方法范围（也就是局部方法变量）;SqlSessionFactory:线程安全的类;SqlSession不是线程安全的

#### 3.2、XML 映射配置文件
##### 3.2.1、settings
##### 3.2.2、处理枚举类型
若想映射枚举类型 Enum，则需要从 EnumTypeHandler 或者 EnumOrdinalTypeHandler 中选一个来使用。

##### 3.2.3、Mapper XML 文件

##### 3.2.4、动态 SQL

#### 3.3、mybatis-spring




参考:
[mybatis](http://www.mybatis.org/mybatis-3/zh/getting-started.html)

[mybatis-spring](http://www.mybatis.org/spring/zh/)

开发环境通过普通的DataSource连接数据库：
测试和产品环境都是通过 JNDI 方式连接数据库：