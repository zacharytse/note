[TOC]
# 线程安全的三大特性
## 原子性
对于涉及到共享变量访问的操作，若该操作从执行线程以外的线程来看是不可分割的，则该操作具有原子性。
### 实现方法
- 利用锁的排它性，保证同一时刻只有一个线程在操作一个共享变量
- 利用CAS保证
- java语言规范中，保证了除long和double型以外任何变量的写操作都是原子的
- volatile关键字修饰的变量可以保证其写操作的原子性
## 可见性
是指一个线程更新完一个共享变量后，该次更新其他线程在后续操作中是否可见
### 实现方法
- 当前处理器需要刷新处理器缓存(内存->处理器)，使得其余处理器对变量所作的更新可以同步到当前的处理器缓存中
- 当前处理器对共享变量更新之后，需要flush处理器缓存，使得该更新可以被写入处理器缓存中(处理器->内存)

## 有序性
一个处理器上运行的线程所执行的内存访问操作在另一个处理器上运行的线程来看是否有序

# synchronized
是java中的关键字，是一个内部锁。
## 底层实现
- 进入时，执行monitorenter,将计数器+1,释放锁monitorexit时，计数器-1
- 当一个线程判断到计数器为0时，则当前锁空闲，可以占用。反之，当前线程进入等待状态

### 对原子性的保证
通过互斥来保障原子性，一次只有一个线程可以进入到临界区
### 对可见性的保证
通过写线程冲刷处理器缓存和读线程刷新处理器缓存
- 获得锁之后，刷新处理器缓存，将之前线程的更新读入到当前处理器的缓存中
- 释放锁之后，需要冲刷处理器缓存，将当前线程对共享变量做的更新推送到下一个线程的高速缓存中。

### 对有序性的保证
原子性和可见性的保证使得其在临界区的操作都是完全按照代码顺序执行的

> jvm对内部锁的调度是一种非公平的调度方式。JVM会给每一个内部锁分配一个入口集(Entry Set),用于记录等待获得相应内部锁的线程。被唤醒的线程还有其他新的活跃线程会和该线程抢占这个锁。

# volatile关键字
是一个轻量级的锁，可以保证可见性和有序性，但不保证原子性。
- volatile可以保证主内存和工作内存直接产生交互，进行读写操作，保证可见性
- 它只保证写操作的原子性，但对于复杂的读写操作无法保证其原子性
- 可以禁止指令重排序

**volatile开销较大，因为它需要从高速缓存或者内存中读取，而不能从寄存器中读取**

> volatile比较适合多个线程共享一个状态变量


# ReentrantLock
```java{.line-numbers}
public class Counter {
    private final Lock lock = new ReentrantLock();
    private int count;

    public void add(int n) {
        lock.lock();
        try {
            count += n;
        } finally {
            lock.unlock();
        }
    }
}
```
是一种显式锁，支持公平和非公平调度，默认采用非公平调度。它提供了一个tryLock()方法，尝试去获取锁，成功返回true，若失败则直接返回false,不会阻塞当前线程，从而避免了死锁。