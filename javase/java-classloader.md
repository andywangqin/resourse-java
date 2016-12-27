### 1、概念
---
*类加载机制*: 是指虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的java类型。类加载和连接的过程都是在运行期间完成的。

### 2、类的加载方式
---
- 1）：本地编译好的class中直接加载
- 2）：网络加载：java.net.URLClassLoader可以加载url指定的类
- 3）：从jar、zip等等压缩文件加载类，自动解析jar文件找到class文件去加载util类
- 4）：从java源代码文件动态编译成为class文件

### 3、类的生命周期
---
类从被加载到虚拟机内存中开始，直到卸载出内存为止，它的整个生命周期包括了：加载、验证、准备、解析、初始化、使用和卸载这7个阶段。其中，验证、准备和解析这三个部分统称为连接（linking）。
![](/image/20140317163048593.jpg)
其中，加载、验证、准备、初始化和卸载这五个阶段的顺序是确定的，类的加载过程必须按照这种顺序按部就班的“开始”（仅仅指的是开始，而非执行或者结束，因为这些阶段通常都是互相交叉的混合进行，通常会在一个阶段执行的过程中调用或者激活另一个阶段），而解析阶段则不一定（它在某些情况下可以在初始化阶段之后再开始，这是为了支持Java语言的运行时绑定）。

### 4、类的加载过程
---
#### 4.1 加载
 “加载”(Loading)阶段是“类加载”(Class Loading)过程的第一个阶段，在此阶段，虚拟机需要完成以下三件事情：

1. 通过一个类的全限定名来获取定义此类的二进制字节流。
2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
3. 在Java堆中生成一个代表这个类的java.lang.Class对象，作为方法区这些数据的访问入口。

加载阶段即可以使用系统提供的类加载器完成(Bootstrap Loader——》Extended Loader——》System Loader)，也可以由用户自定义的类加载器来完成。加载阶段与连接阶段的部分内容(如一部分字节码文件格式验证动作)是交叉进行的，加载阶段尚未完成，连接阶段可能已经开始。

#### 4.2 验证
 
验证是连接阶段的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。

Java语言本身是相对安全的语言，使用Java编码是无法做到如访问数组边界以外的数据、将一个对象转型为它并未实现的类型等，如果这样做了，编译器将拒绝编译。但是，Class文件并不一定是由Java源码编译而来，可以使用任何途径，包括用十六进制编辑器(如UltraEdit)直接编写。如果直接编写了有害的“代码”(字节流)，而虚拟机在加载该Class时不进行检查的话，就有可能危害到虚拟机或程序的安全。

不同的虚拟机，对类验证的实现可能有所不同，但大致都会完成下面四个阶段的验证：文件格式验证、元数据验证、字节码验证和符号引用验证。

1. 文件格式验证，是要验证字节流是否符合Class文件格式的规范，并且能被当前版本的虚拟机处理。如验证魔数是否0xCAFEBABE；主、次版本号是否正在当前虚拟机处理范围之内；常量池的常量中是否有不被支持的常量类型……该验证阶段的主要目的是保证输入的字节流能正确地解析并存储于方法区中，经过这个阶段的验证后，字节流才会进入内存的方法区中存储，所以后面的三个验证阶段都是基于方法区的存储结构进行的。
2. 元数据验证，是对字节码描述的信息进行语义分析，以保证其描述的信息符合Java语言规范的要求。可能包括的验证如：这个类是否有父类；这个类的父类是否继承了不允许被继承的类；如果这个类不是抽象类，是否实现了其父类或接口中要求实现的所有方法……
3. 字节码验证，主要工作是进行数据流和控制流分析，保证被校验类的方法在运行时不会做出危害虚拟机安全的行为。如果一个类方法体的字节码没有通过字节码验证，那肯定是有问题的；但如果一个方法体通过了字节码验证，也不能说明其一定就是安全的。
4. 符号引用验证，发生在虚拟机将符号引用转化为直接引用的时候，这个转化动作将在“解析阶段”中发生。验证符号引用中通过字符串描述的权限定名是否能找到对应的类；在指定类中是否存在符合方法字段的描述符及简单名称所描述的方法和字段；符号引用中的类、字段和方法的访问性(private、protected、public、default)是否可被当前类访问

验证阶段对于虚拟机的类加载机制来说，不一定是必要的阶段。如果所运行的全部代码确认是安全的，可以使用-Xverify：none参数来关闭大部分的类验证措施，以缩短虚拟机类加载时间。

#### 4.3 准备
准备阶段是为类的静态变量分配内存并将其初始化为默认值，这些内存都将在方法区中进行分配。*准备阶段不分配类中的实例变量的内存，实例变量将会在对象实例化时随着对象一起分配在Java堆中*。

public static int value=123;//在准备阶段value初始值为0 。在初始化阶段才会变为123 。

#### 4.4 解析
解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。

符号引用（Symbolic Reference）：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可。符号引用与虚拟机实现的内存布局无关，引用的目标并不一定已经加载到内存中。

直接引用（Direct Reference）：直接引用可以是直接指向目标的指针、相对偏移量或是一个能间接定位到目标的句柄。直接引用是与虚拟机实现的内存布局相关的，如果有了直接引用，那么引用的目标必定已经在内存中存在。

#### 4.5 初始化
**注意与实例化不是一个概念**

类初始化是类加载过程的最后一步，前面的类加载过程，除了在加载阶段用户应用程序可以通过自定义类加载器参与之外，其余动作完全由虚拟机主导和控制。到了初始化阶段，才真正开始执行类中定义的Java程序代码。

初始化阶段是执行类构造器<clinit>()方法的过程。<clinit>()方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块(static{}块)中的语句合并产生的。

##### 4.5.1、类初始化的触发时机
什么情况下需要开始类加载过程的第一个阶段:"加载",虚拟机规范中并没强行约束，这点可以交给虚拟机的的具体实现自由把握，但是对于初始化阶段虚拟机规范是严格规定了如下几种情况，如果类未初始化会对类进行初始化:

- 创建类的实例
- 访问类的静态变量(除常量【被final修辞的静态变量】原因:常量一种特殊的变量，因为编译器把他们当作值(value)而不是域(field)来对待。如果你的代码中用到了常变量(constant variable)，编译器并不会生成字节码来从对象中载入域的值，而是直接把这个值插入到字节码中。这是一种很有用的优化，但是如果你需要改变final域的值那么每一块用到那个域的代码都需要重新编译。
- 访问类的静态方法
- 反射如(Class.forName("my.xyz.Test"))
- 当初始化一个类时，发现其父类还未初始化，则先触发父类的初始化
- 虚拟机启动时，定义了main()方法的那个类先初始化

以上情况称为对一个类进行“主动引用”，除此种情况之外，均不会触发类的初始化，称为“被动引用”

接口的加载过程与类的加载过程稍有不同。接口中不能使用static{}块。当一个接口在初始化时，并不要求其父接口全部都完成了初始化，只有真正在使用到父接口时（例如引用接口中定义的常量）才会初始化。

###### 4.5.1.1、被动引用例子

- 子类调用父类的静态变量，子类不会被初始化。只有父类被初始化。。对于静态字段，只有直接定义这个字段的类才会被初始化.
- 通过数组定义来引用类，不会触发类的初始化
- 访问类的常量，不会初始化类

```
class SuperClass {
    static {
        System.out.println("superclass init");
    }
    public static int value = 123;
}
 
class SubClass extends SuperClass {
    static {
        System.out.println("subclass init");
    }
}
 
public class Test {
    public static void main(String[] args) {
        System.out.println(SubClass.value);// 被动应用1
        SubClass[] sca = new SubClass[10];// 被动引用2
    }
}
```

程序运行输出    
superclass init 
123
从上面的输入结果证明了被动引用1与被动引用2

```
class ConstClass {
    static {
        System.out.println("ConstClass init");
    }
    public static final String HELLOWORLD = "hello world";
}
 
public class Test {
    public static void main(String[] args) {
        System.out.println(ConstClass.HELLOWORLD);// 调用类常量
    }
}
```

程序输出结果
hello world
从上面的输出结果证明了被动引用3

### 5、经典案例分析
---
上面很详细的介绍了类的加载时机和类的加载过程，通过上面的理论来分析本文开门见上的题目
```
class SingleTon {
    private static SingleTon singleTon = new SingleTon();
    public static int count1;
    public static int count2 = 0;
 
    private SingleTon() {
        count1++;
        count2++;
    }
 
    public static SingleTon getInstance() {
        return singleTon;
    }
}
 
public class Test {
    public static void main(String[] args) {
        SingleTon singleTon = SingleTon.getInstance();
        System.out.println("count1=" + singleTon.count1);
        System.out.println("count2=" + singleTon.count2);
    }
}
```
分析:

- 1:SingleTon singleTon = SingleTon.getInstance();调用了类的SingleTon调用了类的静态方法，触发类的初始化
- 2:类加载的时候在准备过程中为类的静态变量分配内存并初始化默认值 singleton=null count1=0,count2=0
- 3:类初始化化，为类的静态变量赋值和执行静态代码快。singleton赋值为new SingleTon()调用类的构造方法
- 4:调用类的构造方法后count=1;count2=1
- 5:继续为count1与count2赋值,此时count1没有赋值操作,所有count1为1,但是count2执行赋值操作就变为0

### 6、类加载器
---
JVM设计者把类加载阶段中的“通过'类全名'来获取定义此类的二进制字节流”这个动作放到Java虚拟机外部去实现，以便让应用程序自己决定如何去获取所需要的类。实现这个动作的代码模块称为“类加载器”。
 
从Java开发人员的角度来看，大部分Java程序一般会使用到以下三种系统提供的类加载器：
- 1)启动类加载器（Bootstrap ClassLoader）：负责加载JAVA_HOME\lib目录中并且能被虚拟机识别的类库到JVM内存中，如果名称不符合的类库即使放在lib目录中也不会被加载。该类加载器无法被Java程序直接引用。
- 2)扩展类加载器（Extension ClassLoader）：该加载器主要是负责加载JAVA_HOME\lib\，该加载器可以被开发者直接使用。
- 3)应用程序类加载器（Application ClassLoader）：也称为系统类加载器，它负责加载用户类路径（Classpath）上所指定的类库，开发者可以直接使用该类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

我们的应用程序都是由这三类加载器互相配合进行加载的，我们也可以加入自己定义的类加载器。

如上图所示的类加载器之间的这种层次关系，就称为类加载器的双亲委派模型（Parent Delegation Model）。该模型要求除了顶层的启动类加载器外，其余的类加载器都应当有自己的父类加载器。子类加载器和父类加载器不是以继承（Inheritance）的关系来实现，而是通过组合（Composition）关系来复用父加载器的代码。

双亲委派模型的工作过程为：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的加载器都是如此，因此所有的类加载请求都会传给顶层的启动类加载器，只有当父加载器反馈自己无法完成该加载请求（该加载器的搜索范围中没有找到对应的类）时，子加载器才会尝试自己去加载。

使用这种模型来组织类加载器之间的关系的好处是Java类随着它的类加载器一起具备了一种带有优先级的层次关系。例如java.lang.Object类，无论哪个类加载器去加载该类，最终都是由启动类加载器进行加载，因此Object类在程序的各种类加载器环境中都是同一个类。否则的话，如果不使用该模型的话，如果用户自定义一个java.lang.Object类且存放在classpath中，那么系统中将会出现多个Object类，应用程序也会变得很混乱。如果我们自定义一个rt.jar中已有类的同名Java类，会发现JVM可以正常编译，但该类永远无法被加载运行。
在rt.jar包中的java.lang.ClassLoader类中，我们可以查看类加载实现过程的代码，具体源码如下：
```
protected synchronized Class loadClass(String name, boolean resolve)  
        throws ClassNotFoundException {  
    // 首先检查该name指定的class是否有被加载  
    Class c = findLoadedClass(name);  
    if (c == null) {  
        try {  
            if (parent != null) {  
                // 如果parent不为null，则调用parent的loadClass进行加载  
                c = parent.loadClass(name, false);  
            } else {  
                // parent为null，则调用BootstrapClassLoader进行加载  
                c = findBootstrapClass0(name);  
            }
        } catch (ClassNotFoundException e) {  
            // 如果仍然无法加载成功，则调用自身的findClass进行加载  
            c = findClass(name);  
        }  
    }  
    if (resolve) {  
        resolveClass(c);  
    }  
    return c;  
}  
```
通过上面代码可以看出，双亲委派模型是通过loadClass()方法来实现的，根据代码以及代码中的注释可以很清楚地了解整个过程其实非常简单：先检查是否已经被加载过，如果没有则调用父加载器的loadClass()方法，如果父加载器为空则默认使用启动类加载器作为父加载器。如果父类加载器加载失败，则先抛出ClassNotFoundException，然后再调用自己的findClass()方法进行加载。
 
##### 6.1、自定义类加载器
若要实现自定义类加载器，只需要继承java.lang.ClassLoader 类，并且重写其findClass()方法即可。java.lang.ClassLoader 类的基本职责就是根据一个指定的类的名称，找到或者生成其对应的字节代码，然后从这些字节代码中定义出一个 Java 类，即 java.lang.Class 类的一个实例。除此之外，ClassLoader 还负责加载 Java 应用所需的资源，如图像文件和配置文件等，ClassLoader 中与加载类相关的方法如下：

| 方法          | 说明 |
| ------------- |:-------------:|
| getParent()  | 返回该类加载器的父类加载器。 |
| loadClass(String name) | 加载名称为 二进制名称为name 的类，返回的结果是 java.lang.Class 类的实例。|
| findClass(String name) | 查找名称为 name 的类，返回的结果是 java.lang.Class 类的实例。|
| findLoadedClass(String name) | 查找名称为 name 的已经被加载过的类，返回的结果是 java.lang.Class 类的实例。|
| resolveClass(Class<?> c) | 链接指定的 Java 类。|

注意：在JDK1.2之前，类加载尚未引入双亲委派模式，因此实现自定义类加载器时常常重写loadClass方法，提供双亲委派逻辑，从JDK1.2之后，双亲委派模式已经被引入到类加载体系中，自定义类加载器时不需要在自己写双亲委派的逻辑，因此不鼓励重写loadClass方法，而推荐重写findClass方法。
在Java中，任意一个类都需要由加载它的类加载器和这个类本身一同确定其在java虚拟机中的唯一性，即比较两个类是否相等，只有在这两个类是由同一个类加载器加载的前提之下才有意义，否则，即使这两个类来源于同一个Class类文件，只要加载它的类加载器不相同，那么这两个类必定不相等(这里的相等包括代表类的Class对象的equals()方法、isAssignableFrom()方法、isInstance()方法和instanceof关键字的结果)。例子代码如下：
```
/** 
     * 一、ClassLoader加载类的顺序 
     *  1.调用 findLoadedClass(String) 来检查是否已经加载类。 
     *  2.在父类加载器上调用 loadClass 方法。如果父类加载器为 null，则使用虚拟机的内置类加载器。 
     *  3.调用 findClass(String) 方法查找类。 
     * 二、实现自己的类加载器 
     *  1.获取类的class文件的字节数组 
     *  2.将字节数组转换为Class类的实例 
     * @author lei 2011-9-1 
     */  
    public class ClassLoaderTest {  
        public static void main(String[] args) throws InstantiationException, IllegalAccessException, ClassNotFoundException {  
            //新建一个类加载器  
            MyClassLoader cl = new MyClassLoader("myClassLoader");  
            //加载类，得到Class对象  
            Class<?> clazz = cl.loadClass("classloader.Animal");  
            //得到类的实例  
            Animal animal=(Animal) clazz.newInstance();  
            animal.say();  
        }  
    }  
    class Animal{  
        public void say(){  
            System.out.println("hello world!");  
        }  
    }  
    class MyClassLoader extends ClassLoader {  
        //类加载器的名称  
        private String name;  
        //类存放的路径  
        private String path = "E:\\workspace\\Algorithm\\src";  
        MyClassLoader(String name) {  
            this.name = name;  
        }  
        MyClassLoader(ClassLoader parent, String name) {  
            super(parent);  
            this.name = name;  
        }  
        /** 
         * 重写findClass方法 
         */  
        @Override  
        public Class<?> findClass(String name) {  
            byte[] data = loadClassData(name);  
            return this.defineClass(name, data, 0, data.length);  
        }  
        public byte[] loadClassData(String name) {  
            try {  
                name = name.replace(".", "//");  
                FileInputStream is = new FileInputStream(new File(path + name + ".class"));  
                ByteArrayOutputStream baos = new ByteArrayOutputStream();  
                int b = 0;  
                while ((b = is.read()) != -1) {  
                    baos.write(b);  
                }  
                return baos.toByteArray();  
            } catch (Exception e) {  
                e.printStackTrace();  
            }  
            return null;  
        }  
    }  
```
类加载器双亲委派模型是从JDK1.2以后引入的，并且只是一种推荐的模型，不是强制要求的，因此有一些没有遵循双亲委派模型的特例：(了解)
(1).在JDK1.2之前，自定义类加载器都要覆盖loadClass方法去实现加载类的功能，JDK1.2引入双亲委派模型之后，loadClass方法用于委派父类加载器进行类加载，只有父类加载器无法完成类加载请求时才调用自己的findClass方法进行类加载，因此在JDK1.2之前的类加载的loadClass方法没有遵循双亲委派模型，因此在JDK1.2之后，自定义类加载器不推荐覆盖loadClass方法，而只需要覆盖findClass方法即可。
(2).双亲委派模式很好地解决了各个类加载器的基础类统一问题，越基础的类由越上层的类加载器进行加载，但是这个基础类统一有一个不足，当基础类想要调用回下层的用户代码时无法委派子类加载器进行类加载。为了解决这个问题JDK引入了ThreadContext线程上下文，通过线程上下文的setContextClassLoader方法可以设置线程上下文类加载器。
JavaEE只是一个规范，sun公司只给出了接口规范，具体的实现由各个厂商进行实现，因此JNDI，JDBC,JAXB等这些第三方的实现库就可以被JDK的类库所调用。线程上下文类加载器也没有遵循双亲委派模型。
(3).近年来的热码替换，模块热部署等应用要求不用重启java虚拟机就可以实现代码模块的即插即用，催生了OSGi技术，在OSGi中类加载器体系被发展为网状结构。OSGi也没有完全遵循双亲委派模型。

##### 6.2、动态加载Jar && ClassLoader 隔离问题
动态加载Jar：
Java 中动态加载 Jar 比较简单，如下：
```
URL[] urls = new URL[] {new URL("file:libs/jar1.jar")};  
URLClassLoader loader = new URLClassLoader(urls, parentLoader);  
```
表示加载 libs 下面的 jar1.jar，其中 parentLoader 就是上面1中的 parent，可以为当前的 ClassLoader。

ClassLoader 隔离问题：
大家觉得一个运行程序中有没有可能同时存在两个包名和类名完全一致的类？
JVM 及 Dalvik 对类唯一的识别是 ClassLoader id + PackageName + ClassName，所以一个运行程序中是有可能存在两个包名和类名完全一致的类的。并且如果这两个”类”不是由一个 ClassLoader 加载，是无法将一个类的示例强转为另外一个类的，这就是 ClassLoader 隔离。 如 Android 中碰到如下异常
```
android.support.v4.view.ViewPager can not be cast to android.support.v4.view.ViewPager  
```
当碰到这种问题时可以通过 instance.getClass().getClassLoader(); 得到 ClassLoader，看 ClassLoader 是否一样。
 
加载不同 Jar 包中公共类：
现在 Host 工程包含了 common.jar, jar1.jar, jar2.jar，并且 jar1.jar 和 jar2.jar 都包含了 common.jar，我们通过 ClassLoader 将 jar1, jar2 动态加载进来，这样在 Host 中实际是存在三份 common.jar，如下图：
![](/image/14301963930_2f0f0fe8aa_o.png)
我们怎么保证 common.jar 只有一份而不会造成上面3中提到的 ClassLoader 隔离的问题呢，其实很简单，在生成 jar1 和 jar2 时把 common.jar 去掉，只保留 host 中一份，以 host ClassLoader 为 parentClassLoader 即可。

##### 6.3、常见问题
能不能自己写个类叫java.lang.System？
答案：通常不可以，但可以采取另类方法达到这个需求。 
解释：为了不让我们写System类，类加载采用委托机制，这样可以保证爸爸们优先，爸爸们能找到的类，儿子就没有机会加载。而System类是Bootstrap加载器加载的，就算自己重写，也总是使用Java系统提供的System，自己写的System类根本没有机会得到加载。
但是，我们可以自己定义一个类加载器来达到这个目的，为了避免双亲委托机制，这个类加载器也必须是特殊的。由于系统自带的三个类加载器都加载特定目录下的类，如果我们自己的类加载器放在一个特殊的目录，那么系统的加载器就无法加载，也就是最终还是由我们自己的加载器加载。

##### 6.4、线程上下文类加载器
线程上下文类加载器（context class loader）是从 JDK 1.2 开始引入的。类 java.lang.Thread中的方法 getContextClassLoader()和 setContextClassLoader(ClassLoader cl)用来获取和设置线程的上下文类加载器。如果没有通过 setContextClassLoader(ClassLoader cl)方法进行设置的话，线程将继承其父线程的上下文类加载器。Java 应用运行的初始线程的上下文类加载器是系统类加载器。在线程中运行的代码可以通过此类加载器来加载类和资源。

前面提到的类加载器的代理模式并不能解决 Java 应用开发中会遇到的类加载器的全部问题。Java 提供了很多服务提供者接口（Service Provider Interface，SPI），允许第三方为这些接口提供实现。常见的 SPI 有 JDBC、JCE、JNDI、JAXP 和 JBI 等。这些 SPI 的接口由 Java 核心库来提供，如 JAXP 的 SPI 接口定义包含在 javax.xml.parsers包中。这些 SPI 的实现代码很可能是作为 Java 应用所依赖的 jar 包被包含进来，可以通过类路径（CLASSPATH）来找到，如实现了 JAXP SPI 的 Apache Xerces所包含的 jar 包。SPI 接口中的代码经常需要加载具体的实现类。如 JAXP 中的 javax.xml.parsers.DocumentBuilderFactory类中的 newInstance()方法用来生成一个新的 DocumentBuilderFactory的实例。这里的实例的真正的类是继承自 javax.xml.parsers.DocumentBuilderFactory，由 SPI 的实现所提供的。如在 Apache Xerces 中，实现的类是 org.apache.xerces.jaxp.DocumentBuilderFactoryImpl。而问题在于，SPI 的接口是 Java 核心库的一部分，是由引导类加载器来加载的；SPI 实现的 Java 类一般是由系统类加载器来加载的。引导类加载器是无法找到 SPI 的实现类的，因为它只加载Java的核心库。它也不能代理给系统类加载器，因为它是系统类加载器的祖先类加载器。也就是说，类加载器的代理模式无法解决这个问题。

*使用线程上下文类加载器，可以在执行线程中抛弃双亲委派加载链模式，使用线程上下文里的类加载器加载类*

线程上下文类加载器正好解决了这个问题。如果不做任何的设置，Java应用的线程的上下文类加载器默认就是系统上下文类加载器。在SPI接口的代码中使用线程上下文类加载器，就可以成功的加载到SPI实现的类。线程上下文类加载器在很多 SPI 的实现中都会用到。

##### 6.5、Thread.currentThread().getContextClassLoader()与Test.class.getClassLoader()区别
Each class will use its own classloader to load other classes. So if ClassA.class references ClassB.class then ClassB needs to be on the classpath of the classloader of ClassA, or its parents.

The thread context classloader is the current classloader for the current thread. An object can be created from a class in ClassLoaderC and then passed to a thread owned by ClassLoaderD. In this case the object needs to use Thread.currentThread().getContextClassLoader() directly if it wants to load resources that are not available on its own classloader.


#### 7、另一种加载类的方法(Class.forName)
Class.forName是一个静态方法，同样可以用来加载类。该方法有两种形式：Class.forName(String name, boolean initialize, ClassLoader loader)和 Class.forName(String className)。第一种形式的参数 name表示的是类的全名；initialize表示是否初始化类；loader表示加载时使用的类加载器。第二种形式则相当于设置了参数 initialize的值为 true，loader的值为当前类的类加载器。Class.forName的一个很常见的用法是在加载数据库驱动的时候。如 Class.forName("org.apache.derby.jdbc.EmbeddedDriver").newInstance()用来加载 Apache Derby 数据库的驱动。

##### 7.1、Class.forName和ClassLoader.loadClass的比较:
Class的装载分了三个阶段，loading，linking和initializing，分别定义在The Java Language Specification的12.2，12.3和12.4。
Class.forName(className)实际上是调用Class.forName(className, true, this.getClass().getClassLoader())。注意第二个参数，是指Class被loading后是不是必须被初始化。
ClassLoader.loadClass(className)实际上调用的是ClassLoader.loadClass(name, false)，第二个参数指出Class是否被link。
区别就出来了。Class.forName(className)装载的class已经被初始化，而ClassLoader.loadClass(className)装载的class还没有被link。
一般情况下，这两个方法效果一样，都能装载Class。但如果程序依赖于Class是否被初始化，就必须用Class.forName(name)了。
例如，在JDBC编程中，常看到这样的用法，Class.forName("com.mysql.jdbc.Driver")，如果换成了getClass().getClassLoader().loadClass("com.mysql.jdbc.Driver")，就不行。
为什么呢？打开com.mysql.jdbc.Driver的源代码看看，
```
//
// Register ourselves with the DriverManager
//
static {
    try {
        java.sql.DriverManager.registerDriver(new Driver());
    } catch (SQLException E) {
        throw new RuntimeException("Can't register driver!");
    }
}
```
原来，Driver在static块中会注册自己到java.sql.DriverManager。而static块就是在Class的初始化中被执行。所以这个地方就只能用Class.forName(className)。

### 参考资料：
[深入理解Java类加载器(1)：线程上下文类加载器](http://blog.csdn.net/zhoudaxia/article/details/35824249)