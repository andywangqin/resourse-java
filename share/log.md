###1、简介
####1.1、java.util.logging
由Sun提供的JDK内的写日志的方案一直无法全面推广。造成这种情况的原因当然是其缺乏可配置性和灵活性。JDK的日志方案对于比较简单的项目来讲当然是一种解决办法，但对于企业级的应用来讲就不然了。

####1.2.log4j vs logback
现在，除了log4j之外，另有一种新的比log4j更强大、更快和更灵活的实现已经上市了：logback。好吧，实际上logback是始于2006年的，但其版本1.0在2011年11月份才发布。
logback开发出来就是为了替代log4j的，它和log4j都是出自同一个开发者。版本1.0经过多年的测试和开发现已可供使用了（最新版本是1.0.1）。为了避免由于其版本号这么小而造成误解，应该指出的是，logback已经在业界使用多年了，总之其版本号绝不是反映其稳定性和功能性方面的声明。
logback同log4j相比具有众多优势。下面列出一部分：
- 更快的实现
- 自动重新装载日志配置文件
- 更好的过滤器（filter）
- 自动压缩归档的日志文件
- 堆栈跟踪里包括了Java包（jar文件）的信息
- 自动删除旧日志归档文件

####1.3、slf4j
slf4j是The Simple Logging Facade for Java的简称，是一个简单日志门面抽象框架，它本身只提供了日志Facade API和一个简单的日志类实现，一般常配合Log4j，LogBack，java.util.logging使用。Slf4j作为应用层的Log接入时，程序可以根据实际应用场景动态调整底层的日志实现框架(Log4j/LogBack/JdkLog...)；
建议配合Slf4j使用，这样可以灵活地替换底层日志框架。

###2、logback
####2.1、LogBack的结构
LogBack分为3个组件，logback-core, logback-classic 和 logback-access。
其中logback-core提供了LogBack的核心功能，是另外两个组件的基础。
logback-classic则实现了Slf4j的API，所以当想配合Slf4j使用时，则需要引入这个包。
logback-access是为了集成Servlet环境而准备的，可提供HTTP-access的日志接口。

####2.2、Log的行为级别
OFF、
FATAL、
ERROR、
WARN、
INFO、
DEBUG、
ALL
从下向上，当选择了其中一个级别，则该级别向下的行为是不会被打印出来。
举个例子，当选择了INFO级别，则INFO以下的行为则不会被打印出来。

####2.3、appender



####3、slf4j+logback
####3.1、 集成
只要在你的Maven POM中转换一个依赖就算准备好了：
```xml
<dependency> 
   <groupId>ch.qos.logback</groupId> 
   <artifactId>logback-classic</artifactId> 
   <version>1.1.3</version> 
</dependency>
```

####4、企业应用
####4.1、业务日志


####4.2、摘要日志








参考：

-[Reference]()