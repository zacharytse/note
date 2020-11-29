[TOC]
# 线程池
![](https://gitee.com/zacharytse/image/raw/master/img/503d21502a149d5b9ca98a7ee9dde5d.jpg)
## ThreadPoolExecutor
线程池内部维护的工作者线程的数量就是该线程池的大小。它有3种形态
- 当前线程池大小
表示线程池中实际工作者线程的数量
- 最大线程池大小(maxinumPoolSize)
表示线程池中允许存在的工作者线程的数量上限
- 核心线程池大小(corePoolSize)
表示一个不大于最大线程池大小的工作者线程数量上限
## 优势
- 可以重复利用已经创建的线程
- 使得请求可以快速响应，避免了创建线程的时间
- 线程的创建需要占用系统内存，消耗系统资源，使用线程池可以更好的管理线程，做到统一分配，调优和监控线程，提高系统的稳定性

## 排队策略
- 如果运行的线程小于corePoolSize,则Executor始终选择添加新的线程，而不进行排队
- 如果运行的线程等于或多于corePoolSize,则Executor始终首选将请求加入队列，而不是添加新线程
- 如果无法将请求加入队列，即队列已经满了，则创建新的线程，除非创建此线程超出maxinumPoolSize,这时会直接拒绝该请求。

## 常见的线程池类型
- newCachedThreadPool()
    - 核心线程池大小为0，最大线程池不受限，即来一个创建一个线程
    - 适合执行大量耗时较短且提交频率较高的任务
    - 内部使用SynchronousQueue实现
- newFixedThreadPool()
    - 固定大小的线程池
    - 当线程池大小达到核心线程池大小，就不会增加也不会减少工作者线程的固定大小的线程池(max = core)

- newSingleThreadExecutor()
    - 便于实现单(多)生产者-消费者模式

## 阻塞队列
### ArrayBlockingQueue
- 使用一个数组作为其存储空间，数组的存储空间是预先分配的。
- 优点是put和take操作不会增加GC的负担
- 缺点是put和take操作使用同一个锁，可能导致锁争用
- 适合在并发程度较低的情况下使用

### LinkedBlockingQueue
- 无界队列(实际长度是Integer.MAX_VALUE)
- 内部存储空间是一个链表，并且链表节点所需的存储空间是动态分配的
- 优点是put和take操作使用两个显式锁
- 缺点是增加了GC的负担，因为空间是动态分配的
- LinkedBlockingQueue适合在生产者线程和消费者线程之间的并发程度较高的情况下使用

### SynchronousQueue
生产者生产一个产品后，会等待消费者线程来取走这个产品，然后才会接着生产下一个产品。

# CountDownLatch和CyclicBarrier
## CountDownLatch
```java{.line-numbers}
package com.xcq.thread.countdown;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

public class CountDownLatchExample {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(5);
        Service service = new Service(latch);
        Runnable task = () -> service.exec();
        for (int i = 0; i < 5; ++i) {
            Thread thread = new Thread(task);
            thread.start();
        }
        System.out.println("main thread await...");
        latch.await();
        System.out.println("main thread finish await...");
    }
}

class Service {

    private CountDownLatch latch;

    public Service(CountDownLatch latch) {
        this.latch = latch;
    }

    public void exec() {
        try {
            System.out.println(Thread.currentThread().getName()
                    + " execute task");
            sleep(2);
            System.out.println(Thread.currentThread().getName()
                    + " finish task");
        } finally {
            latch.countDown();
        }
    }

    private void sleep(int seconds) {
        try {
            TimeUnit.SECONDS.sleep(seconds);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
### 实现原理
- CountDownLatch内部维护一个计数器，countDown()会使得计数器的值减一
- 当计数器不为0时，await()方法的调用会导致执行线程被暂停，这些线程叫做CountDownLatch上的等待线程
- countDown()是一个通知方法，当计数器值达到0时，唤醒所有等待线程
## CyclicBarrier
```java{.line-numbers}
package com.xcq.thread.barrier;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.TimeUnit;

public class CyclicBarrierExample {

    public static void main(String[] args) {
        CyclicBarrier barrier = new CyclicBarrier(5, new Runnable() {

            @Override
            public void run() {
                System.out.println("start work...");
            }

        });
        for(int i = 0; i < 5; ++i) {
            new MyThread(barrier,Integer.toString(i)).start();
        }
    }
}

class MyThread extends Thread {

    private CyclicBarrier barrier;

    public MyThread(CyclicBarrier barrier,String name) {
        super(name);
        this.barrier = barrier;
    }

    @Override
    public void run() {
        try {
            System.out.println(Thread.currentThread().getName()
                    + " start preparing...");
            sleep(3);
            barrier.await();
            System.out.println(Thread.currentThread().getName()
                    + " finish preparing...");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (BrokenBarrierException e) {
            e.printStackTrace();
        }
    }

    private void sleep(int seconds) {
        try {
            TimeUnit.SECONDS.sleep(seconds);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
在实际使用时，可以用它来模拟**高并发**的场景
### 实现原理
- 通过CyclicBarrier实现等待的线程被称为参与方(Party),参与方只需要执行await()方法就可以实现等待。
- 内部维护了一个显式锁，当最后一个参与方调用await()方法时，前面等待的参与方都会被唤醒，并且最后一个参与方也不会被暂停
- 内部维护了一个计数器变量，count=参与方的个数。调用await()方法可以使得count-1.当判断最后一个参与方时，调用signalAll唤醒所有线程
# ThreadLocal
使用ThreadLocal维护变量时，为每个使用该变量的线程提供**独立**的变量副本
## 实现机制
- 每一个线程内部都会维护一个类似HashMap的对象，称为ThreadLocalMap，里面包含了若干Entry，相应的线程被称为这些Entry的属主线程
- Entry的key是一个ThreadLocal实例，value是一个线程特有的对象
- Entry对key的引用是弱引用，对value的引用是强引用
## 内存泄漏的问题
因为ThreadLocalMap会持有线程特有对象的引用，所以它的生命周期和线程一样长。所以要手动删除key,否则会导致弱引用
# Atomic
AtomicInteger类提供getAndIncrement和incrementAndGet等原子性的自增自减操作。内部使用CAS来保证原子性