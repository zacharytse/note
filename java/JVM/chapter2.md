[TOC]
# 内存模型
![](https://gitee.com/zacharytse/image/raw/master/img/fdfsd.png)
## 程序计数器
这块空间较小，可以看作是当前线程所执行的字节码的行号指示器
为了线程切换后，还能恢复到正确的位置接着执行这条线程的字节码，每一个线程都会有一个独立的程序计数器。
**如果执行的是java方法，那么这个计数器记录的是正在执行的虚拟机字节码指令的地址。如果执行的是Native方法，这个计数器的值应该为空**
**这块区域也是唯一一块没有定义OutOfMemoryError情况的区域**
## java虚拟机栈
线程私有，生命周期与线程相同。
描述的是java方法执行的线程内存模型。
该模型的描述如下：
每一个方法被执行的时候，java虚拟机都会同步创建一个栈帧存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法被调用直至执行完毕，都对应着一个栈帧在虚拟机栈中入栈到出栈的过程。
### 局部变量表
存放的是基本类型的数据，以及对象的引用类型以及returnAddress类型(指向了一条字节码指令的地址)
它们在局部变量表中的存储是以局部变量槽(slot)来表示。64bit的long和double会占据2个slot，其余的数据都只占用一个slot
**这里要注意，运行期间局部变量表大小不变，是指局部变量槽的数量不变，一个slot多大，是由具体的虚拟机实现来决定的**

> java虚拟机栈这块区域运行时可能会抛出两种异常，一种是线程请求的栈深度大于虚拟机所允许的深度，将抛出**StackOverFlowError**异常。如果java虚拟机栈容量可以动态扩展，在栈扩展时无法申请到足够的内存会抛出OutOfMemoryError异常

动态扩展:HotSpot虚拟机的容量是不可以被扩展的，而之前的Classic虚拟机是可以的。因此HotSpot虚拟机是不会因为虚拟机栈的动态扩展而抛出OutOfMemoryError异常，但如果线程申请栈空间失败，仍然会抛出OutOfMemoryError异常

## java堆
在虚拟机启动时创建，这块区域唯一的作用就是存放对象实例。
java堆在物理上可以是不连续的区域，但在逻辑上应该是连续的。
当java堆中没有内存进行对象实例的分配，也没有空间可以扩展堆时，就会抛出OutOfMemoryError异常
## 方法区
存储已被虚拟机加载的类型信息，常量，静态变量，即时编译器编译后的代码缓存等数据。在《java虚拟机规范》中把方法区描述为堆的一个逻辑部分，但是它也有一个别名——“非堆”，用于和java堆做区分。
同样地，如果方法区无法满足新的内存分配需求时，就会抛出OutOfMemoryError异常
### 运行时常量池
是方法区的一部分。Class文件中的常量池表在类加载后会存放到方法区的运行时常量池中。
**常量池表用于存放编译器生成的各种字面量和符合引用**
运行时常量池具备动态性，不仅仅是class文件中的常量可以放入到运行时常量池中，运行期间产生的常量也可以放进去
*因为运行时常量池是方法区的一部分，所以其大小受到方法区大小的限制，当其不能再申请到内存的时候，会抛出OutOfMemoryError异常*
## 直接内存
不属于java虚拟机管理的一块区域。
在jdk1.4中新加入的NIO，使用native函数库直接分配堆外内存，然后通过java堆中的DirectByteBuffer对象作为这块内存的引用进行操作。
但如果java堆内存和直接内存的总大小超过了机器可分配的内存大小，那么在动态分配时，必然会产生OutOfMemoryError异常
![](https://gitee.com/zacharytse/image/raw/master/img/20201026154836.png)
# 对象创建的过程
1. 类加载检查
   java虚拟机遇到一条new指令时，会先检查new指令的参数能否在常量池中找到一个对应类的符号引用，并且再检查下这个类是否已经被加载，解析和初始化过，如果没有，对这个类进行一次加载。
2. 为新生对象分配内存
   每一个对象所需要分配的内存大小在类加载完成之后，就已经确定了。
   为其分配所需的内存空间共有两种方法
   - 指针碰撞法
      简单的使用一个指针指向空闲区域和已分配区域的交界点，在分配内存时，把指针向后移动相应的位置即可。
      这种方法适用于规整的java堆
   - 空闲列表
  虚拟机维护一个列表，这个列表中记录了哪些内存是可用的，在分配的时候从这个列表中找出足够大的空间分配给对象即可。这种方式主要适合不规整的java堆。

  要注意的是java堆是否规整，要看其使用的垃圾收集器是否带有空间压缩整理的功能。所以如果使用Serial,ParNew等带有空间压缩整理功能的收集器时，可以直接使用指针碰撞法来分配内存。而如果使用的是CMS这种基于清除算法的收集器时，理论上只能采用空闲列表来分配。
  > CMS为了在多数情况下分配的更快，设计了一个叫做Liner Allocation Buffer的缓冲区。这块缓冲区是通过空闲列表拿到的，在这块缓冲区内部可以使用指针碰撞法

3. 线程安全
   在多线程环境下，即使是指针碰撞法，也有可能出现给A对象分配内存，指针还没来得及移动，B对象也使用这个指针来分配内存。解决这种问题的方案有两种。
   - 采用CAS配上失败重试保证更新操作的原子性(虚拟机实际使用)
   - 把内存分配动作按照线程划分在不同的空间进行。即为每一个线程在java堆中都预先分配一小块区域，这块区域称为本地线程分配缓冲(Thread Local Allocation Buffer,TLAB)。线程要分配内存时，直接从自己的这块区域中去分配。只有本地缓冲区用完了，分配新的缓冲区时，才进行同步锁定。
   虚拟机是否使用TLAB，使用参数-XX:+/-UseTLAB来设定
4. 对象初始化
   内存分配完成后，虚拟机会将分配到的内存空间(**不包括对象头**)都初始化为零值。如果使用TLAB的话，这一步可以和第三步合并
5. 对对象进行一些必要的设置
   例如这个对象是哪个类的实例，如何才能找到类的元数据信息，对象的哈希码，对象的GC分代年龄等信息，这些信息都会存放到对象的对象头之中
6. 完成对象的构建
   调用对象的构造函数，完成最后的设置   
   
具体代码实例可以参照书p49-p50
# 对象的内存布局
对象在堆内存中的存储布局分为三个部分
- 对象头
- 实例数据
- 对齐填充
![](https://gitee.com/zacharytse/image/raw/master/img/20201026161846.png)
## 对象头
对象头包含两部分信息
- 存储对象自身的运行时数据
  如哈希码，GC分代年龄，锁状态标志，线程持有的锁，偏向线程ID，偏向时间戳等。
  这部分数据在32位虚拟机中占32bit，在64位虚拟机中，占64bit。
  官方也称其为"Mark Word"
  它被设计成一种动态的数据结构，这是为了在极小的空间内存储尽量多的数据，根据对象状态复用自己的存储空间
- 类型指针
  对象指向它的类型元数据的指针，虚拟机通过这个指针来确定该对象是哪个类的实例
## 实例数据
存储对象真正的数据。**无论是在父类继承下来的**，还是在子类中定义的字段都必须记录下来。
它的存储顺序受到虚拟机分配策略参数(-XX:FieldsAllocationStyle)和字段在源码中定义顺序的影响。
HotSpot默认的分配顺序为longs/doubles、ints、shorts/chars、bytes/booleans、oops(Ordinary Object Pointers,OOPs)。即相同宽度的字段总是被分配到一起。
父类中定义的变量会出现在子类之前。
如果-XX:CompactFields参数值为true，子类中较窄的变量也允许插入父类变量的空隙之中。
## 对齐填充
因为HotSpot要求对象的起始地址必须是8字节的整数倍，所以对象的大小也就必须是8的整数倍，如果对象大小不满足条件，就会通过对齐填充使其大小达到8的整数倍
# 访问对象
java对象会通过栈上的reference数据来访问堆中的对象。具体的访问方式有使用句柄和直接指针两种
- 句柄访问
  java堆中会划出一块区域作为句柄池，reference中存储的是对象的句柄地址。句柄中包含了对象的实例数据和类型数据各自具体的地址。
  ![](https://gitee.com/zacharytse/image/raw/master/img/20201029163907.png)
- 直接指针访问
  reference中存储的是对象的地址，但要考虑对象类型地址的存放。所以如果要访问对象类型的话，就需要一次间址的开支
  ![](https://gitee.com/zacharytse/image/raw/master/img/20201029164413.png)
**二者的比较**
使用句柄访问，在对象被移动时，只需要改变句柄中对象实例数据的指针即可。
使用直接指针访问，节省了一次指针定位的开销。
# 内存溢出的实例
## java堆溢出
在测试时，要将堆的最小值与最大值设为一样，这样可以避免堆的自动扩展
-XX:+HeapDumpOnOutOfMemoryError可以让虚拟机在出现内存溢出异常的时候，dump出当前的内存堆转储快照方便事后分析
```java{.line-numbers}
package com.xcq;

import java.util.ArrayList;
import java.util.List;

public class Main {
    //-Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
    static class OOMObject{

    }
    public static void main(String[] args) {
	// write your code here
        List<OOMObject> list = new ArrayList<>();
        while(true){
            list.add(new OOMObject());
        }
    }
}
```
```
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid13000.hprof ...
Heap dump file created [28172002 bytes in 0.494 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3210)
	at java.util.Arrays.copyOf(Arrays.java:3181)
	at java.util.ArrayList.grow(ArrayList.java:265)
	at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:239)
	at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:231)
	at java.util.ArrayList.add(ArrayList.java:462)
	at com.xcq.Main.main(Main.java:14)
```
## 虚拟机栈和本地方法栈溢出
存在两种异常
1. 当线程请求的栈深度大于虚拟机所允许的最大深度，将抛出StackOverflowError异常
2. 如果虚拟机的栈内存允许动态扩展，当扩展栈无法申请到足够的内存时，将抛出OutOfMemoryError异常
对于第一种情况的测试
```java{.line-numbers}
package com.xcq;

/**
 * -Xss128k,限制栈容量
 */
public class JavaVMStackSOF {

    private int stackLength = 1;

    public void stackLeak(){
        stackLength++;
        stackLeak();
    }
    
    public static void main(String[] args) {
        JavaVMStackSOF stackSOF = new JavaVMStackSOF();
        try {
            stackSOF.stackLeak();
        }catch (Throwable e){
            System.out.println("stack length:" + stackSOF.stackLength);
            throw e;
        }
    }
}
```
```
stack length:995
Exception in thread "main" java.lang.StackOverflowError
	at com.xcq.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:8)
	at com.xcq.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:9)
	at com.xcq.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:9)
	at com.xcq.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:9)
   .....
```
不停的创建线程，因为每一个线程都会有自己的栈空间，所以最终会因为线程数量过多，使得虚拟机栈没有足够的空间分配内存给线程的栈空间，会出现OutOfMemoryError。
> 分配给线程的栈空间越大，这样可以创建的线程数也就越少，越容易出现OutOfMemoryError

## 方法区和运行时常量池溢出
字符串常量池溢出jdk7以后一般不太可能，因为jdk7以后，字符串常量池被挪到了堆中。
方法区因为用来存储类名，访问修饰符，字段描述，方法描述，常量池等，所以如果产生了太多的类，就有可能造成方法区溢出。
方法区的溢出一般是由于在运行时产生了大量动态类
在jdk8之后，因为将永久代替换为了元空间，所以方法区也一般不会溢出了
> 元空间不在虚拟机中，而是使用本地内存，因此默认情况下，元空间的大小仅受本地内存的限制

> 为什么要将永久代从方法区挪到元空间中

1. 字符串放在永久代，会有内存溢出以及性能问题
2. 类以及方法的信息很难确定大小，所以对于永久代的大小指定很困难，太小，会造成永久代溢出，太大的话，可分配的堆空间就变少了，因此老年代可分配空间也会变少，最终可能会造成老年代的溢出
3. 永久代中因为也存放了一些类的信息，GC时同样需要考虑，所以为GC带来了不必要的复杂度，并且回收效率偏低

## 本机直接内存溢出
指定直接内存容量的大小(-XX:MaxDirectMemorySize)。如果不指定的话，默认是和java堆的内存大小是一致的
```java{.line-numbers}
package com.xcq;

import sun.misc.Unsafe;

import java.lang.reflect.Field;

/**
 * -XX:MaxDirectMemorySize=10M -Xmx20M -XX:+HeapDumpOnOutOfMemoryError
 */
public class DirectMemoryOOM {

    private static final int _1MB = 1024 * 1024;
    public static void main(String[] args) throws IllegalAccessException {
        Field unsafeField = Unsafe.class.getDeclaredFields()[0];
        unsafeField.setAccessible(true);
        Unsafe unsafe = (Unsafe) unsafeField.get(null);
        while (true){
            unsafe.allocateMemory(_1MB);
        }
    }
}
```
```
Exception in thread "main" java.lang.OutOfMemoryError
	at sun.misc.Unsafe.allocateMemory(Native Method)
	at com.xcq.DirectMemoryOOM.main(DirectMemoryOOM.java:15)
```
可以发现Heap dump文件没有异常情况。所以如果Heap dump没有什么问题，但程序又间接或直接使用了DirectMemory(例如NIO)，那就有可能是直接内存溢出