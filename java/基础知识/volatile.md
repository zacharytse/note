[TOC]
# 什么是Volatile
**java的原子性**
在java中，对基本数据类型的变量的读取和赋值操作是原子性的，即这些操作是不可被中断的，要么执行，要么不执行
```java{.line-numbers}
x = 10;         //语句1
y = x;         //语句2
x++;           //语句3
x = x + 1;     //语句4
```
对于上面4句话，只有语句1是具有原子性的。
解释下语句2，语句2实际上包含了两个操作，先要读取x的值，然后把x的值赋值给y，所以是两步操作，不具备原子性
**只有简单的读取、赋值（而且必须是将数字赋值给某个变量，变量之间的相互赋值不是原子操作）才是原子操作。**
**java的可见性**
java的内存模型:
![](https://pic4.zhimg.com/80/v2-632a9436e9758d3e4f012dd3affcb477_hd.jpg)
对于可见性，java就是通过volatile关键字来保证。
当一个共享变量被volatile修饰时，它会保证修改的值立刻被更新到主内存中。当有其他线程需要读取时，它会从内存中读取新值
**有序性**
java内存模型中，允许编译器和处理起堆指令进行重排序，这一过程不会影响到单线程程序的执行，但会影响到多线程并发执行的正确性

java内存模型天生具备一些有序性，即不需要通过任何手段就能够得到保证的有序性，这也被称为happens-before原则。
> happens-before原则
- 程序次序规则：**一个线程内**，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作
- 锁定规则：一个unLock操作先行发生于后面对同一个锁的lock操作
- volatile变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作
- 传递规则：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C(说明happens-before具有传递性)
- 线程启动规则：Thread对象的start()方法先行发生于此线程的每个一个动作
- 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
- 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行
- 对象终结规则：一个对象的初始化完成先行发生于他的finalize()方法的开始

## volatile关键字的两层语义
1. 保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。
2. 禁止进行指令重排序。
```java{.line-numbers}
//线程1
boolean stop = false;
while(!stop){
    doSomething();
}
 
//线程2
stop = true;
```
一般可以通过上面的语句去停止线程1。但可以考虑一种极端情况，如果线程2将stop设置为true，但只是将其保存在了工作内存，而没有将stop的值写回内存中，这时候线程1中的stop还是false,会继续执行。
但如果用volatile修饰后，线程2在修改完stop的值之后，会立刻将其从工作内存更新到内存中。同时会将线程1中的stop缓存行置为无效，线程1此时只能等待内存更新完成后，重新从内存中读取stop的值，此时线程1读到的stop一定是正确的值

```java{.line-numbers}
package com.xcq;

public class TestThread {
    private volatile int inc = 0;

    public void increase(){
        ++inc;
    }

    public static void main(String[] args) {
        final TestThread testThread = new TestThread();
        for(int i = 0;i<10;++i){
            new Thread(){
                @Override
                public void run() {
                    for(int j = 0; j < 1000;++j)
                        testThread.increase();
                }
            }.start();
        }

        while(Thread.activeCount() > 2){//主线程+gc线程不能考虑
            Thread.yield();
        }
        System.out.println(testThread.inc);
    }
}
```
再来看上面这段代码。可以发现每次运行得到的inc的结果都不一样，这说明了volatile并不能保证对变量操作的原子性

> volatile可以在一定程度上保证指令执行的有序性

```java{.line-numbers}
//x、y为非volatile变量
//flag为volatile变量
 
x = 2;        //语句1
y = 0;        //语句2
flag = true;  //语句3
x = 4;         //语句4
y = -1;       //语句5
```
volatile可以保证在语句3执行前，语句1，2已经执行完毕，同时也可以保证语句4，5没有执行。但语句1，2的执行顺序与语句4，5的执行顺序是没有办法保证的