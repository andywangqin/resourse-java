###1、简介
---
####1.1、概述
Nginx("engine x")是一个高性能的HTTP和反向代理服务器(代理服务器和server位于同一网段)，也是一个 IMAP/POP3/SMTP 代理服务器 。Nginx是由Igor Sysoev为俄罗斯访问量第二的Rambler.ru站点开发的，它已经在该站点运行超过四年多了。Igor将源代码以类BSD许可证的形式发布。自Nginx 发布四年来，Nginx 已经因为它的稳定性、丰富的功能集、示例配置文件和低系统资源的消耗而闻名了。目前国内各大门户网站已经部署了Nginx，如新浪、网易、腾讯等；国内几个重要的视频分享网站也部署了Nginx，如六房间、酷6等。 新近发现Nginx 技术在国内日趋火热，越来越多的网站开始部署Nginx。

####1.2、优点
Nginx 是一个很牛的高性能Web和反向代理服务器, 它具有有很多非常优越的特性:

- 在高连接并发的情况下，Nginx是Apache服务器不错的替代品: Nginx在美国是做虚拟主机生意的老板们经常选择的软件平台之一. 能够支持高达50,000 个并发连接数的响应, 感谢Nginx为我们选择了 epoll and kqueue 作为开发模型.
- Nginx作为负载均衡服务器: Nginx 既可以在内部直接支持 Rails 和 PHP 程序对外进行服务, 也可以支持作为HTTP代理 服务器对外进行服务. Nginx采用C进行编写, 不论是系统资源开销还是CPU使用效率都比 Perlbal 要好很多.
- 作为邮件代理服务器: Nginx 同时也是一个非常优秀的邮件代理服务器（最早开发这个产品的目的之一也是作为邮件代理服务器）, Last.fm 描述了成功并且美妙的使用经验.
- Nginx 是一个 [#installation 安装] 非常的简单 , 配置文件非常简洁（还能够支持perl语法）, Bugs 非常少的服务器: Nginx 启动特别容易, 并且几乎可以做到7*24不间断运行，即使运行数个月也不需要重新启动. 你还能够不间断服务的情况下进行软件版本的升级 .

####1.3、安装Nginx

####1.4、运行Nginx
选项
-c </path/to/config> 为 Nginx 指定一个配置文件，来代替缺省的。
-t 不运行，而仅仅测试配置文件。nginx 将检查配置文件的语法的正确性，并尝试打开配置文件中所引用到的文件。
-v 示 nginx 的版本。
-V 显示 nginx 的版本，编译器版本和配置参数。

这个示例仅测试指定位置的配置文件是否正常，并给出提示。
```
/usr/bin/nginx -t -c ~/mynginx.conf
```

####1.5、通过系统的信号控制Nginx
*信号（英语：Signals）*是Unix、类Unix以及其他POSIX兼容的操作系统中进程间通讯的一种有限制的方式。它是一种异步的通知机制，用来提醒进程一个事件已经发生。当一个信号发送给一个进程，操作系统中断了进程正常的控制流程，此时，任何非原子操作都将被中断。如果进程定义了信号的处理函数，那么它将被执行，否则就执行默认的处理函数。

主进程可以处理以下的信号：

|TERM, INT|	快速关闭|
|--:|--:|
|QUIT	|从容关闭|
|HUP|	重载配置,用新的配置开始新的工作进程,从容关闭旧的工作进程|
|USR1	|重新打开日志文件|
|USR2	|平滑升级可执行程序。|
|WINCH	|从容关闭工作进程|

尽管你不必自己操作_工作进程_，但是，它们也支持一些信号：

|TERM, INT	|快速关闭|
|--:|--:|
|QUIT	|从容关闭|
|USR1	|重新打开日志文件|

- 使用信号加载新的配置
- 升级nginx

####1.6、事件模型
1. hash表

Ngnix使用hash表来协助完成请求的快速处理。

考虑到保存键及其值的hash表存储单元的大小不至于超出设定参数(hash bucket size)， 在启动和每次重新配置时，Nginx为hash表选择尽可能小的尺寸。

直到hash表超过参数(hash max size)的大小才重新进行选择. 对于大多数hash表都有指令来修改这些参数。例如，保存服务器名字的hash表是由指令 server_names_hash_max_size 和 server_names_hash_bucket_size所控制的。参数hash bucket size总是等于hash表的大小，并且是一路处理器缓存大小的倍数。在减少了在内存中的存取次数后，使在处理器中加速查找hash表键值成为可能。如果hash bucket size等于一路处理器缓存的大小，那么在查找键的时候，最坏的情况下在内存中查找的次数为2。第一次是确定存储单元的地址，第二次是在存储单元中查找键值。因此，如果Nginx给出需要增大 hash max size 或 hash bucket size的提示，那么首要的是增大前一个参数的大小.

2. 事件模型

Nginx支持如下处理连接的方法（I/O复用方法），这些方法可以通过use指令指定。

- select - 标准方法。 如果当前平台没有更有效的方法，它是编译时默认的方法。你可以使用配置参数 --with-select_module 和 --without-select_module 来启用或禁用这个模块。
- poll - 标准方法。 如果当前平台没有更有效的方法，它是编译时默认的方法。你可以使用配置参数 --with-poll_module 和 --without-poll_module 来启用或禁用这个模块。
- kqueue - 高效的方法，使用于 FreeBSD 4.1+, OpenBSD 2.9+, NetBSD 2.0 和 MacOS X. 使用双处理器的MacOS X系统使用kqueue可能会造成内核崩溃。
- epoll - 高效的方法，使用于Linux内核2.6版本及以后的系统。在某些发行版本中，如SuSE 8.2, 有让2.4版本的内核支持epoll的补丁。
- rtsig - 可执行的实时信号，使用于Linux内核版本2.2.19以后的系统。默认情况下整个系统中不能出现大于1024个POSIX实时(排队)信号。这种情况对于高负载的服务器来说是低效的；所以有必要通过调节内核参数 /proc/sys/kernel/rtsig-max 来增加队列的大小。可是从Linux内核版本2.6.6-mm2开始， 这个参数就不再使用了，并且对于每个进程有一个独立的信号队列，这个队列的大小可以用 RLIMIT_SIGPENDING 参数调节。当这个队列过于拥塞，nginx就放弃它并且开始使用 poll 方法来处理连接直到恢复正常。
- /dev/poll - 高效的方法，使用于 Solaris 7 11/99+, HP/UX 11.22+ (eventport), IRIX 6.5.15+ 和 Tru64 UNIX 5.1A+.
- eventport - 高效的方法，使用于 Solaris 10. 为了防止出现内核崩溃的问题， 有必要安装 这个 安全补丁。

####1.7、调试 nginx
Nginx的一个 杀手级特性 就是你能使用 debug_connection 指令只调试 某些 连接。

这个设置只有是你使用 --with-debug 编译的nginx才有效。

在 这里 你能看到所有可能的调试选项。

###2、主模块
[#daemon daemon]
[#debug_points debug_points]
[#error_log error_log]
[#include include]
[#lock_file lock_file]
[#master_process master_process]
[#pid pid]
[#ssl_engine ssl_engine]
[#timer_resolution timer_resolution]
[#user user group]
[#worker_cpu_affinity worker_cpu_affinity]
[#worker_priority worker_priority]
[#worker_processes worker_processes]
[#worker_rlimit_core worker_rlimit_core]
[#worker_rlimit_nofile worker_rlimit_nofile]
[#worker_rlimit_sigpending worker_rlimit_sigpending]
[#working_directory working_directory]

###3、事件模块
accept_mutex
accept_mutex_delay
debug_connection
devpoll_changes
devpoll_events
kqueue_changes
kqueue_events
epoll_events
multi_accept
rtsig_signo
rtsig_overflow_events
rtsig_overflow_test
rtsig_overflow_threshold

###4、HTTP模块

###5、第三方模块


###9、案例






参考：

- [Nginx 的中文维基](http://tool.oschina.net/apidocs/apidoc?api=nginx-zh)