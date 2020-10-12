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
