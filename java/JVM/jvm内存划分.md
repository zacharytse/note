[TOC]
#JVM整体架构
![](https://gitee.com/zacharytse/image/raw/master/img/dfafdsgfsdg.png)
##运行时数据区
![](https://gitee.com/zacharytse/image/raw/master/img/20201015093828.png)
运行时数据区主要分为两块，一块是单线程私有的，而另一块是线程共享的。
###线程私有区域
线程私有区域中有3块，分别是虚拟机栈，本地方法栈，程序计数器。
- 虚拟机栈
  程序在运行时，调用的方法以及方法中定义的局部变量都会打包成一个栈帧，压入虚拟机栈中。然后在方法区中，弹出栈帧，也就是方法的执行
####栈帧
![](https://gitee.com/zacharytse/image/raw/master/img/20201015094823.png)
即虚拟机栈中的元素。
栈帧由四部分组成——操作数栈，局部变量表，完成出口，动态连接
- 操作数栈
  每一个局部变量在创建时，都会压入操作数栈中。当执行具体的操作命令时，会从操作数栈中取出操作数用于执行相应的操作
- 动态连接
java运行是进行动态连接的，所有的类在加载后，其中的方法都是使用的符号连接，只有在调用这个方法时，才会由符号连接解析为直接引用，即为动态连接(对应的直接地址)
- 局部变量表
  存储局部变量的引用
- 完成出口
 记录该栈帧所对应的方法在完成后的跳转地址,例如
 ```java{.line-numbers}
 public class A{
     public void a(){

     }
     public void b(){
         a();
         xxxx
     }
 }
 ```
 假设当前栈帧对应了a方法，那么出口地址记录的就是第7行的代码所对应的指令的地址。执行跳转命令时，程序计数器(PC)也要指向到这条指令

- 本地方法栈
  本地方法栈是为了调用native method
- 程序计数器
始终指向下一条要执行的指令
###线程共享区域
- 方法区
  弹出虚拟机栈中的栈中，并通过执行引擎去执行这些方法
- 堆
  存储对象
###动态连接详解
例如对于以下代码
```java{.line-numbers}
public class X {
  public void foo() {
    bar();
  }

  public void bar() { }
}
```
经过命令
```
javac -g X.java
javap -p -v X
```
得到如下的代码
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
首先看foo()方法里的一条指令
```
1: invokevirtual #2 
```
invokevirtual对应的操作码是0xB6，所以对应的实际编码为
```
[B6] [00 02]
```
然后接着去找下标为2的常量池项
```
 #2 = Methodref          #3.#17         // X.bar:()V
```
V是virtual的意思
实际编码为
```
[0A] [00 03] [00 11]
```
0x0A是CONSTANT_Methodref_info的tag，17的16进制数是11，3，11对应的是class_index和name_and_type_index。接着根据深度优先搜索的原则，并将这些引用用树的形式表示出来
```
#2 Methodref X.bar:()V
     /                     \
#3 Class X       #17 NameAndType bar:()V
    |                /             \
#18 Utf8 X    #13 Utf8 bar     #6 Utf8 ()V
```
这些字符串就是符号引用
**直接引用**
以Sun Classic VM为例
```
HObject             ClassObject
                       -4 [ hdr            ]
--> +0 [ obj     ] --> +0 [ ... fields ... ]
    +4 [ methods ] \
                    \         methodtable            ClassClass
                     > +0  [ classdescriptor ] --> +0 [ ... ]
                       +4  [ vtable[0]       ]      methodblock
                       +8  [ vtable[1]       ] --> +0 [ ... ]
                       ... [ vtable...       ]
```
Sun Classic VM采用的是句柄的方式，HObject是指向对象的句柄，ClassObject是对应的对象的结构体，methods指向相应的方法，所有的引用都必须通过句柄来获取
例如调用foo方法时，发现上面的符号引用还没有解析。进行解析时，会先通过class_index找到类名，然后找到类的ClassClass结构体。然后通过name_and_type_index找到方法名和方法描述符，并在ClassClass结构体上记录的方法列表里找到匹配的那个methodblock。
假设foo对应的methodblock是0x45672300，则常量池#2(#2 = Methodref          #3.#17         // X.bar:()V)的内容会被替换为
```
[00 23 67 45]
```
这里是因为采用了低地址优先，这里的methodblock就是一个直接引用
invokevirtual在解析前，对应的内容是
```
[B6] [00 02]
```
在解析之后，代码会变为
```
[D6] [06] [01]
```
D6表示opcode从invokevirtual变为了invokevirtual_quick，表示指令解析完毕了。[06][01]两个字节分别表示虚方法表的下标以及方法的参数个数(1个参数是因为包含了this)
####总结
动态连接使用了和C++实现多态类似的方法，都是借助虚函数表来实现多态
##JVM缓存架构与计算机架构的对比
![](https://gitee.com/zacharytse/image/raw/master/img/1602739079(1).png)


