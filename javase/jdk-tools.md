### 1、jvisualvm
---
#### 1.1、简介
简单说来，VisualVM是jConsole的升级版，但它可比jConsole好用多了。它能为您提供强大的thread和heap分析能力。它囊括的命令行工具包括JConsole, jstack,jstat, jmap ,jps。参考网址：

- VisualVM入门指南：https://visualvm.dev.java.net/zh_CN/gettingstarted.html 
- VIsualVM介绍：    http://www.iteye.com/topic/516447 
- jstatd介绍：      http://java.sun.com/javase/6/docs/technotes/tools/share/jstatd.html

#### 1.2、遇到的坎

- 添加jstatd连接后，显示列表为空，不能列出待监控的应用，解决方法：jstatd -J-Djava.security.policy=.jstatd.all.policy -J-Djava.rmi.server.hostname=10.82.83.117 -J-Djava.rmi.server.logCalltrue,参考[jvisualvm connect to remote jstatd not showing applications](http://stackoverflow.com/questions/32515727/jvisualvm-connect-to-remote-jstatd-not-showing-applications)

- 对于“堆 dump”来说，在远程监控jvm的时候，VisualVM是没有这个功能的，只有本地监控的时候才有。另外，就算是本地监控，它在dump和得到实例的 速度那是相当的慢的。所以鉴于这几个原因，不建议用VisualVM，而是用jmap加上Mat来分析内存情况。

#### 1.3、JProfiler

http://www.ej-technologies.com

#### 1.4、java工具
>>3.1、OutOfMemoryError
- 1、use jvmstat first
- 2、try to see heap dump，使用jHAT分析；

32位的JAVA所能操作的内存只有2G？JVM heap全部占完了会导致Native heap无法allocate memory？

### 2、jmap+mat
---




### 3、jinfo
---



### 4、jstat
---



### 5、jconsole
---