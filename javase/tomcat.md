###1、简介
---
Tomcat是JEE开发人员最常用到的开发工具，在Java Web应用的调试开发和实际部署中，我们都可以看到Tomcat的影子。大多数时候，我们可以将Tomcat当做一个黑盒来看待，只需要将编写的Java Web工程进行部署即可，但是，在遇到一些比较复杂难解决的问题时，如果我们了解了Tomcat的内部实现原理将会处理起来更得心应手更快地定位问题。另外，通过学习Tomcat的源码还可以更加深入地了解JEE规范，学习常见的设计模式。本系列的文章，将会介绍Tomcat的核心功能是如何实现的，一方面作为自己学习的总结，另一方面也希望给学习Tomcat的朋友提供一点帮助材料。

###2、server.xml
---
本文首先介绍Tomcat的基本配置，涉及的配置文件就是\conf\server.xml文件。Tomcat本身通过一系列的连接器和内部组件来分别实现网络请求的监听和处理。一个示例性的server.xml如下：
```
<?xml version='1.0' encoding='utf-8'?>
<Server port="8005" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
  <Listener className="org.apache.catalina.core.JasperListener" />
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <GlobalNamingResources>
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>

  <Service name="Catalina">
    <Connector port="8080" protocol="HTTP/1.1" 
               connectionTimeout="20000" 
               redirectPort="8443" />
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
    <Engine name="Catalina" defaultHost="localhost">       
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>

      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"  
               prefix="localhost_access_log." suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" resolveHosts="false"/>
      </Host>
    </Engine>
  </Service>
</Server>
```
从上面的配置我们中，位于配置文件顶层的是Server和Service元素，其中Server元素是整个配置文件的根元素，而Service元素则是配置服务器的核心元素。在Service元素内部，定义了一系列的连接器和内部容器类的组件。现在分别对其进行简单的介绍，后续的文章将会对其进行逐一分析。

<Server>元素对应的是整个Servlet容器，是整个配置的顶层元素，由org.apache.catalina.Server接口来定义，默认的实现类是org.apache.catalina.core. StandardServer。该元素可配置的属性主要是port和shutdown，分别指的是监听shutdown命令的端口和命令（这两个属性没玩过，后续试试）。在该元素中可以定义一个或多个<Service>元素，除此以外还可以定义一些全局的资源或监听器。

<Service>元素由org.apache.catalina.Service接口定义，默认的实现类为org.apache.catalina.core. StandardService。在该元素中可以定义一个<Engine>元素、一个或多个<Connector>元素，这些<Connector>元素共享同一个<Engine>元素来进行请求的处理。

<Engine>元素由org.apache.catalina.Engine元素来定义，默认的实现类是org.apache.catalina.core. StandardEngine。<Engine>元素会用来处理<Service>中所有<Connector>接收到的请求，在<Engine>中可以定义多个<Host>元素作为虚拟主机。<Engine>是Tomcat配置中第一个实现org.apache.catalina.Container的接口，因此可以在其中定义一系列的子元素如<Realm>、<Valve>。

< Connector >元素由org.apache.catalina.connector. Connector类来定义。< Connector>是接受客户端浏览器请求并向用户最终返回响应结果的组件。该元素位于< Service>元素中，可以定义多个，在我们的示例中配置了两个，分别接受AJP请求和HTTP请求，在配置中，需要为其制定服务的协议和端口号。

<Host>元素由org.apache.catalina.Host接口来定义，默认实现为org.apache.catalina.core. StandardHost，该元素定义在<Engine>中，可以定义多个。每个<Host>元素定义了一个虚拟主机，它可以包含一个或多个Web应用（通过<Context>元素来进行定义）。因为<Host>也是容器类元素，所以可以在其中定义子元素如<Realm>、<Valve>。

<Context>元素由org.apache.catalina.Context接口来定义，默认实现类为org.apache.catalina.core. StandardContext。该元素也许是大家用的最多的元素，在其中定义的是Web应用。一个<Host>中可以定义多个<Context>元素，分别对应不同的Web应用。该元素的属性，大家经常会用到如path、reloadable等，可以在<Context>中定义子元素如<Realm>、<Valve>。

以上简单介绍了Tomcat元素的配置，使我们可能对这个庞大的产品有个整体的了解，后续会对每个部分进行详细的介绍，下部分会首先介绍Tomcat的启动流程。


###3、Tomcat的核心组成和启动过程
---
前面的文章中介绍了Tomcat的基本配置，每个配置项也基本上对应了Tomcat的组件结构，如果要用一张图来形象展现一下Tomcat组成的话，整个Tomcat的组成可以如下图所示：

![](image/2012090217005339.png)

Tomcat在接收到用户请求时，将会通过以上组件的协作来给最终用户产生响应。首先是最外层的Server和Service来提供整个运行环境的基础设施，而Connector通过指定的协议和接口来监听用户的请求，在对请求进行必要的处理和解析后将请求的内容传递给对应的容器，经过容器一层层的处理后，生成最终的响应信息，返回给客户端。

Tomcat的容器通过实现一系列的接口，来统一处理一些生命周期相关的操作，而Engine、Host、Context等容器通过实现Container接口来完成处理请求时统一的模式，具体表现为该类容器内部均有一个Pipeline结构，实际的业务处理都是通过在Pipeline上添加Valve来实现，这样就充分保证整个架构的高度可扩展性。Tomcat核心组件的类图如下图所示：

![](image/2012090217013881.jpg)

在介绍请求的处理过程时，将会详细介绍各个组件的作用和处理流程。本文将会主要分析Tomcat的启动流程，介绍涉及到什么组件以及初始化的过程，简单期间将会重点分析HTTP协议所对应Connector启动过程。

Tomcat在启动时的重点功能如下：

初始化类加载器：主要初始化CommonLoader、CatalinaLoader以及SharedLoader；
解析配置文件：使用Digester组件解析Tomcat的server.xml，初始化各个组件（包含各个web应用，解析对应的web.xml进行初始化）；
初始化连接器：初始化声明的Connector，以指定的协议打开端口，等待请求。

不管是通过命令行启动还是通过Eclipse的WST server UI，Tomcat的启动流程是在org.apache.catalina.startup.Bootstrap类的main方法中开始的，在启动时，这个类的核心代码如下所示：
```
public static void main(String args[]) {
        if (daemon == null) {
            daemon = new Bootstrap();//实例化该类的一个实例
            try {
                daemon.init();//进行初始化
            } catch (Throwable t) {
                ……;
            }
        }
        try {
    ……//此处略去代码若干行
    if (command.equals("start")) {
                daemon.setAwait(true);
                daemon.load(args);//执行load，生成组件实例并初始化
                daemon.start();//启动各个组件
            }
    ……//此处略去代码若干行
}
```

从以上的代码中，可以看到在Tomcat启动的时候，执行了三个关键方法即init、load、和start。后面的两个方法都是通过反射调用org.apache.catalina.startup.Catalina的同名方法完成的，所以后面在介绍时将会直接转到Catalina的同名方法。首先分析一下Bootstrap的init方法，在该方法中将会初始化一些全局的系统属性、初始化类加载器、通过反射得到Catalina实例，在这里我们重点看一下初始化类加载器的方法：

```
private void initClassLoaders() {
        try {
            commonLoader = createClassLoader("common", null);
            if( commonLoader == null ) {
                // no config file, default to this loader - we might be in a 'single' env.
                commonLoader=this.getClass().getClassLoader();
            }
            catalinaLoader = createClassLoader("server", commonLoader);
            sharedLoader = createClassLoader("shared", commonLoader);
        } catch (Throwable t) {
            log.error("Class loader creation threw exception", t);
            System.exit(1);
        }
    }
```
在以上的代码中，我们可以看到初始化了三个类加载器，这三个类加载器将会有篇博文进行简单的介绍。

然后我们进入Catalina的load方法：
```
public void load() {
//……
        //初始化Digester组件，定义了解析规则
        Digester digester = createStartDigester();
        //……中间略去代码若干，主要作用为将server.xml文件转换为输入流
        try {
            inputSource.setByteStream(inputStream);
            digester.push(this);
//通过Digester解析这个文件，在此过程中会初始化各个组件实例及其依赖关系
            digester.parse(inputSource);
            inputStream.close();
        } catch (Exception e) {
          
        }
        // 调用Server的initialize方法，初始化各个组件
        if (getServer() instanceof Lifecycle) {
            try {
                getServer().initialize();
            } catch (LifecycleException e) {
                if (Boolean.getBoolean("org.apache.catalina.startup.EXIT_ON_INIT_FAILURE"))
                    throw new java.lang.Error(e);
                else   
                    log.error("Catalina.start", e);
                
            }
        }

}
```
在以上的代码中，关键的任务有两项即使用Digester组件按照给定的规则解析server.xml、调用Server的initialize方法。关于Digester组件的使用，后续会有一篇专门的博文进行讲解，而Server的initialize方法中，会发布事件并调用各个Service的initialize方法，从而级联完成各个组件的初始化。每个组件的初始化都是比较有意思的，但是我们限于篇幅先关注Connector的初始化，这可能是最值得关注的。

Connector的initialize方法，核心代码如下：
```
public void initialize() throws LifecycleException{
     //该适配器会完成请求的真正处理   
adapter = new CoyoteAdapter(this);
    //对于不同的实现，会有不同的ProtocolHandler实现类，我们来看    //Http11Protocol，它用来处理HTTP请求
        protocolHandler.setAdapter(adapter);
        try {
            protocolHandler.init();
        } catch (Exception e) {
            ……
        }
    }
```
在Http11Protocol的init方法中，核心代码如下：
```
public void init() throws Exception {
        endpoint.setName(getName());//endpoint为JIoEndpoint的实现类
        endpoint.setHandler(cHandler);
        try {
            endpoint.init();//核心代码就是调用 JIoEndpoint的初始化方法
        } catch (Exception ex) {
           ……
        }
    }
```
我们看到最终的初始化方法最终都会调到JIoEndpoint的init方法，网络初始化和对请求的最初处理都是通过该类及其内部类完成的，所以后续的内容将会重点关注此类：
```
public void init() throws Exception {
        if (acceptorThreadCount == 0) {//接受请求的线程数
            acceptorThreadCount = 1;
        }
        if (serverSocket == null) {
            try {
                if (address == null) {
    //基于特定端口创建一个ServerSocket对象，准备接受请求
                    serverSocket = serverSocketFactory.createSocket(port, backlog);
                } else {
                    serverSocket = serverSocketFactory.createSocket(port, backlog, address);
                }
            } catch (BindException orig) {
             ……
            }
        }
}
```
在上面的代码中，我们可以看到此时初始化了一个ServerSocket对象，用来准备接受请求。

如果将其比作赛跑，此时已经到了“各就各位”状态，就等最终的那声“发令枪”了，而Catalina的start方法就是“发令枪”啦：
```
public void start() {
        if (getServer() == null) {
            load();
        }
        if (getServer() == null) {
            log.fatal("Cannot start server. Server instance is not configured.");
            return;
        }
        if (getServer() instanceof Lifecycle) {
            try {
                ((Lifecycle) getServer()).start();
            } catch (LifecycleException e) {
                log.error("Catalina.start: ", e);
            }
        }
      //……
 }
```
此时会调用Server的start方法，这里我们重点还是关注JIoEndpoint的start方法：
```
public void start()  throws Exception {
        if (!initialized) {
            init();
        }
        if (!running) {
            running = true;
            paused = false;
            if (executor == null) {
    //初始化处理连接的线程，maxThread的默认值为200，这也就是为什么    //说Tomcat只能同时处理200个请求的来历
                workers = new WorkerStack(maxThreads);
            }
            for (int i = 0; i < acceptorThreadCount; i++) {
    //初始化接受请求的线程
                Thread acceptorThread = new Thread(new Acceptor(), getName() + "-Acceptor-" + i);
                acceptorThread.setPriority(threadPriority);
                acceptorThread.setDaemon(daemon);
                acceptorThread.start();
            }
        }
    }
```
从以上的代码，可以看到，如果没有在server.xml中声明Executor的话，将会使用内部的一个容量为200的线程池用来后续的请求处理。并且按照参数acceptorThreadCount的设置，初始化线程来接受请求。而Acceptor是真正的幕后英雄，接受请求并分派给处理过程：
```
protected class Acceptor implements Runnable {
        public void run() {
            while (running) {
                // 接受发送过来的请求
    Socket socket = serverSocketFactory.acceptSocket(serverSocket);
                    serverSocketFactory.initSocket(socket);
                    //处理这个请求
                    if (!processSocket(socket)) {
                        //关闭连接
                        try {
                            socket.close();
                        } catch (IOException e) {
                            // Ignore
                        }
                    }
            }
        }
}
```
从这里我们可以看到，Acceptor接受Socket请求，并调用processSocket方法来进行请求的处理。至此，Tomcat的组件整装待命，等待请求的到来。关于请求的处理，会在下篇文章中介绍。

###4、Tomcat对HTTP请求处理的整体流程
前面的文章中介绍了Tomcat初始化的过程，本文将会介绍Tomcat对HTTP请求的处理的整体流程，更细节的。

在上一篇文章中，介绍到JIoEndpoint 中的内部类Acceptor用来接受Socket请求，并调用processSocket方法来进行请求的处理，所以会从本文这个方法开始进行讲解。
```
protected boolean processSocket(Socket socket) {
        try {
            if (executor == null) {
                getWorkerThread().assign(socket);
            } else {
                executor.execute(new SocketProcessor(socket));
            }
        } catch (Throwable t) {
            //……此处略去若干代码
        }
        return true;
    }
```
在以上的代码中，首先会判断是否在server.xml配置了进程池，如果配置了的话，将会使用该线程池进行请求的处理，如果没有配置的话将会使用JIoEndpoint中自己实现的线程池WorkerStack来进行请求的处理，我们将会介绍WorkerStack的请求处理方式。
```
protected Worker getWorkerThread() {
        // Allocate a new worker thread
        synchronized (workers) {
            Worker workerThread;
            while ((workerThread = createWorkerThread()) == null) {
                try {
                    workers.wait();
                } catch (InterruptedException e) {
                    // Ignore
                }
            }
            return workerThread;
        }
    }
```
在以上的代码中，最终返回了一个Worker的实例，有其来进行请求的处理，在这里，我们再看一下createWorkerThread方法，该方法会生成或者在线程池中取到一个线程。
```
protected Worker createWorkerThread() {
        synchronized (workers) {
            if (workers.size() > 0) {
            //如果线程池中有空闲的线程，取一个
                curThreadsBusy++;
                return workers.pop();
            }
            if ((maxThreads > 0) && (curThreads < maxThreads)) {
                //如果还没有超过最大线程数，会新建一个线程
                curThreadsBusy++;
                return (newWorkerThread());
            } else {
                if (maxThreads < 0) {
                    curThreadsBusy++;
                    return (newWorkerThread());
                } else {
                    return (null);
                }
            }
        }
    }
```
到此，线程已经获取了，接下来，最关键的是调用线程实现Worker的run方法：
```
public void run() {
            // Process requests until we receive a shutdown signal
            while (running) {
                // Wait for the next socket to be assigned
                Socket socket = await();
                if (socket == null)
                    continue;
                   if (!setSocketOptions(socket) || !handler.process(socket)) {
                    try {
                        socket.close();
                    } catch (IOException e) {
                    }
                }
                socket = null;
                recycleWorkerThread(this);
            }
}
```
这里跟请求处理密切相关的是handler.process(socket)这一句代码，此处handle对应的类是Http11Protocol中的内部类Http11ConnectionHandler，在此后的处理中，会有一些请求的预处理，我们用一个时序图来表示一下：
![](2012090922371273.png)
在这个过程中，会对原始的socket进行一些处理，到CoyoteAdapter时，接受的参数已经是org.apache.coyote.Request和org.apache.coyote.Response了，但是要注意的是，此时这两个类并不是我们常用的HttpServletRequest和HttpServletResponse的实现类，而是Tomcat内部的数据结构，存储了和输入、输出相关的信息。值得注意的是，在CoyoteAdapter的service方法中，会调用名为postParseRequest的方法，在这个方法中，会解析请求，调用Mapper的Map方法来确定该请求该由哪个Engine、Host和Context来处理。

在以上的信息处理完毕后，在CoyoteAdapter的service方法中，会调用这样一个方法：

connector.getContainer().getPipeline().getFirst().invoke(request, response);
这个方法会涉及到Tomcat组件中的Container实现类的重要组成结构，即每个容器类组件都有一个pipeline属性，这个属性控制请求的处理过程，在pipeline上可以添加Valve，进而可以控制请求的处理流程。可以用下面的图来表示，请求是如何流动的：
[](2012090922384444.png)
可以将请求想象成水的流动，请求需要在各个组件之间流动，中间经过若干的水管和水阀，等所有的水阀走完，请求也就处理完了，而每个组件都会有一个默认的水阀（以Standard作为类的前缀）来进行请求的处理，如果业务需要的话，可以自定义Valve，将其安装到容器中。

后面的处理过程就比较类似了，会按照解析出来的Engine、Host、Context的顺序来进行处理。这里用了两张算不上标准的时序图来描述这一过程：
[](2012090922391743.png)
[](2012090922395537.png)

在以上的流程中，会一层层地调用各个容器组件的Valve的invoke方法，其中StandardWrapperValve这个标准阀门将会调用StandardWrapper的allocate方法来获取真正要执行的Servlet（在Tomcat中所有的请求最终都会映射到一个Servlet，静态资源和JSP也是如此），并按照请求的地址来构建过滤器链，按照顺序执行各个过滤器并最终调用目标Servlet的service方法，来完成业务的真正处理。

以上的处理过程中，涉及到很多重要的代码，后续的文章会择期要者进行解析，如：

Mapper中的internalMapWrapper方法（用来匹配对应的Servlet）
ApplicationFilterFactory的createFilterChain方法（用来创建该请求的过滤器链）
ApplicationFilterChain的internalDoFilter方法（用来执行过滤器方法以及最后的Servlet）
Http11Processor中的process方法、prepareRequest方法以及prepareResponse方法（用来处理HTTP请求相关的协议、参数等信息）
至此，我们简单了解一个请求的处理流程。

###5、













参考：
[Tomcat源码解读系列](http://www.cnblogs.com/levinzhang/archive/2012/08/25/2655617.html)
