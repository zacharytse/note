#HashMap是线程不安全的
##HashMap的存储结构
```java{.line-numbers}
transient Node<K,V>[] table;
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
}
```
HashMap内部存储使用了一个Node类型的数组table,默认大小时16，Node中包含了Node类型的next变量，所以是一个单链表结构
![](https://pic.yupoo.com/jiananshi/e578c267/c8d4d250.png)
#HashMap的自动扩容机制
HashMap内部的Node数组默认大小是16，所以如果有100万个元素，最好情况(均匀分配到每个Node上),每个hash桶上有62500个元素。此时的冲突率非常高，平均查找次数也是最多的，为了减少平均查找此时，HashMap提供了自动扩容机制。
当元素个数达到数组大小loadFactor(装填因子$\alpha$)后会扩大数组的大小。在默认情况下，数组大小为16，loadFactor为0.75。当Node的元素数量超过16*0.75=12时，会把数组大小扩展为2*16=32，并且重新计算每个元素在新数组中的位置。
![](https://pic.yupoo.com/jiananshi/ce749998/37356c77.jpg)
#为啥线程不安全
HashMap在扩容时，多线程会导致Node链表形成环形链表，那么在获取Node时就会造成死循环。
先看HashMap的扩容代码
```java{.line-numbers}
void resize(int newCapacity)
{
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    ......
    //创建一个新的Hash Table
    Entry[] newTable = new Entry[newCapacity];
    //将Old Hash Table上的数据迁移到New Hash Table上
    transfer(newTable);
    table = newTable;
    threshold = (int)(newCapacity * loadFactor);
}
```
迁移代码如下
```java{.line-numbers}
void transfer(Entry[] newTable)
{
    Entry[] src = table;
    int newCapacity = newTable.length;
    //下面这段代码的意思是：
    //  从OldTable里摘一个元素出来，然后放到NewTable中
    for (int j = 0; j < src.length; j++) {
        Entry<K,V> e = src[j];
        if (e != null) {
            src[j] = null;
            do {
                Entry<K,V> next = e.next;
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            } while (e != null);
        }
    }
} 
```
单线程下扩容，会先创建一个新的Node表，然后把原表迁移到新表中。每一个hash桶的元素重新hash，使用尾插(保证原来插入的相对顺序)到新的桶中。
![](https://coolshell.cn/wp-content/uploads/2013/05/HashMap01.jpg)
但在多线程下，这会出现问题
![](https://coolshell.cn/wp-content/uploads/2013/05/HashMap02.jpg)
迁移代码中的这句话
```java
Entry<K,V> next = e.next;
```
因为e本身是一个竞争变量，如果线程1在获取到e.next被挂起，而此时另一个线程完成了操作，线程1再接着执行，就会出现问题。
如上图所示，线程1挂起，线程2执行完毕。此时e指向了Key:3,而next指向了7。但是7的next仍然是3，即产生了循环链表
![](https://coolshell.cn/wp-content/uploads/2013/05/HashMap03.jpg)
1