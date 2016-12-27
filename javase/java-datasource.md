###1、Java数据源简介
---
####1.1、Java数据源类图
[](image/20160318180949963.png)

简单说明:
> CommonDataSource：是对数据源概念的顶层抽象，约束了数据源必需实现的方法。
> 
> 从架构图中可以看出数据源有三种类型，即CommonDataSource抽象有三种实现方式，分别是：
DataSource、XADataSource、ConnectionPoolDataSource，三种类型数据源说明如下：

- DataSource：基础实现，数据库物理连接的工厂，用于创建标准的数据库物理连接Connection，JDBC 2.0中诞生，与早先的DriverManager用途一样，新版本JDBC编程已推荐使用DataSource；
- XADataSource：分布式事务实现，为支持分布式事务而诞生，此数据源直接生产出的不是数据库物理连接Connection，而是一个支持XA的XAConnection对象，
XAConnection对象可以直接生产数据库物理连接Connection，同时生产XAResource用于支持XA事务，通常XAConnection对象生产出的数据库物理连接Connection需要和该XAConnection生产出的XAResource对象配合使用以完成XA事务处理（请参考 XA 和 JTA 规范）；
- ConnectionPoolDataSource：连接池实现，此数据源实现并不直接创建数据库物理连接，而是一个逻辑实现，它的作用在于池化数据库物理连接，由于数据库物理连接是一个重量级的对象，频繁的创建销毁很影响性能，将物理连接池化后可降低创建和销毁的频率，复用连接以充分利用连接资源，此数据源通常不为用户应用所知，通常是由中间件服务方来调度，中间件服务方通过它获取一个池化对象PooledConnection，再通过该PooledConnection间接获取到物理连接，获取方式即是调用其getConnection()方法，然后创建出一个规范定义的Connection接口句柄提供给应用使用，应用通过该句柄间接使用物理连接，在应用调用句柄的close方法时，中间件服务方的实现并没有真正调用物理连接的关闭，而是将其归还到连接池中。
另外，从类结构上可以看到XAConnection继承自PooledConnection，本身即是一个受管连接，可能设计之初也是考虑XA连接的池化机制了吧，也许是想管理普通Connection的池化对象PooledConnection可以兼容管理XAConnection（继承关系嘛）

####1.2、XADataSource（2PC）
XADataSource 通过四步Start, End, Prepare, Commit 保证多个数据源的事务完整性. 它主要额外提供了Prepare这个过程. 那么提供这个Prepare过程有什么用呢？这个过程有两个很重要的步骤, 1. 询问每个DataSource 你准备好了Commit吗？ 在这个期间，如果有任何一个Datasource返回No，所有事务回滚. 2. 向系统内写日志.(这一步对于在Commit的时候产生异常进行灾难性的恢复相当关键)。这里需要强调的是，对于XADataSource，JTA的事务管理范围包括 Start, End, Prepare 这三个步骤.

如果我们有2个数据源数据源A和数据源B, 来讨论如下几种事务失败的情况.
1. 在 Start <--> End(Java Transaction boundary) 之间, 凡是有任何异常，均可以正常回滚.
2. 在 Prepare 的时候出错, Java 的业务逻辑 处理完成, XADataSource现在会询问所有的 DataSource "are you ready to commit", 如果任何一个DataSource说"No", 所有DataSource的事务全部回滚.
在 项目开发的时候，遇到过一个很有趣的例子,  两个数据源 远程MQ Server + 本地DB2 DataBase, 用的atomikos JTA.  这里，我故意把atomikos的XADataSource 换成了Hibernate 的DataSource，写了一段很简单的程序，先执行Database 的存取操作，然后将相关数据发送给MQ，我故意在程序执行的最后(随便写了段System.out.println()语句) 处打了个断点，然后用Debug模式执行到该断点处，然后我扒开网线，这个时候实际上MQ已经失效了，然后让程序继续执行，最后MQ事务显然失败，但是数据库的事务却提交了. 这里的问题很明显，首先第一，我没用XADataSource,而用的Hibernate的DataSource，所以不会有Prepare这一步。第二，这个例子中，Start <--> END 已经执行完成，对于不是XADataSource的 JTA会直接到 Commit。而这步实际上就是在Commit 的时候出的错. 在JTA的范围之外了，所以整个事务回滚不了.  如果我们将Hibernate的DataSource换成atomikos的XADataSource，然后用相同的操作执行上面的步骤，MQ和DB2事务都失败. 这正是因为在Commit之前会有一步Prepare的过程，XADataSource会询问 MQ Server 和 DB2 DataSource 你们准备好了吗？而此时，MQ会说NO，所以整个事务回滚。这里，我们可以看到XADataSource 的必要性。
3. 在Commit 的时候出错. 这步就是灾难性的一步了，也是XADataSource最棘手的地方了。试想，JTA执行过程中一切成功，并且在Prepare过程的时候，所有的数据源都回答OK。最后某个DataSource在Commit自己的数据的时候不幸出错. 那么此时该如何解决呢？ 这里普片的做法就是在Prepare的时候，让XADataSource写日记记录数据提交的详细记录。那么是如何操作的呢? XADatasource会针对每个 XADataSource实例记录一个编号, 在Prepare的时候会针对这个XADataSource实例写日志，并且保留Commit失败时候的日志，如果Commit的时候发生异常 （我们知道，这步已经是JTA事务管理之外的了，换句话说，这里是XADataSource 的JTA 已经不可能回滚的了），之后 atmokios 根据该失败的XADataSource实例日志 会定期检查对应处理失败的数据源是否恢复正常，如果恢复正常，那么它会根据日志将进行再次提交，直到成功为止。 但是现在并不是任何 XADataSource 的 Provider 都提供了Prepare()过程写日志的功能，有些 仅仅是商业版本的才会提供。或许是因为这步是最棘手也是最难实现的的部分吧.

###2、dbcp
---
Tomcat 在 7.0 以前的版本都是使用 commons-dbcp 做为连接池的实现，但是 dbcp 饱受诟病，原因有：

- dbcp 是单线程的，为了保证线程安全会锁整个连接池
- dbcp 性能不佳
- dbcp 太复杂，超过 60 个类
- dbcp 使用静态接口，在 JDK 1.6 编译有问题
- dbcp 发展滞后

###3、Tomcat jdbc pool
---
Tomcat 从 7.0 开始引入一个新的模块：Tomcat jdbc pool

- tomcat jdbc pool 近乎兼容 dbcp ，性能更高
- 异步方式获取连接
- tomcat jdbc pool 是 tomcat 的一个模块，基于 tomcat JULI，使用 Tomcat 的日志框架
- 使用 javax.sql.PooledConnection 接口获取连接
- 支持高并发应用环境
- 超简单，核心文件只有8个，比 c3p0 还
- 更好的空闲连接处理机制
- 支持 JMX
- 支持 XA Connection

异步获取连接的方法：
```
Connection con = null;
try {
  Future<Connection> future = datasource.getConnectionAsync();
  while (!future.isDone()) {
      System.out.println("Connection is not yet available. Do some background work");
      try {
      Thread.sleep(100); //simulate work
      }catch (InterruptedException x) {
      Thread.currentThread().interrupted();
      }
  }
  con = future.get(); //should return instantly
  Statement st = con.createStatement();
  ResultSet rs = st.executeQuery("select * from user");
```
