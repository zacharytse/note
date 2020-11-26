[TOC]
# 线程的六种状态
## NEW
已经创建出来，但还没有调用start方法
## RUNNABLE
包含两种状态，一种是正在运行，另一种是正在等待CPU资源
## BLOCKED
阻塞状态，当线程准备进入synchronized同步块或同步方法时，需要申请一个监视器锁而进行的等待，会使线程进入BLOCKED状态
## WAITING
调用了Object.wait()或者Thread.join()或者LockSupport.park()。处于该状态下的线程在等待另一个线程执行一些其余action来将其唤醒
## TIME_WAITING
和WAITING状态一样，但是等待时间是明确的
## TERMINATED
消亡状态，线程执行结束
# synchronized
## 修饰的对象
1. 修饰一个代码块，这个代码块被称为同步语句块
2. 修饰一个方法，这个方法被称为同步方法
3. 修饰一个静态方法，作用的对象是这个类的所有对象
4. 修饰一个类，作用的对象是这个类的所有对象
### 修饰一个方法
- 写法一
```java{.line-numbers}
public class ThreadSyn1 implements Runnable {

    private volatile static int count = 0;

    public static void main(String[] args) {
        ThreadSyn1 threadOne = new ThreadSyn1();
        new Thread(threadOne,"1").start();
        new Thread(threadOne,"2").start();
    }
    @Override
    public void run() {
        synchronized (this) {
            for (int i = 0; i < 5; ++i) {
                try {
                    System.out.println(Thread.currentThread().getName()
                            + ":" + (count++));
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```
- 写法二
```java{.line-numbers}
public synchronized void method() {
    //todo
}
```
**这里要注意，synchronized方法是不能被继承的，父类中使用了该关键字的方法，若在子类中重新实现，子类中的该方法不是同步的**
### 修饰一个静态方法
```java{.line-numbers}
public synchronized static void method() {
    //todo
}
```
### 修饰一个类
```java{.line-numbers}
public class ClassName {
    public void method() {
        synchronized(ClassName.class) {
            //todo
        }
    }
}
```


# 一些常用的函数
## sleep
Thread的静态方法，当前线程阻塞n毫秒，睡眠时间到了之后，线程会进入RUNNABLE状态，等待cpu的到来，sleep时不会释放锁
## wait
必须与synchronized关键字一起使用，线程会进入阻塞状态，当notif或者notifyall被调用后，会解除阻塞。只有重新占用互斥锁之后，才会进入RUNNABLE状态。睡眠时会释放互斥锁。
```java{.line-numbers}
package com.xcq.thread;

public class WaitNotify {

    public static void main(String[] args) {
        final byte[] a = {0};
        new Thread(new NumberPrint(1,a),"1").start();
        new Thread(new NumberPrint(2,a),"2").start();
    }
}

class NumberPrint implements Runnable {

    private int number;
    private byte[] res;
    public static int count = 5;

    public NumberPrint(int number, byte[] res) {
        this.number = number;
        this.res = res;
    }

    @Override
    public void run() {
        synchronized (res) {
            while (count-- > 0) {
                try {
                    res.notify();
                    System.out.println(" " + number);
                    res.wait();
                    System.out.println("------thread" + Thread.currentThread().getName()
                            + "get the lock,after wait():" + number);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

## join
当前线程调用，则其他线程全部停止，等待当前线程执行完毕，其他线程再接着执行
## yield
该方法使得线程放弃当前分得的CPU时间，但是不使线程阻塞，即线程仍然处于可执行状态，随时可能再次分得CPU时间
```java{.line-numbers}
package com.xcq.thread.yield;

public class YieldTest extends Thread {

    public YieldTest(String name) {
        super(name);
    }

    @Override
    public void run() {
        for (int i = 1; i <= 50; ++i) {
            System.out.println(this.getName() + "-------------" + i);
            if (i == 30) {
                this.yield();
            }
        }
    }

    public static void main(String[] args) {
        YieldTest yt1 = new YieldTest("zhangsan");
        YieldTest yt2 = new YieldTest("lisi");
        yt1.start();
        yt2.start();
    }
}
```
# 死锁
## 产生死锁的4个条件
- 资源互斥： 一个资源一次只能被一个线程使用
- 请求与保持条件：一个线程因请求资源而阻塞，且其持有的资源不会主动释放
- 不剥夺条件：线程已获得的资源，在其执行结束前，不会被剥夺
- 循环等待：若干线程之间形成一种头尾相接的循环等待资源关系

## 避免死锁产生的方法
1. 粗锁法

增加锁的粒度来消除请求与保持条件(让线程一次性申请完资源，这样不会在执行过程中发生阻塞)

2. 锁排序法

在执行时，指定获取锁的顺序，来消除循环等待条件
```java{.line-numbers}
package com.xcq.thread.lock.sort;

public class BankAccount {

    private static final Object tieLock = new Object();//排序相等时用来同步的锁对象
    private String id;
    private volatile double amount;

    public BankAccount(String id, double amount) {
        this.id = id;
        this.amount = amount;
    }

    public void transfer(BankAccount to, double amount) {
        int fromHash = System.identityHashCode(this);
        int toHash = System.identityHashCode(to);
        if (fromHash < toHash) {
            synchronized (this) {
                synchronized (to) {
                    this.amount -= amount;
                    to.amount += amount;
                }
            }
        } else if (fromHash > toHash) {
            synchronized (to) {
                synchronized (this) {
                    this.amount -= amount;
                    to.amount += amount;
                }
            }
        } else {
            synchronized (tieLock) {
                synchronized (this) {
                    synchronized (to) {
                        this.amount -= amount;
                        to.amount += amount;
                    }
                }
            }
        }
    }
}
```
# 线程锁死(和死锁做区分)
线程锁死是指唤醒该线程的条件永远无法成立或者其他线程无法唤醒这个线程而使其一直处于非运行状态(线程并未终止)导致其任务一直无法进展。
## 分类
1. 信号丢失锁死

没有对应的通知线程来将等待线程唤醒，导致该线程一直处于等待状态。
例如在使用Object.wait()/Condition.await()时，没有先对synchronized的保护变量进行判断，有可能该变量已经满足条件，不需要wait,但仍然进行了wait操作，此时已满足条件，所以不会再有别的线程执行notify()来唤醒该线程，造成了信号丢失锁死

2. 嵌套监视器锁死

嵌套锁导致等待线程永远无法被唤醒
例如一个线程，只执行了内层的y.wait()操作，但始终持有外层的X锁，而另一个线程需要先获取到x锁，才能执行内层的y.notifyall()操作
# 活锁
线程始终处于**RUNNABLE**状态，但却一直无法获取其所需的资源，导致其执行的任务没有进展。例如线程一直在申请内存资源，但却始终申请不到。
# 线程饥饿
线程一直无法获取到其所需资源而导致任务无法进行。
## 线程调度模式
1. 公平调度
2. 非公平调度

只有在非公平调度下，才会出现线程饥饿的情况。