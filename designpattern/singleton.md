###1、定义
####1.1、懒汉

####1.2、恶汉

####1.3、静态内部类

####1.4、枚举

####1.5、双重校验锁

###2、问题
有两个问题需要注意：
     1、如果单例由不同的类装载器装入，那便有可能存在多个单例类的实例。假定不是远端存取，例如一些servlet容器对每个servlet使用完全不同的类  装载器，这样的话如果有两个servlet访问一个单例类，它们就都会有各自的实例。
     2、如果Singleton实现了java.io.Serializable接口，那么这个类的实例就可能被序列化和复原。不管怎样，如果你序列化一个单例类的对象，接下来复原多个那个对象，那你就会有多个单例类的实例。
对第一个问题修复的办法是：
```
private static Class getClass(String classname)      
                                           throws ClassNotFoundException {     
        ClassLoader classLoader = Thread.currentThread().getContextClassLoader();     
        
        if(classLoader == null)     
           classLoader = Singleton.class.getClassLoader();     
        
        return (classLoader.loadClass(classname));     
     }     
}  
```
对第二个问题修复的办法是： 
```
public class Singleton implements java.io.Serializable {     
     public static Singleton INSTANCE = new Singleton();     
        
     protected Singleton() {     
          
     }     
     private Object readResolve() {     
              return INSTANCE;     
        }    
}  
```

###3、案例









参考：

- [单例模式的5种写法](http://www.blogjava.net/kenzhh/archive/2016/03/28/357824.html)
- 