###1、解决OO
---
http://jinnianshilongnian.iteye.com/blog/1997293

http://www.oschina.net/question/560995_137855

###2、线上应用故障排查之一：高CPU占用


最后，总结下排查CPU故障的方法和技巧有哪些：

1、top命令：Linux命令。可以查看实时的CPU使用情况。也可以查看最近一段时间的CPU使用情况。

2、PS命令：Linux命令。强大的进程状态监控命令。可以查看进程以及进程中线程的当前CPU使用情况。属于当前状态的采样数据。

3、jstack：Java提供的命令。可以查看某个进程的当前线程栈运行情况。根据这个命令的输出可以定位某个进程的所有线程的当前运行状态、运行代码，以及是否死锁等等。

4、pstack：Linux命令。可以查看某个进程的当前线程栈运行情况。


###3、线上应用故障排查之二：高内存占用

最后，总结下排查内存故障的方法和技巧有哪些：

1、top命令：Linux命令。可以查看实时的内存使用情况。  

2、jmap -histo:live [pid]，然后分析具体的对象数目和占用内存大小，从而定位代码。

3、jmap -dump:live,format=b,file=xxx.xxx [pid]，然后利用MAT工具分析是否存在内存泄漏等等。

###4、线上应用故障排查之三：高I/O占用，包括磁盘I/O、网络I/O、数据库I/O等。


###5、线上应用故障排查之四：程序僵死






参考:

- [](http://www.blogjava.net/hankchen/archive/2012/05/09/377738.html)