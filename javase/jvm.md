### 1、JVM内存组成
---
![](/image/20140218172737265.png)
由上图可知，java内存主要分为6部分，分别是程序计数器，虚拟机栈，本地方法栈，堆，方法区和直接内存(?直接内存不算吧),下面将逐一详细描述。
1、program counter register程序计数器
线程私有，即每个线程都会有一个，线程之间互不影响，独立存储。
代表着当前线程所执行字节码的行号指示器。

2、vm statck虚拟机栈(又名方法栈)
线程私有，它的生命周期和线程相同。
描述的是java方法执行的内存模型：每个方法在执行的同时都会创建一个栈帧，用于存储局部变量表、操作数栈、动态链表、方法出口等信息。
每一个方法从调用直至完成的过程，就对应着一个栈帧在虚拟机中入栈到出栈的过程。
局部变量表存放了编译期可知的各种基本数据类型和对象引用，所需内存空间在编译期确定。
-Xoss参数设置本地方法栈大小（对于HotSpot无效）
-Xss参数设置栈容量 例： -Xss128k

3、本地方法栈
同虚拟机栈，只不过本地方法栈是虚拟机使用到的native方法服务。
Sun HotSpot虚拟机把本地方法栈和虚拟机栈合二为一。

4、java堆
线程共享
主要用于分配对象实例和数组。
-Xms参数设置最小值
-Xmx参数设置最大值 例：VM Args: -Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
若-Xms=-Xmx,则可避免堆自动扩展。
-XX:+HeapDumpOnOutOfMemoryError可以让虚拟机在出现内存溢出是dump出当前的内存堆转储快照。

5、方法区
线程共享
用于存储已被虚拟机加载的类信息、常量、静态变量、即使编译后的代码等数据。
别名永久代（Permanent Generation）
-XX:MaxPermSize设置上限
-XX:PermSize设置最小值 例：VM Args:-XX:PermSize=10M -XX:MaxPermSize=10M
运行时常量池(Runtime Constant Pool)是方法区的一部分。
Class文件中除了有类的版本、字段、方法、接口等信息外，还有一项是常量池（Constant Pool Table）,用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放。
运行时常量池相对于Class文件常量池的一个重要特征是具备动态性：即除了Class文件中常量池的内容能被放到运行时常量池外，运行期间也可能将新的常量放入池中，比如String类的intern（）方法。

6、直接内存(堆外内存？)
直接内存并不是虚拟机运行时数据区的一部分。
在NIO中，引入了一种基于通道和缓冲区的I/O方式，它可以使用native函数直接分配堆外内存，然后通过一个存储在java堆中的DirectByteBuffer对象作为这块内存的引用进行操作。
-XX:MaxDirectMemorySize设置最大值，默认与java堆最大值一样。
例：-XX:MaxDirectMemorySize=10M -Xmx20M

### 2、堆内存
---
Heap Memory 又被分为两大区域：
- Young/New Generation 新生代
 新生对象放置在新生代中，新生代由Eden 与Survivor Space 组成。
- Old/Tenured Generation 老年代
老年代用于存放程序中经过几次垃圾回收后还存活的对象
![](/image/6af6a224-8b2d-3f23-8b58-79263cfda9c4.png)

*Non Heap Memory 非堆内存*
除了堆内存区域用来存放存活(living)的数据，JVM 还需要尤其是类描述、元数据等更多信息。所以这些信息统一被存放在命名为Permanent generation(永久/常驻代)的区域。
非堆内存其实就是JVM 留给自己用的，所以方法区、JVM 内部处理或优化所需的内存(如JIT编译后的代码缓存)、每个类结构(如运行时常数池、字段和方法数据)以及方法和构造方法的代码等都在非堆内存中。
非堆内存由JVM 管理，我们无法在程序中使用。

1.Young/New Generation 新生代
程序中新建的对象都将分配到新生代中，新生代又由Eden(伊甸园)与两块Survivor(幸存者) Space 构成。Eden 与Survivor Space 的空间大小比例默认为8:1，即当Young/New Generation 区域的空间大小总数为10M 时，Eden 的空间大小为8M，两块Survivor Space 则各分配1M，这个比例可以通过-XX:SurvivorRatio 参数来修改。Young/New Generation的大小则可以通过-Xmn参数来指定。
 
Eden：刚刚新建的对象将会被放置到Eden 中，这个名称寓意着对象们可以在其中快乐自由的生活。
Survivor Space：幸存者区域是新生代与老年代的缓冲区域，两块幸存者区域分别为s0 与s1，当触发Minor GC 后将仍然存活的对象移动到S0中去(From Eden To s0)。这样Eden 就被清空可以分配给新的对象。
当再一次触发Minor GC后，S0和Eden 中存活的对象被移动到S1中(From s0To s1)，S0即被清空。在同一时刻, 只有Eden和一个Survivor Space同时被操作。所以s0与s1两块Survivor 区同时会至少有一个为空闲的，这点从下面的图中可以看出。
当每次对象从Eden 复制到Survivor Space 或者从Survivor Space 之间复制，计数器会自动增加其值。 默认情况下如果复制发生超过16次，JVM 就会停止复制并把他们移到老年代中去。如果一个对象不能在Eden中被创建，它会直接被创建在老年代中。
新生代GC(Minor GC)：指发生在新生代的垃圾收集动作，因为 Java 对象大多都具备朝生夕灭的特性，通常很多的对象都活不过一次GC，所以Minor GC 非常频繁，一般回收速度也比较快。
Minor GC 清理过程(图中白色区域为垃圾)：

2.Old/Tenured Generation 老年代
老年代用于存放程序中经过几次垃圾回收后还存活的对象，例如缓存的对象等，老年代所占用的内存大小即为-Xmx 与-Xmn 两个参数之差。
堆是JVM 中所有线程共享的，因此在其上进行对象内存的分配均需要进行加锁，这也导致了new 对象的开销是比较大的，鉴于这样的原因，Hotspot JVM 为了提升对象内存分配的效率，对于所创建的线程都会分配一块独立的空间，这块空间又称为TLAB（Thread Local Allocation Buffer），其大小由JVM 根据运行的情况计算而得，在TLAB 上分配对象时不需要加锁，因此JVM 在给线程的对象分配内存时会尽量的在TLAB 上分配，在这种情况下JVM 中分配对象内存的性能和C 基本是一样高效的，但如果对象过大的话则仍然是直接使用堆空间分配，TLAB 仅作用于新生代的Eden，因此在编写Java 程序时，通常多个小的对象比大的对象分配起来更加高效，但这种方法同时也带来了两个问题，一是空间的浪费，二是对象内存的回收上仍然没法做到像Stack 那么高效，同时也会增加回收时的资源的消耗，可通过在启动参数上增加 -XX:+PrintTLAB来查看TLAB 这块的使用情况。
 
老年代GC(Major GC/Full GC)：指发生在老年代的GC，出现了Major GC，通常会伴随至少一次Minor GC（但也并非绝对，在ParallelScavenge 收集器的收集策略里则可选择直接进行Major GC）。MajorGC 的速度一般会比Minor GC 慢10倍以上。
 虚拟机给每个对象定义了一个对象年龄(age)计数器。如果对象在 Eden 出生并经过第一次 Minor GC 后仍然存活，并且能被 Survivor 容纳的话，将被移动到 Survivor 空间中，并将对象年龄设为 1。对象在 Survivor 区中每熬过一次 Minor GC，年龄就增加 1 岁，当它的年龄增加到一定程度（默认为 15 岁）时，就会被晋升到老年代中。对象晋升老年代的年龄阈值，可以通过参数 -XX:MaxTenuringThreshold 来设置。

当一个Object被创建后，内存申请过程如下：
1.JVM 会试图为相关Java 对象在Eden 中初始化一块内存区域。
2.当Eden 空间足够时，内存申请结束。否则进入第三步。
3.JVM 试图释放在Eden 中所有不活跃的对象（这属于1或更高级的垃圾回收）, 释放后若Eden 空间仍然不足以放入新对象，则试图将部分Eden 中活跃对象放入Survivor 区。
4.Survivor 区被用来作为新生代与老年代的缓冲区域，当老年代空间足够时，Survivor 区的对象会被移到老年代，否则会被保留在Survivor 区。
5.当老年代空间不够时，JVM 会在老年代进行0级的完全垃圾收集(Major GC/Full GC)。
 6.Major GC/Full G后，若Survivor 及老年代仍然无法存放从Eden 复制过来的部分对象，导致JVM 无法在Eden 区为新对象创建内存区域，JVM 此时就会抛出内存不足的异常。

通过jvmstat 可以清晰的观察出JVM 的各个过程：
![](42eadbf7-937f-3436-bf2e-c69c6bd20f63.png)
从jvmstat中可以清晰的观察到汇编，加载，垃圾回收消耗的时间与各区域内存使用情况，在图中s0与s1的内存使用永远都是相斥的，即至多只有一个会在使用。
        还可以使用'YourKit Java Profiler'这个强大的工具观察更多的内存及class情况，关于YourKit Java Profiler 可以参考另一篇文章。
        在32位系统下可以为JVM 分配最大2GB 堆内存大小，64位则没有限制，下列是一些常用与Heap 相关的参数：
-Xms：初始堆大小，默认为物理内存的1/64(<1GB)；默认(MinHeapFreeRatio参数可以调整)空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制 

-Xmx：最大堆大小，默认(MaxHeapFreeRatio参数可以调整)空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制 

-Xmn：新生代的内存空间大小，即Eden+ 2个survivor space。 
在保证堆大小不变的情况下，增大新生代后,将会减小老生代大小。此值对系统性能影响较大,Sun官方推荐配置为整个堆的3/8。 

-XX:SurvivorRatio：新生代中Eden区域与Survivor区域的容量比值，默认值为8。两个Survivor区与一个Eden区的比值为2:8,一个Survivor区占整个年轻代的1/10。 

-Xss：每个线程的堆栈大小。JDK5.0以后每个线程堆栈大小为1M,以前每个线程堆栈大小为256K。应根据应用的线程所需内存大小进行适当调整。在相同物理内存下,减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。一般小的应用， 如果栈不是很深， 应该是128k够用的，大的应用建议使用256k。这个选项对性能影响比较大，需要严格的测试。和threadstacksize选项解释很类似,官方文档似乎没有解释,在论坛中有这样一句话:"-Xss is translated in a VM flag named ThreadStackSize”一般设置这个值就可以了。 

-XX:PermSize：设置永久代(perm gen)初始值。默认值为物理内存的1/64。 

-XX:MaxPermSize：设置持久代最大值。物理内存的1/4。



- 永久代
永久代或者“Perm Gen”包含了JVM需要的应用元数据，这些元数据描述了在应用里使用的类和方法。注意，永久代不是Java堆内存的一部分。

永久代存放JVM运行时使用的类。永久代同样包含了Java SE库的类和方法。永久代的对象在full GC时进行垃圾收集。

方法区是永久代空间的一部分，并用来存储类型信息（运行时常量和静态变量）和方法代码和构造函数代码。

You can run the following command to see default values:
java -XX:+PrintFlagsFinal -version

### 3、垃圾回收机制

*在垃圾收集领域有个很有意思的语句“Stop the world”，原因是当执行垃圾回收时需要停止所有的用户线程。*

![](/image/20140220152548218.png)

Serial GC（-XX:+UseSerialGC）：Serial GC使用简单的标记、清除、压缩方法对年轻代和年老代进行垃圾回收，即Minor GC和Major GC。Serial GC在client模式（客户端模式）很有用，比如在简单的独立应用和CPU配置较低的机器。这个模式对占有内存较少的应用很管用。

Parallel GC（-XX:+UseParallelGC）：除了会产生N个线程来进行年轻代的垃圾收集外，Parallel GC和Serial GC几乎一样。这里的N是系统CPU的核数。我们可以使用 -XX:ParallelGCThreads=n 这个JVM选项来控制线程数量。并行垃圾收集器也叫throughput收集器。因为它使用了多CPU加快垃圾回收性能。Parallel GC在进行年老代垃圾收集时使用单线程。

Parallel Old GC（-XX:+UseParallelOldGC）：和Parallel GC一样。不同之处，Parallel Old GC在年轻代垃圾收集和年老代垃圾回收时都使用多线程收集。

并发标记清除（CMS）收集器（-XX:+UseConcMarkSweepGC)：CMS收集器也被称为短暂停顿并发收集器。它是对年老代进行垃圾收集 的。CMS收集器通过多线程并发进行垃圾回收，尽量减少垃圾收集造成的停顿。CMS收集器对年轻代进行垃圾回收使用的算法和Parallel收集器一样。 这个垃圾收集器适用于不能忍受长时间停顿要求快速响应的应用。可使用 -XX:ParallelCMSThreads=n JVM选项来限制CMS收集器的线程数量。

G1垃圾收集器（-XX:+UseG1GC) G1（Garbage First）：垃圾收集器是在Java 7后才可以使用的特性，它的长远目标时代替CMS收集器。G1收集器是一个并行的、并发的和增量式压缩短暂停顿的垃圾收集器。G1收集器和其他的收集器运 行方式不一样，不区分年轻代和年老代空间。它把堆空间划分为多个大小相等的区域。当进行垃圾收集时，它会优先收集存活对象较少的区域，因此叫 “Garbage First”。

一、垃圾回收触发条件
  1、Minor gc触发条件
当新生代空间不足时会主动触发Minor gc，并且自动扩容（可通过控制使新生代直接处于最大内存空间，避免自动扩容和垃圾收集）。
  2、Full gc触发条件
和新生代一样，当老年代空间不足时会触发Full gc，并且自动扩容；另外当在代码中调用System.gc()时也会触发Full gc。 可通过参数-XX:+DisableExplicitGC 控制使System.gc()失效。
  3、永久待触发条件
类似上面，当永久待空间不足时，会发出Full gc，可通过控制PermSize=MaxPermSize 避免自动扩容和垃圾回收。另外可通过参数-Xnoclassgc 来控制虚拟机不对类（永久待类对象）进行回收。

### 4、jvm参数
一个性能较好的web服务器jvm参数配置：
-server	//服务器模式
-Xmx2g //JVM最大允许分配的堆内存，按需分配
-Xms2g //JVM初始分配的堆内存，一般和Xmx配置成一样以避免每次gc后JVM重新分配内存。
-Xmn256m //年轻代内存大小，整个JVM内存=年轻代 + 年老代 + 持久代
-XX:PermSize=128m //持久代内存大小
-Xss256k //设置每个线程的堆栈大小
-XX:+DisableExplicitGC //忽略手动调用GC, System.gc()的调用就会变成一个空调用，完全不触发GC
-XX:+UseConcMarkSweepGC //并发标记清除（CMS）收集器
-XX:+CMSParallelRemarkEnabled //降低标记停顿
-XX:+UseCMSCompactAtFullCollection //在FULL GC的时候对年老代的压缩
-XX:LargePageSizeInBytes=128m //内存页的大小
-XX:+UseFastAccessorMethods //原始类型的快速优化
-XX:+UseCMSInitiatingOccupancyOnly //使用手动定义初始化定义开始CMS收集
-XX:CMSInitiatingOccupancyFraction=70 //使用cms作为垃圾回收使用70％后开始CMS收集

垃圾收集打印参数:
 -XX:+PrintGCApplicationStoppedTime 参数打印gc停顿时间
 -XX:+PrintGCDateStamps  参数打印gc时间戳，系统时间
 -XX:+PrintGCTimeStamps 参数打印gc时间，相对于jvm启动时间
 -Xloggc:gclog.log  设置gc日志文件
 -XX:+PrintReferenceGC  打印gc引用
 -XX:+PrintGCApplicationConcurrentTime 打印每次垃圾回收前，程序未中断的执行时间
 -XX:+PrintHeapAtGC 打印GC前后的详细堆栈信息

heap dump:



### 5、JIT编译器
首先，我们大家都知道，通常通过 javac 将程序源代码编译，转换成 java 字节码，JVM 通过解释字节码将其翻译成对应的机器指令，逐条读入，逐条解释翻译。很显然，经过解释执行，其执行速度必然会比可执行的二进制字节码程序慢很多。为了提高执行速度，引入了 JIT 技术。
在运行时 JIT 会把翻译过的机器码保存起来，以备下次使用，因此从理论上来说，采用该 JIT 技术可以接近以前纯编译技术。下面我们看看，JIT 的工作过程。
JIT 工作原理图:
![](/image/img001.png)

寄存器和主存:


参考：
[深入理解java虚拟机](http://blog.csdn.net/chaofanwei/article/details/19418753)
[Java 堆内存(Heap)](http://286.iteye.com/blog/1931174)
[深入浅出 JIT 编译器](https://www.ibm.com/developerworks/cn/java/j-lo-just-in-time/)
