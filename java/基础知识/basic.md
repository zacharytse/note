[TOC]
# String
```java{.line-numbers}
String str1 = "abc";
String str2 = "abc";
System.out.println(str1 == str2);
```
此时返回的true。
解释:在java中的字符串常量在jdk7以后都是放在java堆的字符串常量池中。str1和str2使用的都是同一个常量，同一个的常量的引用必然相同，所以是true
```java{.line-numbers}
String str1 = new String("abc");
String str2 = new String("abc");
System.out.println(str1 == str2);
```
此时返回的是false
这里因为str1和str2都是新创建的String对象，不同的对象引用必然不同，所以返回false
```java{.line-numbers}
 String str1 = new StringBuilder("计算机").append("软件").toString();
        System.out.println(str1.intern() == str1);
```
这里返回的是true
先来解释下intern()函数。
> intern()在使用时，会先从字符串常量池中找一个等于此String对象的字符串，如果找到，则返回该String对象的引用，否则会先把此String对象包含的字符串添加到常量池中，再返回该对象的引用。

"计算机软件"这个字符串之前没有出现过，所以intern()会把这个字符串加入到常量池中，并返回str1指向的String对象的引用，因此返回true

```java{.line-numbers}
String str2 = new StringBuilder("ja").append("va").toString();
        System.out.println(str2.intern() == str2);
```
这里返回的却是false
因为"java"字符串在程序开始运行时，会先由sun.misc.Version这个类率先加载到常量池中。所以intern()返回的是在sun.misc.Version这个类加载"java"进常量池时产生的引用，显然和str2是不等的

# Java Object类方法
Object中有13个方法
1. 构造及注册native方法
   - Object()(Object的构造方法)
   - registerNatives()向本地注册native方法，它会向本地注册hashCode，clone等方法
2. 拷贝函数
   - clone()它返回的对象是浅拷贝对象
3. 获取类对象的方法
   - getClass()
4. 对象判等的方法
   - equals(),Object类中的equal()是通过==实现的
5. 返回当前对象的hash值
   - hashCode().同一对象在一次运行中返回的hash值是相同的，equal()定义相等的对象返回的hash值也应该是相同的。hashCode()的作用是增强了哈希表的性能。例如对于Set,在对象判等时，可以借助hashCode进行判等，无需进行一次set的遍历

6. toString()返回该对象的字符串表示
7. 线程相关的
   - wait()
   - notify()
   - notifyall()

8. 回收资源相关的
   - finialize()

# equals的实现
1. 若两个对象的引用相同，则直接返回true，否则执行步骤2
2. 判断传入的Object对象和this的类型是否一致，不一致的话返回false,否则执行步骤3
3. 对传入的Object对象强制转换为当前对象的类型
4. 两者进行值比较

# String,StringBuilder,StringBuffer
- 相同点
  - 都是final类，不允许被继承

- 不同点
  - String长度是不可变的，StringBuilder和StringBuffer长度是可以改变的
  - StringBuffer是线程安全的，StringBuilder不是线程安全的

## String和StringBuffer
1. 因为String是不可变的对象，所以对String做操作时都会产生新的String对象，所以可能会影响GC的效率
2. 使用StringBuffer类，每次都会对StringBuffer对象本身进行操作，而不是生成新的对象，所以一般使用StringBuffer
3. 某些时候，对String的字符串拼接会被java compile成StringBuffer对象的拼接，这时候使用String并不一定比StringBuffer慢

## StringBuilder
可以看成是StringBuffer的单线程简易版，在单线程环境下，比StringBuilder快

# Exception，Error和Throwable
1. Throwable是java语言中所有错误和异常的超类，Exception和Error是它的子类
2. Error是Throwable的子类，用于指示合理的程序不应该试图捕获的严重问题。如OutOfMemoryError以及StackOverFlowError
3. Exception及其子类是Throwable的一种形式，它指出了合理的程序应该要捕获的条件，并对其进行处理
   1. 运行时异常(RunTimeException)，如NullPointerException、IndexOutOfBoundsException
   2. 非运行时异常是RunTimeException以外的异常，如IOException或者用户自定义的异常，这种异常一般是要求进行捕获的，否则会报错