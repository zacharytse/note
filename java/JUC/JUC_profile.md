[TOC]
#什么是JUC
java.util.concurrent
#线程和进程
一个进程可以包含多个线程，至少包含一个
java默认有2个线程，一个是main线程，一个是gc线程
**java只能通过native方法去创建线程**
##并发和并行
并发：多线程操作同一个资源(单核下只有并发)
并行：同时进行
## 线程状态
- NEW
  新生
- RUNNABLE
  运行
- BLOCKED
  阻塞
- WAITING
  等待，一直等
- TIME_WAITING
  超时等待
- TERMINATED
  终止
## wait和sleep区别
- 来自不同的类,wait来自Object，sleep来自Thread
- 关于锁的释放
  wait会释放锁，sleep不会释放锁
- 使用的范围是不同的
  sleep可以在任何地方睡
  wait必须在同步代码块中
##Lock锁(重点)
> 传统Synchronized
```java{.line-numbers}
/**
 * 线程就是一个单独的资源类
 * 没有任何附属的操作
 * 资源包括属性和方法
 */
public class demo01 {
    public static void main(String[] args) {
        //多线程操作同一个资源类，把资源类丢入线程中
        Ticket ticket = new Ticket();
        new Thread(()->{
            for(int i = 0; i < 60; ++i){
                ticket.sale();
            }
        },"A").start();
        new Thread(()->{
            for(int i = 0; i < 60; ++i){
                ticket.sale();
            }
        },"B").start();
        new Thread(()->{
            for(int i = 0; i < 60; ++i){
                ticket.sale();
            }
        },"C").start();
    }
}


/**
 * 单纯的资源类，不要去实现接口，否则会提高耦合度
 */
class Ticket{
    private int number = 50;

    //卖票的方式
    //synchronized本质是队列，锁
    public synchronized void sale(){
        if(number > 0){
            System.out.println(Thread.currentThread().getName() + "卖出了第" + (number--) + "票，剩余" + number);
        }
    }
}
```
> lock
- 公平锁
  十分公平，可以先来后到
- 非公平锁
  十分不公平，可以插队(默认)
```java{.line-numbers}
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class demo02 {
    public static void main(String[] args) {
        //多线程操作同一个资源类，把资源类丢入线程中
        Ticket ticket = new Ticket();
        new Thread(()->{
            for(int i = 0; i < 60; ++i){
                ticket.sale();
            }
        },"A").start();
        new Thread(()->{
            for(int i = 0; i < 60; ++i){
                ticket.sale();
            }
        },"B").start();
        new Thread(()->{
            for(int i = 0; i < 60; ++i){
                ticket.sale();
            }
        },"C").start();
    }
}

class Ticket{
    private int number = 50;

    Lock lock = new ReentrantLock();
    //卖票的方式
    //synchronized本质是队列，锁
    public void sale(){
        lock.lock();//加锁
        try {
            if(number > 0){
                System.out.println(Thread.currentThread().getName() + "卖出了第" + (number--) + "票，剩余" + number);
            }
        }catch (Exception e){
            
        }finally {
            lock.unlock();
        }

    }
}
```
> lock和synchronized的区别
- synchronized是java内置的关键字，lock是一个java类
- synchronized无法判断锁的状态，lock可以判断是否获取到了锁
- synchronized会自动释放锁，lock必须要手动释放锁，如果不释放，会造成死锁
- synchronized 线程1(获得锁，阻塞),线程2(会一直等)，而lock锁不会一直等
- synchronized是可重入锁，不可以中断，非公平；lock，可重入，可以判断锁，可以自己设置是否公平
- Synchronized适合锁少量的代码同步问题，lock适合锁大量的同步代码·1