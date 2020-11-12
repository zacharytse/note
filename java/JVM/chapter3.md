[TOC]
# 两种确定垃圾的算法
## 引用计数法
每当有一个地方引用它，引用计数就加一，当引用失效时，引用计数就减1。当计数器为0时，对象就不能再使用了。
### 循环引用
如果对象A持有了对象B的引用，对象B也持有对象A的引用。这时即使将A和B置为null，但因为堆上对象A和B始终存储着对方的引用，所以A,B的引用计数仍然为1，无法被回收
```java{.line-numbers}
package com.xcq.gc;

public class ReferenceCountingGC {
    public Object instance = null;

    private static final int _1MB = 1024 * 1024;

    private byte[] bigSize = new byte[2 * _1MB];

    public static void main(String[] args) {
        ReferenceCountingGC objA = new ReferenceCountingGC();
        ReferenceCountingGC objB = new ReferenceCountingGC();
        objA.instance = objB;
        objB.instance = objA;

        objA = null;
        objB = null;

        System.gc();
    }
}
```
```
[GC (System.gc())  7373K->752K(123904K), 0.0014725 secs]
[Full GC (System.gc())  752K->607K(123904K), 0.0048972 secs]
```
从日志中可以看到7373K->752K，说明jvm使用的不是引用计数算法判断对象的可回收性
## 可达性分析算法
定义了一系列的GC Roots的根对象，从这些根对象出发，如果某个对象是不可达的，那么说明这个对象是可以被回收的
![](https://gitee.com/zacharytse/image/raw/master/img/20201030203336.png)
图中的obj5,6,7到GC Roots是不可达的，所以会被判定为可回收的对象
### 可作为GC Roots的对象
- 虚拟机栈中引用的对象。例如局部变量、临时变量等
- 方法区中类静态属性引用的对象，例如引用类型的静态变量
- 方法区中常量引用的对象
- JNI引用的对象
- 虚拟机内部的引用，例如一些Class对象，常驻的异常对象，还有系统类加载器
- 被同步锁(synchronized关键字)持有的对象
- 反映java虚拟机内部情况的JMXBean,JVMTI中注册的回调，本地代码缓存等

## 对于引用的认识
在jdk1.2之前，引用被直接定义为某个对象的起始地址。
但在1.2之后，java对引用进行了扩充，将引用分为了强引用(Strongly Reference),软引用(Soft Reference),弱引用(Weak Reference)和虚引用(Weak Reference)
- 强引用
  是最常用的引用，类似"Object obj = new Object()"
- 软引用
  描述一些还有用，但非必须的对象。被软引用关联的对象，在系统要发生内存溢出异常前，会把这些对象列进回收范围之中进行第二次回收。在这次回收之后，如果还是没有足够的内存，就会抛出异常。在jdk1.2版后提供了SoftReference类来提供软引用
- 弱引用
  描述非必须对象，但比软引用强度还要弱，它们只能活到下一次垃圾收集发生为止。在jdk1.2版之后提供了WeakReference类来提供弱引用
- 虚引用
  称为"幽灵引用"或者"幻影引用"。是最弱的一种引用。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象的实例。虚引用的唯一作用是在对象被收集器回收时可以收到一个系统通知。在jdk1.2之后通过PhantomReference类来实现虚引用

## 对象的存活
当可达性算法判定对象是不可达时，对象不会被马上回收。jvm会对该对象进行第一次标记，随后会进行一次筛选，再次判断该对象是否要被回收。筛选的标准是对象是否有必要执行finalize()方法。如果该对象没有覆盖finalize()或者finalize()已经被调用过了，那么对象会被回收。
而如果对象被判定需要执行finalize()方法，则会把该对象放到一个优先级比较低的F-Queue队列中，然后虚拟机会创建一个低调度优先级的Finalizer线程去执行finalize()方法。**这里的执行只是保证开始执行，但不保证执行完成，是为了防止某些对象finalize方法执行缓慢甚至死循环，最终引起整个内存回收子系统的崩溃**
```java{.line-numbers}
package com.xcq.gc;

public class FinallyEscapeGC {

    public static FinallyEscapeGC SAVE_HOOK = null;

    public void isAlive() {
        System.out.println("still alive");
    }

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("finalize method executed");
        FinallyEscapeGC.SAVE_HOOK = this;
    }

    public static void main(String[] args) throws Throwable {
        SAVE_HOOK = new FinallyEscapeGC();

        //对象第一次拯救自己
        SAVE_HOOK = null;
        System.gc();
        Thread.sleep(500);
        if (SAVE_HOOK != null) {
            SAVE_HOOK.isAlive();
        } else {
            System.out.println("i am dead");
        }

        SAVE_HOOK = null;
        System.gc();
        Thread.sleep(500);
        if (SAVE_HOOK != null) {
            SAVE_HOOK.isAlive();
        } else {
            System.out.println("i am dead");
        }
    }
}
```
运行结果：
```
finalize method executed
still alive
i am dead
```
# 回收方法区
方法区的垃圾收集主要是两方面：废弃的字面值常量以及不再使用的类型。判断一个常量是否被废弃，只要看它还有没有引用就行了。但对于不再使用的类型需要满足下面的3个条件
- 该类所有实例均被回收
- 加载该类的类加载器也被回收
- 该类对应的java.lang.Class对象没有在任何地方被引用

# 垃圾收集算法
垃圾收集算法可以划分为"引用计数式垃圾收集"(Reference Counting GC)和"**追踪式垃圾收集**"(Tracing GC)。主流用的是追踪式垃圾收集
## 垃圾收集的三个原则
1. 绝大多数对象都是朝生夕死的
2. 熬过越多次垃圾收集过程的对象就越难以消亡
3. 跨代引用相比于同代引用来说仅占少数

基于原则1，2，垃圾回收会分区域进行，也就是所谓的"Minor GC""Major GC","Full GC"
基于原则3，在minor gc时没有必要去扫描整个老年代，找出那些跨代引用。而是在新生代开辟很小的一块区域去标记存在跨代引用的老年代内存块。发生minor gc时，只有这些被标记的内存块里的对象才会再次进行可达性分析
## 标记清除算法
首先标记出所有需要回收的对象，在标记完成后，统一回收掉所有被标记的对象。
缺点：
1. 执行效率不稳定，每次需要标记的对象数量是不确定的。当有大量对象需要被回收时，算法效率很低
2. 会产生大量的内存碎片

## 标记复制算法(最常使用)
按内存容量大小划分为大小相等的两块。每次只使用其中的一块，当某一块使用完之后，将其中仍然存活的对象复制到另一块，然后将这一块整个回收
### Appel式回收
由Andrew Appel提出。它将新生代分为3块区域，一块是Eden区，另外两块是survivor区域，这3块区域的大小比例是8:1:1。每次分配内存只使用Eden和一个survivor,在垃圾清理时，也只清理这两块区域。所以这种回收方式的空间利用率为10%
## 标记整理
基本过程和标记清除算法一致，但后续步骤不是对可回收对象进行整理，而是将所有存活的对象都向内存空间一端移动。
因为要移动存活对象，对于老年代这样的有大量存活对象的区域该算法是不太合适的。
# 垃圾收集的细节
## 根节点枚举细节
在做根节点枚举时，因为要暂时保证引用关系在一段时间内是不可变的，所以会出现stop the world现象。
因为主流的虚拟机都是准确式垃圾收集(知道内存中存储的是地址还是对象的值)，所以可以直接用一段空间(oopMap)去存储对象的引用位置。
### oopMap的存储
oopMap如果过多，显然存储的代价过大，但如果过少的话，那么每一次垃圾收集的间隔时间就会过长。因此需要选择合适数量和位置的oopMap，这些对应的位置就称为安全点(safe point)
### oopMap的更新
在每一次赋值操作完成后，都要更新一次记忆集，而这种更新是通过基于aop的写后屏障来实现的
```c
void oop_field_store(oop* field,oop new_value){
    //引用字段赋值
    *field = new_value;
    //写后屏障，在这里完成卡表状态的更新
    post_write_barrier(field,new_value);
}
```
## 三色算法
定义三种颜色表示垃圾收集器标记对象时，对象的三种状态
- 黑色
  表示该对象已经被垃圾收集器标记过，且它的所有直接引用也被垃圾收集器访问过
- 灰色
  表示该对象已经被垃圾收集器标记过，但它的直接引用对象没有全部被垃圾收集器访问过
- 白色
  表示该对象还没有被垃圾收集器访问过

所以可以认为灰色是白色和黑色中间态，标记完成后，只会有黑色以及白色对象，白色对象就是最后要被回收的对象。

## 并发中出现的对象消失问题
在并发中，如果某个对象在进行完可达性分析后，这个时候有别的线程将另一个对象引用赋值给这个对象，但因为该对象不会再重复分析可达性，因此新连上的对象最后会被判定为不可达，而被标记为可回收对象。

可以分为两种情况:
1. 在并发过程中，用户线程将原来应该被标记为白色的可回收对象和已经被标记为黑色的对象进行了连接，此时该对象原则上应该被标记为黑色，但因为黑色节点不会再被扫描，因此扫描结束后，该对象仍然是白色，会被回收(增加引用)
2. 在并发过程中，用户线程将灰色对象上的连接删除，本来该对象在这次的垃圾回收中，应该不被回收的，但却因为连接删除，最终被标记为白色而被回收。(删除引用)


为了解决这个问题，提出了两种方案
- 增量更新(Incremental Update)
- 原始快照(Snapshot At The Beginning,SATB)

增量更新是当加入新的引用连接到已分析过可达性的对象时，将这个引用记录下来。最后当这次并发扫描结束后，再以这些已分析过的对象为根，再进行一次可达性分析(用于解决情况1)

原始快照是指当要删除一个指向未分析完可达性的对象的引用时，将该引用记录下来，当并发扫描结束后，再以这些对象为根，再次进行可达性分析(解决情况2)
# 经典的垃圾处理器
![](https://gitee.com/zacharytse/image/raw/master/img/20201017161840.png)
## Serial收集器
Serial收集器是最早的单线程垃圾收集器。它工作在新生代。

![](https://gitee.com/zacharytse/image/raw/master/img/未命名文件.jpg)
## ParNew收集器
是serial的多线程版本

![](https://gitee.com/zacharytse/image/raw/master/img/dasdas.jpg)

## Parallel Scavenge收集器
同样基于标记-复制算法，和前面两个收集器最大的不同是它关注的是吞吐量，吞吐量的计算公式如下:
$$
吞吐量=\frac{运行用户代码时间}{运行用户代码时间+运行垃圾收集时间}
$$

# Serial Old收集器
是serial收集器的老年版本，采用标记整理算法

![](https://gitee.com/zacharytse/image/raw/master/img/未命名文件.jpg)

# Parallel Old收集器

是Parallel scavange收集器的老年代版本，基于标记-整理算法

![](https://gitee.com/zacharytse/image/raw/master/img/fgfg.jpg)

# CMS收集器
Concurrent Mark Sweep，基于标记-清除算法
整个过程分为四步
1. 初始标记(CMS initial mark)
2. 并发标记(CMS concurrent mark)
3. 重新标记(CMS remark)
4. 并发清除(CMS concurrent sweep)

初始标记是标记和GC roots直接相连的对象。
并发标记是从GC roots**直接关联对象开始**遍历整个对象图的过程
重新标记是基于增量更新对并发期间重新增加的引用对象进行标记
并发清除就是清除被标记的已经死亡的对象
> 初始标记和重新标记是需要stop the world

## 缺陷
- CMS因为是一款并发的垃圾收集器，所以对系统资源比较敏感
- CMS在垃圾收集的过程中会产生浮动垃圾。这些浮动垃圾只有等到下次垃圾回收时才会被回收。
- CMS在运行过程中是允许用户线程继续运行的，所以老年代要留出一部分空间供用户线程新产生的对象使用
- CMS因为是基于标记-清除算法实现的，所以会产生空间碎片

# Garbage First收集器
简称G1收集器。是一款面向服务端应用的垃圾收集器。
G1是基于Region来实现。将内存区域划分为大小相等的独立区域(Region)，每一个Region可以根据需要，扮演不同的角色(Eden,survior,老年代)。收集器会对扮演不同角色的Region采用不同的垃圾收集策略。
每次收集的内存空间都是Region的整数倍。
具体思路是让G1收集器去跟踪各个Region垃圾回收的"价值"，这里的价值是指在region中回收后可获得的空间大小以及所需的回收时间。并对这个价值进行排序，价值高的那些Region会被优先回收。

主要的四个步骤:
1. 初始标记(Initial Marking)
2. 并发标记(Concurrent Marking)
3. 最终标记(Final Marking)
4. 筛选回收(Live Data Counting and Evacuation)

初始标记和CMS一致，都是仅标记和GC ROOTS直接相连的那些对象

并发标记阶段从被标记为黑色的那些节点出发遍历整个图，同时要用SATB记录并发时有引用删除的对象(和用户线程并发执行)

并发标记阶段为每一个Region分配了两个TAMS(Top at Mark Start)的指针，这两个指针划分出一块区域专供并发时用户线程的新对象分配

最终标记阶段用于处理SATB中记录的那些对象

筛选回收是根据每一个Region的价值对那些被标记为白色的对象进行回收

低延迟的收集器可以等后面有空看

