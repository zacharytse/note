[TOC]
# CAS是什么
CAS是CompareAndSwap的缩写。
CAS指令执行时，当且仅当内存地址V的值与预期值A相等时，将内存地址V的值修改为B，否则就什么都不做。
## intel lock前缀
用于保证指令的原子性，确保处理器可以独占使用某些共享内存。
### 实现方式
**方法一**
直接锁定总线，让某个核心独占使用总线。
这种方法代价太大，总线被锁定后，其他核心就不能访问内存，可能会导致其他核心短时间内停止工作
**方法二**
锁定缓存。即如果要锁定的内存区域在某个核心的缓存上，则只锁定该缓存所对应的内存区域，其他处理器在锁定期间无法访问这一块内存区域

**CAS的实现依赖于lock前缀**

# 源码分析
CAS在java中的实现可以参考AtomicInteger类中的compareAndSet方法
```java{.line-numbers}
public class AtomicInteger extends Number implements java.io.Serializable {

    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            // 计算变量 value 在类对象中的偏移
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;
    
    public final boolean compareAndSet(int expect, int update) {
        /*
         * compareAndSet 实际上只是一个壳子，主要的逻辑封装在 Unsafe 的 
         * compareAndSwapInt 方法中
         */
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }
    
    // ......
}
```
compareAndSet方法的实现是由unsafe中的compareAndSwapInt实现
```java{.line-numbers}
public final class Unsafe {
    // compareAndSwapInt 是 native 类型的方法，继续往下看
    public final native boolean compareAndSwapInt(Object o, long offset,
                                                  int expected,
                                                  int x);
    // ......
}
```
compareAndSwapInt的jni实现如下
```c++{.line-numbers}
// unsafe.cpp
/*
 * 这个看起来好像不像一个函数，不过不用担心，不是重点。UNSAFE_ENTRY 和 UNSAFE_END 都是宏，
 * 在预编译期间会被替换成真正的代码。下面的 jboolean、jlong 和 jint 等是一些类型定义（typedef）：
 * 
 * jni.h
 *     typedef unsigned char   jboolean;
 *     typedef unsigned short  jchar;
 *     typedef short           jshort;
 *     typedef float           jfloat;
 *     typedef double          jdouble;
 * 
 * jni_md.h
 *     typedef int jint;
 *     #ifdef _LP64 // 64-bit
 *     typedef long jlong;
 *     #else
 *     typedef long long jlong;
 *     #endif
 *     typedef signed char jbyte;
 */
UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSwapInt(JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jint e, jint x))
  UnsafeWrapper("Unsafe_CompareAndSwapInt");
  oop p = JNIHandles::resolve(obj);
  // 根据偏移量，计算 value 的地址。这里的 offset 就是 AtomaicInteger 中的 valueOffset
  jint* addr = (jint *) index_oop_from_field_offset_long(p, offset);
  // 调用 Atomic 中的函数 cmpxchg，该函数声明于 Atomic.hpp 中
  return (jint)(Atomic::cmpxchg(x, addr, e)) == e;
UNSAFE_END

// atomic.cpp
unsigned Atomic::cmpxchg(unsigned int exchange_value,
                         volatile unsigned int* dest, unsigned int compare_value) {
  assert(sizeof(unsigned int) == sizeof(jint), "more work to do");
  /*
   * 根据操作系统类型调用不同平台下的重载函数，这个在预编译期间编译器会决定调用哪个平台下的重载
   * 函数。相关的预编译逻辑如下：
   * 
   * atomic.inline.hpp：
   *    #include "runtime/atomic.hpp"
   *    
   *    // Linux
   *    #ifdef TARGET_OS_ARCH_linux_x86
   *    # include "atomic_linux_x86.inline.hpp"
   *    #endif
   *   
   *    // 省略部分代码
   *    
   *    // Windows
   *    #ifdef TARGET_OS_ARCH_windows_x86
   *    # include "atomic_windows_x86.inline.hpp"
   *    #endif
   *    
   *    // BSD
   *    #ifdef TARGET_OS_ARCH_bsd_x86
   *    # include "atomic_bsd_x86.inline.hpp"
   *    #endif
   * 
   * 接下来分析 atomic_windows_x86.inline.hpp 中的 cmpxchg 函数实现
   */
  return (unsigned int)Atomic::cmpxchg((jint)exchange_value, (volatile jint*)dest,
                                       (jint)compare_value);
}
```
先获取到AtomicInteger加了volatile的value变量的地址addr，然后将x,addr,e传入cmpxchg函数(x是需要改变的值，e是用于比较的值)。
cmpxchg函数的作用是将dest所指向的内存单元的内容与compare_value进行比较，若相同，则将exchange_value的值写入到dest所指向的内存单元中。

然后再来看cmpxchg的实现
```c++{.line-numbers}
inline jint Atomic::cmpxchg (jint exchange_value, volatile jint* dest, jint compare_value) {
  // 判断是否是多核 CPU
  int mp = os::is_MP();
  __asm {
    // 将参数值放入寄存器中
    mov edx, dest    // 注意: dest 是指针类型，这里是把内存地址存入 edx 寄存器中
    mov ecx, exchange_value
    mov eax, compare_value
    
    // LOCK_IF_MP
    cmp mp, 0
    /*
     * 如果 mp = 0，表明是线程运行在单核 CPU 环境下。此时 je 会跳转到 L0 标记处，
     * 也就是越过 _emit 0xF0 指令，直接执行 cmpxchg 指令。也就是不在下面的 cmpxchg 指令
     * 前加 lock 前缀。
     */
    je L0
    /*
     * 0xF0 是 lock 前缀的机器码，这里没有使用 lock，而是直接使用了机器码的形式。至于这样做的
     * 原因可以参考知乎的一个回答：
     *     https://www.zhihu.com/question/50878124/answer/123099923
     */ 
    _emit 0xF0
L0:
    /*
     * 比较并交换。简单解释一下下面这条指令，熟悉汇编的朋友可以略过下面的解释:
     *   cmpxchg: 即“比较并交换”指令
     *   dword: 全称是 double word，在 x86/x64 体系中，一个 
     *          word = 2 byte，dword = 4 byte = 32 bit
     *   ptr: 全称是 pointer，与前面的 dword 连起来使用，表明访问的内存单元是一个双字单元
     *   [edx]: [...] 表示一个内存单元，edx 是寄存器，dest 指针值存放在 edx 中。
     *          那么 [edx] 表示内存地址为 dest 的内存单元
     *          
     * 这一条指令的意思就是，将 eax 寄存器中的值（compare_value）与 [edx] 双字内存单元中的值
     * 进行对比，如果相同，则将 ecx 寄存器中的值（exchange_value）存入 [edx] 内存单元中。
     */
    cmpxchg dword ptr [edx], ecx
  }
}
```
再看最后的CAS的自旋
```java{.line-numbers}
public class AtomicTest {
 
    public static void main(String[] args) {
        MyAtomicInteger in = new MyAtomicInteger(1);
        System.out.println(in.getAndIncr());
    }
 
    /**
     * 使用继承AtomicInteger的方式模拟CAS的过程
     */
    static class MyAtomicInteger extends AtomicInteger {
 
        public MyAtomicInteger(int i) {
            super(i);
        }
 
        //先获取值再自增1
        public final int getAndIncr() {
 
            for (; ; ) {
 
                //内存位置拿到的值
                int current = get();
 
                //新的值
                int next = current + 1;
 
                /**
                 * 比较当前内存的值是否跟之前拿到的预期值current一致，如果一致，则将新的值next设值到内存中
                 * 并返回true，反之返回false。（可能有人问你拿current和current比较当然一样，但是在高并发场景
                 * 就算你在拿到current后没有执行int next = current + 1;这句话都有可能刚执行get()拿到current后
                 * CPU被切换到其它线程执行修改操作，何况是get()后还有一句话，也就是说当你拿到current后，在你
                 * compareAndSet(current, next)的时候可能当前内存位置的值已经被其它线程修改成了别的了）回到正题
                 * 当compareAndSet返回true就表示这个自增操作成功，直接返回原先那个值，如果返回false，则重新循环
                 * 这段操作过程，直到成功为止，这也叫CAS自旋。
                 */
 
                if (compareAndSet(current, next))
 
                    return current;
 
            }
        }
 
    }
}
```
总结下CAS的工作原理，也就是首先先从内存或者当前核心的缓存中取出一个值，这个值是作为expect value。然后再根据expect value得到一个update value。接着执行CAS操作，首先对expect value所在的内存区域加锁，保证该核心独占这一块内存区域。接着该核心将expect value与内存区域中的数据进行比较，如果不相同，则说明在lock前已经有其他核心更改了该数据，此次的update value必然是有问题的，此时返回false，接着下一次的循环更新操作。如果expect value和内存区域中的数据是一致的话，说明update value是正确的，此时将update value的值写入到该内存区域中。如此则完成一次CAS操作
# CAS存在的问题
## ABA问题
1. 线程1执行读取操作，获取到了内存区域的值，即expect value,但此时线程被切换走
2. 线程2执行完成了CAS操作，将原值由A修改为了B
3. 此时线程2又再次执行CAS操作，将B又改回了A
4. 现在切换回了线程1操作，线程1发现内存中的值是A，和expect value是相同的，所以顺利修改为了B。

按照这样的流程来看，线程1是因为不知道A已经被成了B再又被改回了A，这一过程对线程1来说是透明的。
要解决这样的问题，对每一次的CAS操作都要设置一个版号。这样线程1再切回来继续执行CAS操作时，发现虽然expect value没有变，但是版号改变了，即中间一定发生了CAS