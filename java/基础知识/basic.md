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