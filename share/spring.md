### 1、spring
---
#### 1.1、spring体系结构
- 依赖注入（DI）: 控制反转（IoC）是一个通用的概念，它可以用许多不同的方式去表达，依赖注入仅仅是控制反转的一个具体的例子。
- 面向切面的程序设计（AOP）

![](../image/arch1.png)

#### 1.2、IoC 容器
Spring 容器是 Spring 框架的核心。容器将创建对象，把它们连接在一起，配置它们，并管理他们的整个生命周期从创建到销毁。Spring 提供了以下两种不同类型的容器：

|序号	|容器 | 描述|
| ------------- |:-------------:|-------------:|
|1	|Spring BeanFactory 容器|它是最简单的容器，给 DI 提供了基本的支持，它用 org.springframework.beans.factory.BeanFactory 接口来定义。BeanFactory 或者相关的接口，如 BeanFactoryAware，InitializingBean，DisposableBean，在 Spring 中仍然存在具有大量的与 Spring 整合的第三方框架的反向兼容性的目的。|
|2	|Spring ApplicationContext 容器|该容器添加了更多的企业特定的功能，例如从一个属性文件中解析文本信息的能力，发布应用程序事件给感兴趣的事件监听器的能力。该容器是由 org.springframework.context.ApplicationContext 接口定义。|

ApplicationContext 包含 BeanFactory 所有的功能，一般情况下，相对于 BeanFactory，ApplicationContext 会被推荐使用。BeanFactory 仍然可以在轻量级应用中使用，比如移动设备或者基于 applet 的应用程序。

#### 1.3、Bean
##### 1.3.1、Bean 定义
bean 是一个被实例化，组装，并通过 Spring IoC 容器所管理的对象

|属性	|描述|
| ------------- |:-------------:|
|class	|这个属性是强制性的，并且指定用来创建 bean 的 bean 类。|
|name	|这个属性指定唯一的 bean 标识符。在基于 XML 的配置元数据中，你可以使用 ID 和/或 name 属性来指定 bean 标识符。|
|scope	|这个属性指定由特定的 bean 定义创建的对象的作用域，它将会在 bean 作用域的章节中进行讨论。|
|constructor-arg|	它是用来注入依赖关系的，并会在接下来的章节中进行讨论。|
|properties	|它是用来注入依赖关系的，并会在接下来的章节中进行讨论。|
|autowiring mode	|它是用来注入依赖关系的，并会在接下来的章节中进行讨论。|
|lazy-initialization mode	|延迟初始化的 bean 告诉 IoC 容器在它第一次被请求时，而不是在启动时去创建一个 bean 实例。|
|initialization 方法	|在 bean 的所有必需的属性被容器设置之后，调用回调方法。它将会在 bean 的生命周期章节中进行讨论。|
|destruction 方法	|当包含该 bean 的容器被销毁时，使用回调方法。它将会在 bean 的生命周期章节中进行讨论。|

##### 1.3.2、Bean 的作用域

| 作用域| 	描述| 
| ------------- |:-------------:|
|singleton	|该作用域将 bean 的定义的限制在每一个 Spring IoC 容器中的一个单一实例(默认)。|
|prototype	|该作用域将单一 bean 的定义限制在任意数量的对象实例。|
|request	|该作用域将 bean 的定义限制为 HTTP 请求。只在web-aware Spring ApplicationContext 的上下文中有效。|
|session	|该作用域将 bean 的定义限制为 HTTP 会话。 只在web-aware Spring ApplicationContext的上下文中有效。|
|global-session	|该作用域将 bean 的定义限制为全局 HTTP 会话。只在 web-aware Spring ApplicationContext 的上下文中有效。|

一般说来，满状态的 bean 使用 prototype 作用域和没有状态的 bean 使用 singleton 作用域。

##### 1.3.3、Bean 的生命周期
本章将只讨论两个重要的生命周期回调方法，它们在 bean 的初始化和销毁的时候是必需的。
为了定义安装和拆卸一个 bean，我们只要声明带有 init-method 和/或 destroy-method 参数的 。init-method 属性指定一个方法，在实例化 bean 时，立即调用该方法。同样，destroy-method 指定一个方法，只有从容器中移除 bean 之后，才能调用该方法。

##### 1.3.4、Bean 后置处理器
BeanPostProcessor 接口定义回调方法，你可以实现该方法来提供自己的实例化逻辑，依赖解析逻辑等。你也可以在 Spring 容器通过插入一个或多个 BeanPostProcessor 的实现来完成实例化，配置和初始化一个bean之后实现一些自定义逻辑回调方法。

#### 1.4、依赖注入
##### 1.4.1、xml装配
|序号	|依赖注入类型 & 描述|
| ------------- |:-------------:|
|1	Constructor-based dependency injection|当容器调用带有多个参数的构造函数类时，实现基于构造函数的 DI，每个代表在其他类中的一个依赖关系。|
|2	Setter-based dependency injection|基于 setter 方法的 DI 是通过在调用无参数的构造函数或无参数的静态工厂方法实例化 bean 之后容器调用 beans 的 setter 方法来实现的。|

- 注入内部 Beans
 
```xml
<!-- Definition for textEditor bean using inner bean -->
   <bean id="textEditor" class="com.tutorialspoint.TextEditor">
      <property name="spellChecker">
         <bean id="spellChecker" class="com.tutorialspoint.SpellChecker"/>
       </property>
   </bean>
```

- 注入集合

|元素	|描述|
| ------------- |:-------------:|
|<list>	|它有助于连线，如注入一列值，允许重复。|
|<set>	|它有助于连线一组值，但不能重复。|
|<map>	|它可以用来注入名称-值对的集合，其中名称和值可以是任何类型。|
|<props>	|它可以用来注入名称-值对的集合，其中名称和值都是字符串类型。|

##### 1.4.2、Beans 自动装配

|模式	|描述|
| ------------- |:-------------:|
|no	|这是默认的设置，它意味着没有自动装配，你应该使用显式的bean引用来连线。你不用为了连线做特殊的事。在依赖注入章节你已经看到这个了。|
|byName	|由属性名自动装配。Spring 容器看到在 XML 配置文件中 bean 的自动装配的属性设置为 byName。然后尝试匹配，并且将它的属性与在配置文件中被定义为相同名称的 beans 的属性进行连接。|
|byType	|由属性数据类型自动装配。Spring 容器看到在 XML 配置文件中 bean 的自动装配的属性设置为 byType。然后如果它的类型匹配配置文件中的一个确切的 bean 名称，它将尝试匹配和连接属性的类型。如果存在不止一个这样的 bean，则一个致命的异常将会被抛出。|
|constructor	|类似于 byType，但该类型适用于构造函数参数类型。如果在容器中没有一个构造函数参数类型的 bean，则一个致命错误将会发生。|
|autodetect|	Spring首先尝试通过 constructor 使用自动装配来连接，如果它不执行，Spring 尝试通过 byType 来自动装配。|

##### 1.4.3、基于注解的配置

|序号	|注解 & 描述|
| ------------- |:-------------:|
|1	|@Required,@Required 注解应用于 bean 属性的 setter 方法。|
|2	|@Autowired,@Autowired 注解可以应用到 bean 属性的 setter 方法，非 setter 方法，构造函数和属性。|
|3	|@Qualifier,通过指定确切的将被连线的 bean，@Autowired 和 @Qualifier 注解可以用来删除混乱。|
|4	|JSR-250 Annotations,Spring 支持 JSR-250 的基础的注解，其中包括了 @Resource，@PostConstruct 和 @PreDestroy 注解。|

##### 1.4.4、Spring组件扫描
```xml
<context:component-scan base-package="com.xhlx.finance.budget" > 
     <context:include-filter type="regex" expression=".service.*"/> 
</context:component-scan>
```
@Component是所有受Spring管理组件的通用形式；而@Repository、@Service和 @Controller则是@Component的细化，用来表示更具体的用例(例如，分别对应了持久化层、服务层和表现层)。也就是说，你能用@Component来注解你的组件类，但如果用@Repository、@Service 或@Controller来注解它们，你的类也许能更好地被工具处理，或与切面进行关联。

##### 1.4.5、基于 Java 的配置
@Configuration 和 @Bean 注解
带有 @Configuration 的注解类表示这个类可以使用 Spring IoC 容器作为 bean 定义的来源。@Bean 注解告诉 Spring，一个带有 @Bean 的注解方法将返回一个对象，该对象应该被注册为在 Spring 应用程序上下文中的 bean。

#### 1.5、Spring 中的事件处理
Spring 提供了以下的标准事件：

|序号|	Spring 内置事件 & 描述|
| ------------- |:-------------:|
|1	|ContextRefreshedEvent,ApplicationContext 被初始化或刷新时，该事件被发布。这也可以在 ConfigurableApplicationContext 接口中使用 refresh() 方法来发生。|
|2	|ContextStartedEvent,当使用 ConfigurableApplicationContext 接口中的 start() 方法启动 ApplicationContext 时，该事件被发布。你可以调查你的数据库，或者你可以在接受到这个事件后重启任何停止的应用程序。|
|3	|ContextStoppedEvent,当使用 ConfigurableApplicationContext 接口中的 stop() 方法停止 ApplicationContext 时，发布这个事件。你可以在接受到这个事件后做必要的清理的工作。|
|4	|ContextClosedEvent,当使用 ConfigurableApplicationContext 接口中的 close() 方法关闭 ApplicationContext 时，该事件被发布。一个已关闭的上下文到达生命周期末端；它不能被刷新或重启。|
|5	|RequestHandledEvent,这是一个 web-specific 事件，告诉所有 bean HTTP 请求已经被服务。|

由于 Spring 的事件处理是单线程的，所以如果一个事件被发布，直至并且除非所有的接收者得到的该消息，该进程被阻塞并且流程将不会继续。因此，如果事件处理被使用，在设计应用程序时应注意。

Spring 中可自定义事件：扩展ApplicationEvent和ApplicationListener

#### 1.6、AOP
Spring AOP 模块提供拦截器来拦截一个应用程序，例如，当执行一个方法时，你可以在方法执行之前或之后添加额外的功能。AOP 术语:

|项	|描述|
| ------------- |:-------------:|
|Join point（连接点）|是程序执行中的一个精确执行点，例如类中的一个方法。它是一个抽象的概念，在实现AOP时，并不需要去定义一个join point。|
|Point cut（切入点）|本质上是一个捕获连接点的结构。在AOP中，可以定义一个point cut，来捕获相关方法的调用。|
|Advice（通知）|是point cut的执行代码，是执行“方面”的具体逻辑。|
|Aspect（切面）|point cut和advice结合起来就是aspect，它类似于OOP中定义的一个类，但它代表的更多是对象间横向的关系。|
|Introduction（引入）|为对象引入附加的方法或属性，从而达到修改对象结构的目的。有的AOP工具又将其称为mixin。|
|Target object(目标对象)|被一个或者多个切面所通知的对象，这个对象永远是一个被代理对象。也称为被通知对象。|
|Weaving（织入）|Weaving 把切面连接到其它的应用程序类型或者对象上，并创建一个被通知的对象的过程。这些可以在编译时，类加载时和运行时完成。|

Spring 切面可以使用下面提到的五种通知工作：

|项	|描述|
| ------------- |:-------------:|
|前置通知	|在一个方法执行之前，执行通知。|
|后置通知	|在一个方法执行之后，不考虑其结果，执行通知。|
|返回后通知	|在一个方法执行之后，只有在方法成功完成时，才能执行通知。|
|抛出异常后通知	|在一个方法执行之后，只有在方法退出抛出异常时，才能执行通知。|
|环绕通知	|在建议方法调用之前和之后，执行通知。|

##### 1.6.1、XML Schema based
基于 AOP 的 XML 架构的示例:
- 1.这里是 Logging.java 文件的内容。这实际上是 aspect 模块的一个示例，它定义了在各个点调用的方法。
```
package com.tutorialspoint;
public class Logging {
   /** 
    * This is the method which I would like to execute
    * before a selected method execution.
    */
   public void beforeAdvice(){
      System.out.println("Going to setup student profile.");
   }
   /** 
    * This is the method which I would like to execute
    * after a selected method execution.
    */
   public void afterAdvice(){
      System.out.println("Student profile has been setup.");
   }
   /** 
    * This is the method which I would like to execute
    * when any method returns.
    */
   public void afterReturningAdvice(Object retVal){
      System.out.println("Returning:" + retVal.toString() );
   }
   /**
    * This is the method which I would like to execute
    * if there is an exception raised.
    */
   public void AfterThrowingAdvice(IllegalArgumentException ex){
      System.out.println("There has been an exception: " + ex.toString());   
   }  
}
```
- 2.下面是 Student.java 文件的内容：
```
package com.tutorialspoint;
public class Student {
   private Integer age;
   private String name;
   public void setAge(Integer age) {
      this.age = age;
   }
   public Integer getAge() {
      System.out.println("Age : " + age );
      return age;
   }
   public void setName(String name) {
      this.name = name;
   }
   public String getName() {
      System.out.println("Name : " + name );
      return name;
   }  
   public void printThrowException(){
       System.out.println("Exception raised");
       throw new IllegalArgumentException();
   }
}
```
- 3.下面是 MainApp.java 文件的内容：
```
package com.tutorialspoint;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
   public static void main(String[] args) {
      ApplicationContext context = 
             new ClassPathXmlApplicationContext("Beans.xml");
      Student student = (Student) context.getBean("student");
      student.getName();
      student.getAge();      
      student.printThrowException();
   }
}
```
- 4.下面是配置文件 Beans.xml：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd 
    http://www.springframework.org/schema/aop 
    http://www.springframework.org/schema/aop/spring-aop-3.0.xsd ">

   <aop:config>
      <aop:aspect id="log" ref="logging">
         <aop:pointcut id="selectAll" 
         expression="execution(* com.tutorialspoint.*.*(..))"/>
         <aop:before pointcut-ref="selectAll" method="beforeAdvice"/>
         <aop:after pointcut-ref="selectAll" method="afterAdvice"/>
         <aop:after-returning pointcut-ref="selectAll" 
                              returning="retVal"
                              method="afterReturningAdvice"/>
         <aop:after-throwing pointcut-ref="selectAll" 
                             throwing="ex"
                             method="AfterThrowingAdvice"/>
      </aop:aspect>
   </aop:config>

   <!-- Definition for student bean -->
   <bean id="student" class="com.tutorialspoint.Student">
      <property name="name"  value="Zara" />
      <property name="age"  value="11"/>      
   </bean>

   <!-- Definition for logging aspect -->
   <bean id="logging" class="com.tutorialspoint.Logging"/> 

</beans>
```

##### 1.6.2、@AspectJ based

#### 1.7、ORM
##### 1.7.1、Spring事务管理
1.Spring 支持两种类型的事务管理:

- 编程式事务管理 ：这意味着你在编程的帮助下有管理事务。这给了你极大的灵活性，但却很难维护。
- 声明式事务管理 ：这意味着你从业务代码中分离事务管理。你仅仅使用注释或 XML 配置来管理事务。

声明式事务管理比编程式事务管理更可取，尽管它不如编程式事务管理灵活，但它允许你通过代码控制事务。但作为一种横切关注点，声明式事务管理可以使用 AOP 方法进行模块化。Spring 支持使用 Spring AOP 框架的声明式事务管理。

2.Spring 事务抽象

Spring 事务抽象的关键是由 org.springframework.transaction.PlatformTransactionManager 接口定义，如下所示：
```
public interface PlatformTransactionManager {
   TransactionStatus getTransaction(TransactionDefinition definition);
   throws TransactionException;
   void commit(TransactionStatus status) throws TransactionException;
   void rollback(TransactionStatus status) throws TransactionException;
}
```

| 序号	| 方法 & 描述| 
| ------------- |:-------------:|
| 1	| TransactionStatus getTransaction(TransactionDefinition definition) 根据指定的传播行为，该方法返回当前活动事务或创建一个新的事务。| 
| 2	| void commit(TransactionStatus status) 该方法提交给定的事务和关于它的状态。| 
| 3	| void rollback(TransactionStatus status)该方法执行一个给定事务的回滚。| 

TransactionDefinition 是在 Spring 中事务支持的核心接口，它的定义如下：
```
public interface TransactionDefinition {
   int getPropagationBehavior();
   int getIsolationLevel();
   String getName();
   int getTimeout();
   boolean isReadOnly();
}
```

#### 1.8、Spring中的Profile
由于我们平时在开发中，通常会出现在开发的时候使用一个开发数据库，测试的时候使用一个测试的数据库，而实际部署的时候需要一个数据库。以前的做法是将这些信息写在一个配置文件中，当我把代码部署到测试的环境中，将配置文件改成测试环境；当测试完成，项目需要部署到现网了，又要将配置信息改成现网的，真的好烦。。。而使用了Profile之后，我们就可以分别定义3个配置文件，一个用于开发、一个用户测试、一个用户生产，其分别对应于3个Profile。当在实际运行的时候，只需给定一个参数来激活对应的Profile即可，那么容器就会只加载激活后的配置文件，这样就可以大大省去我们修改配置信息而带来的烦恼。
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd">

    <beans profile="dev,test">
        <context:property-placeholder location="classpath:application.properties" />

        <bean id="dataSource" class="com.jolbox.bonecp.BoneCPDataSource" destroy-method="close">
            <property name="driverClass" value="${db.driver}"/>
            <property name="jdbcUrl" value="${db.url}"/>
            <property name="username" value="${db.username}"/>
            <property name="password" value="${db.password}"/>
            <property name="idleConnectionTestPeriodInMinutes" value="60"/>
            <property name="idleMaxAgeInMinutes" value="240"/>
            <property name="maxConnectionsPerPartition" value="30"/>
            <property name="minConnectionsPerPartition" value="10"/>
            <property name="partitionCount" value="3"/>
            <property name="acquireIncrement" value="5"/>
            <property name="statementsCacheSize" value="100"/>
            <property name="releaseHelperThreads" value="3"/>
        </bean>
    <beans profile="production">
        <context:property-placeholder ignore-resource-not-found="true" location="classpath:application.properties,classpath:application-production.properties" />
        
        <bean id="dataSource" class="org.springframework.jndi.JndiObjectFactoryBean">
            <property name="jndiName" value="${db.jndi}"/>
        </bean>
    </beans>

</beans>
```
以数据库为例，开发环境使用的是直接将配置写在项目的配置文件里面，而生产环境则使用了jndi。

切换profile可以写在web.xml里面：
```xml
<context-param>  
  <param-name>spring.profiles.active</param-name>  
  <param-value>dev</param-value>  
</context-param>
```

#### 1.9、Spring mvc
##### 1.9.1、DispatcherServlet
![](../image/mvc1.png)

下面是对应于 DispatcherServlet 传入 HTTP 请求的事件序列：

- 收到一个 HTTP 请求后，DispatcherServlet 根据 HandlerMapping 来选择并且调用适当的控制器。
- 控制器接受请求，并基于使用的 GET 或 POST 方法来调用适当的 service 方法。Service 方法将设置基于定义的业务逻辑的模型数据，并返回视图名称到 DispatcherServlet 中。
- DispatcherServlet 会从 ViewResolver 获取帮助，为请求检取定义视图。
- 一旦确定视图，DispatcherServlet 将把模型数据传递给视图，最后呈现在浏览器中。

1.  配置DispatcherServlet
```xml
<servlet>
      <servlet-name>HelloWeb</servlet-name>
      <servlet-class>
         org.springframework.web.servlet.DispatcherServlet
      </servlet-class>
      <load-on-startup>1</load-on-startup>
   </servlet>
   <servlet-mapping>
      <servlet-name>HelloWeb</servlet-name>
      <url-pattern>*.jsp</url-pattern>
   </servlet-mapping>
```

2. 配置[servlet-name]-servlet.xml
```xml
<context-param>
   <param-name>contextConfigLocation</param-name>
   <param-value>/WEB-INF/HelloWeb-servlet.xml</param-value>
</context-param>
<listener>
   <listener-class>
      org.springframework.web.context.ContextLoaderListener
   </listener-class>
</listener>
```

3. HelloWeb-servlet.xml
```xml
	<context:component-scan base-package="com.tutorialspoint" />

   <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
      <property name="prefix" value="/WEB-INF/jsp/" />
      <property name="suffix" value=".jsp" />
   </bean>
```

4. 定义控制器
```
@Controller
@RequestMapping("/hello")
public class HelloController{
   @RequestMapping(method = RequestMethod.GET)
   public String printHello(ModelMap model) {
      model.addAttribute("message", "Hello Spring MVC Framework!");
      return "hello";
   }
}
```

5. 创建 JSP 视图
```
<html>
   <head>
   <title>Hello Spring MVC</title>
   </head>
   <body>
   <h2>${message}</h2>
   </body>
</html>
```

参考:

- [Spring教程](http://wiki.jikexueyuan.com/project/spring/hello-world-example.html)
- [Spring Framework Reference Documentation](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/)