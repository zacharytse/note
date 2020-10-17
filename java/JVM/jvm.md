[TOC]
#java编译的过程
![](https://gitee.com/zacharytse/image/raw/master/img/20201017145723.png)
可以看到是由源代码经过词法分析器和语法分析器以及字节码生成器最后得到了JVM字节码
##JVM字节码的执行
字节码执行是由JVM引擎来完成的，流程图如下:
![](https://gitee.com/zacharytse/image/raw/master/img/20201017145845.png)
##Class文件的组成
- 结构信息。
  包括class文件格式版本号及各部分的数量与大小的信息
- 元数据
  对应于java源码中声明与常量的信息。包含类/继承的超类/实现的接口的声明信息、域与方法声明信息和常量池
- 方法信息
  对应于java源码中语句和表达式对应的信息。包含字节码，异常处理表，求值栈与局部变量区大小、求值栈的类型记录、调试符号信息
```{.line-numbers}
Classfile /E:/X.class
  Last modified 2020-10-15; size 372 bytes
  MD5 checksum be826009b32e9ba300ff43405c032137
  Compiled from "X.java"
public class X
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #4.#16         // java/lang/Object."<init>":()V
   #2 = Methodref          #3.#17         // X.bar:()V
   #3 = Class              #18            // X
   #4 = Class              #19            // java/lang/Object
   #5 = Utf8               <init>
   #6 = Utf8               ()V
   #7 = Utf8               Code
   #8 = Utf8               LineNumberTable
   #9 = Utf8               LocalVariableTable
  #10 = Utf8               this
  #11 = Utf8               LX;
  #12 = Utf8               foo
  #13 = Utf8               bar
  #14 = Utf8               SourceFile
  #15 = Utf8               X.java
  #16 = NameAndType        #5:#6          // "<init>":()V
  #17 = NameAndType        #13:#6         // bar:()V
  #18 = Utf8               X
  #19 = Utf8               java/lang/Object
{
  public X();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   LX;

  public void foo();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokevirtual #2                  // Method bar:()V
         4: return
      LineNumberTable:
        line 3: 0
        line 4: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   LX;

  public void bar();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=0, locals=1, args_size=1
         0: return
      LineNumberTable:
        line 6: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       1     0  this   LX;
}
```
#ClassLoader
##作用
加载Class,负责将Class的字节码形式转换成内存形式的Class对象。字节码可以来自磁盘文件*.class,也可以是jar包里的*.class，也可以是来自远程服务器提供的字节流。字节码的本质是一个字节数组[]byte
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X2pwZy9jV2VYMmlibEx2aWFHbmJDRHRVemVkYUFYZHZ4Y0s4TXlod1pJTUQ0aWNLWkY4Zk1vNGNwT2VVZjFsQnNMd1BiQnkzTGNkZ2lhYlFVVUpVRXJPb28wUE5Wc2cvNjQw?x-oss-process=image/format,png)
java自带了3个类加载器，分别如下
- BootstrapClassLoader
最顶层的加载类，主要加载核心类库，%JRE_HOME%\lib下的rt.jar、resources.jar、charsets.jar和class等。
- ExtentionClassLoader 
扩展的类加载器，加载目录%JRE_HOME%\lib\ext目录下的jar包和class文件，比如swing,js,xml解析器等等
- AppclassLoader
也称为SystemAppClass,用于加载当前应用的**classpath**的所有类，即加载用户自定义的类
#延迟加载
JVM运行时不会一次性加载全部的类，在遇到不认识的新类时，会调用ClassLoader加载这些类，加载完成后，就会将Class对象存在ClassLoader中。
比如调用某个类的静态方法时，类肯定时需要被加载的，但是实例字段是不需要被加载的，只有在创建这个类的实例时，才需要加载这些字段(把这些字段从字节码的形式加载成内存中的class形式)
网络上静态文件服务器提供的jar包和class文件，jdk内置了一个URLClassLoader，用户只需要指定url给构造器，就可以使用URLClassLoader来加载远程类库。
AppClassLoader可以由ClassLoader类提供的静态方法getSystemClassLoader()得到，即系统类加载器。
##加载顺序
1. BootstrapClassLoader
2. ExtentionClassLoader
3. AppClassLoader
##父加载器
每个类加载器都有一个父加载器，比如加载Test.class是由AppClassLoader完成，则AppClassLoader也有一个父加载器，可以通过ClassLoader下的getParent方法获得
测试代码如下:
```java {.line-numbers}
ClassLoader cl = Test.class.getClassLoader();
		
System.out.println("ClassLoader is:"+cl.toString());
System.out.println("ClassLoader\'s parent is:"+cl.getParent().toString());
```
运行结果如下：
```java{.line-numbers}
ClassLoader is:sun.misc.Launcher$AppClassLoader@73d16e93
ClassLoader's parent is:sun.misc.Launcher$ExtClassLoader@15db9742
```
即AppClassLoader的父加载器是ExtClassLoader
**父加载器不是父类**
继承关系如下图所示
![](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMjExMTEyNzU0MTk3?x-oss-process=image/format,png)
getParent方法在ClassLoader.java中，代码如下
```java{.line-numbers}
public abstract class ClassLoader {

// The parent class loader for delegation
// Note: VM hardcoded the offset of this field, thus all new fields
// must be added *after* it.
private final ClassLoader parent;
// The class loader for the system
    // @GuardedBy("ClassLoader.class")
private static ClassLoader scl;

private ClassLoader(Void unused, ClassLoader parent) {
    this.parent = parent;
    ...
}
protected ClassLoader(ClassLoader parent) {
    this(checkCreateClassLoader(), parent);
}
protected ClassLoader() {
    this(checkCreateClassLoader(), getSystemClassLoader());
}
public final ClassLoader getParent() {
    if (parent == null)
        return null;
    return parent;
}
public static ClassLoader getSystemClassLoader() {
    initSystemClassLoader();
    if (scl == null) {
        return null;
    }
    return scl;
}

private static synchronized void initSystemClassLoader() {
    if (!sclSet) {
        if (scl != null)
            throw new IllegalStateException("recursive invocation");
        sun.misc.Launcher l = sun.misc.Launcher.getLauncher();
        if (l != null) {
            Throwable oops = null;
            //通过Launcher获取ClassLoader
            scl = l.getClassLoader();
            try {
                scl = AccessController.doPrivileged(
                    new SystemClassLoaderAction(scl));
            } catch (PrivilegedActionException pae) {
                oops = pae.getCause();
                if (oops instanceof InvocationTargetException) {
                    oops = oops.getCause();
                }
            }
            if (oops != null) {
                if (oops instanceof Error) {
                    throw (Error) oops;
                } else {
                    // wrap the exception
                    throw new Error(oops);
                }
            }
        }
        sclSet = true;
    }
}
}
```
getParent()返回的是一个ClassLoader对象
#Bootstrap ClassLoader是由C++编写
其本身是虚拟即的一部分，不是一个java类。JVM启动时通过Bootstrap类加载器加载rt.jar等核心jar包中的class文件。
#ClassLoader传递性
虚拟机在选择ClassLoader时，会使用当前正在调用的方法所属的class对象中的ClassLoader来加载。
比如虚拟机调度main方法时，此时是用户自定义的类，所以会使用AppClassLoader来加载这个类。
而对于那些延迟加载的类来说，它们也只能由AppClassLoader来加载，这就是ClassLoader传递性
#双亲委托
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X2pwZy9jV2VYMmlibEx2aWFHbmJDRHRVemVkYUFYZHZ4Y0s4TXloSXEwZnBENHNpYzFnV1Q4RkQzaWJjbkJOWExPaWFxSDExM1hmY3hZeHBVYmdhTzY0NHJJQk9oMktBLzY0MA?x-oss-process=image/format,png)
AppClassLoader在加载一个未知的类名时，不会先查找Classpath,而是先委托ExtensionClassLoader来加载，而ExtensionClassLoader加载时，同样也不会立即搜寻ext路径，而是会先委托BootstrapClassLoader来加载。每一个ClassLoader对象内部都有一个parent属性指向它的父加载器。
这里要注意，因为bootstrapClassLoader是由c编写的，获取不到引用，所以所有由bootstrapClassLoader加载的class，他们的ClassLoader字段都是null。同样的，ExtensionClassLoader的parent字段也是null
#Class.forName
经常会用它来加载驱动类
```java
Class.forName("com.mysql.cj.jdbc.Driver");
```
因为mysql驱动里有一个静态代码块。静态代码块会在类被加载的时候执行，所以这个静态代码块在Class.forName加载Driver时被执行。这个静态代码块会将mysql驱动实例注册到全局的jdbc驱动管理器里
```java{.line-numbers}
class Driver {
 static {
    try {
      java.sql.DriverManager.registerDriver(new Driver());
    } catch (SQLException E) {
      throw new RuntimeException("Can't register driver!");
   }
 }
 ...
}
```
forName方法也是根据调用者的ClassLoader来加载。但是也可以自己指定使用哪个ClassLoader
```java{.line-numbers}
Class<?> forName(String name, 
        boolean initialize,
        ClassLoader cl)
```
#自定义类加载器
ClassLoader中的三个方法
- loadClass()
loadClass()是加载目标类的入口，首先会检查当前ClassLoader以及它的双亲是否已经加载了目标类，如果当前ClassLoader不能加载，则会让双亲尝试加载，如果双亲都不行，就会调用findClass()使用自定义的加载器来加载目标类
- findClass()
  是需要重写的
- defineClass()

```java{.line-numbers}
//pesuecode
class ClassLoader {
 // 加载入口，定义了双亲委派规则
 Class loadClass(String name) {
 // 是否已经加载了
 Class t = this.findFromLoaded(name);
 if(t == null) {
 // 交给双亲
 t = this.parent.loadClass(name)
 }
 if(t == null) {
 // 双亲都不行，只能靠自己了
 t = this.findClass(name);
 }
 return t;
 }
 
 // 交给子类自己去实现
 Class findClass(String name) {
 throw ClassNotFoundException();
 }
 
 // 组装Class对象
 Class defineClass(byte[] code, String name) {
 return buildClassFromCode(code, name);
 }
}
class CustomClassLoader extends ClassLoader {
 Class findClass(String name) {
 // 寻找字节码
 byte[] code = findCodeFromSomewhere(name);
 // 组装Class对象
 return this.defineClass(code, name);
 }
}
```
不要随便覆盖loadClass()方法，否则会破坏原来默认加载内置的核心类库的功能。
使用自定义加载器时，要明确好父加载器，如果设为null，则表明父加载器是根加载器(bootstrapClassLoader)
```java{.line-numbers}
// ClassLoader 构造器
protected ClassLoader(String name, ClassLoader parent);
```
##Class.forName vs ClassLoader.loadClass
它俩都能用来加载目标类