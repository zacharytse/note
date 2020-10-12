[TOC]
#ClassLoader
java自带了3个类加载器，分别如下
- BootstrapClassLoader
最顶层的加载类，主要加载核心类库，%JRE_HOME%\lib下的rt.jar、resources.jar、charsets.jar和class等。
- ExtentionClassLoader 
扩展的类加载器，加载目录%JRE_HOME%\lib\ext目录下的jar包和class文件
- AppclassLoader
也称为SystemAppClass,用于加载当前应用的classpath的所有类
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
