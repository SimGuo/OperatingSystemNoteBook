#Semaphores

----

###信号量介绍

---

---
**信号量的基本内容**
- 信号量是一个提供对临界区的互斥访问的抽象的数据类型
- 每个信号量都有一个等待队列
- 信号量是支持P和V操作的整数
  - P(semaphore)：整数减一，如果信号量关着则阻塞
  - V(semaphore)：整数加一，允许另外一个线程进入
  如果有线程在等待队列中，则将其Unblock。如果没有线程在等待，则信号量会“记住”这个空位的存在。
- 信号量的安全性能：信号量的值永远大于等于0

---
**信号量的分类**
- 互斥信号量（二元信号量）
>保证对临界区的互斥访问，只提供对资源的单一的访问途径
- 计数信号量（一般信号量）
>用很多可用的单位资源表示一个资源，或者说一个允许多种非同步化的并发的访问。（也就是多个线程可以同时访问）（同时访问数的上限用count定义）

---
**信号量的语义**
- 一种假想：
```
void P(int *semaphore) {
    while (*semaphore <= 0);
    *semaphore--;
}
void V(int *semaphore) {
    *semaphore++;
}
```
上述实现的操作必须是原子，但忙等其实很耗资源，所以实际信号量的实现并不是这样。

实际信号量是如下的结构：
```
struct Semaphore {
    int value;
    Queue q;
};
```
也就是有一个等待队列。mutex信号量其实很lock很像，仅仅是语义不一样。

---

###使用信号量可能存在的问题

---

---
**问题一：死锁**

先看下面一段代码：

```S = 1; Q = 1;```

P_0：``` P(S); P(Q); V(S); V(Q);```

P_1：``` P(Q); P(S); V(Q); V(S);```

这种情况下，如果P0先抢到了S，P1先抢到了Q，则两个线程会陷入**死锁**。

---
**经典的同步问题——读者写者问题**
- 问题描述：
  - 一个对象被多个线程共享
  - 有一些线程只读，有一些线程只写
  - 我们允许同时有多个读者，但同时只能有一个写者
- 用信号量解决这个问题：
  - 尝试一：
    ```
    Semaphore w_or_r = 1;
    Reader {
        P(w_or_r);
        read;
        V(w_or_r);
    }
    Writer {
        P(w_or_r);
        write;
        V(w_or_r);
    }
    ```
    但这是不可行的，读者和写者的角色没有区分开。读者和读者之间也是互斥的，并不能满足多个读者同时读的要求。
  - 尝试二
    ```
    Semaphore w_or_r = 1;
    int readcount; //record readers
    Reader {
        readercount ++;
        if(readcount == 1) {
            P(w_or_r);   //lock out writers
        }
        read;
        readcount--;
        if(readcount == 0) {
            V(w_or_r); //up for grabs
        }
        Writer {
            P(w_or_r);    //lock out readers
            write
            V(w_or_r);  //up for grabs
        }
    }
    ```
    但这样又存在一个问题：readcount是所有的读者的共享变量。假设所有的读者都在`readcount++;`后被调度下CPU，则读者无法抢到锁。
  - 真实的做法（给readcount也加上锁）：
    使用三个变量`readcount` `mutex` `w_or_r`
    ```
    int readcount = 0; 
    Semaphore mutex = 1;
    Semaphore w_or_r = 1;
    Writer {
        P(w_or_r);
        write;
        V(w_or_r);
    }    
    Reader {
        P(mutex);
        readcount++;
        if(readcount == 1)
            P(w_or_r);
        V(mutex);
        Read;
        P(mutex);
        readcount--;
        if(readcount == 0)
            V(w_or_r);
        V(mutex);
    }
    ```
    这种做法就是让第一个读者作为所有读者的代表去抢读写锁，但一个写者只能自己抢锁。所以会造成写者的饥饿。假想一直有读者，则写者就一直得不到锁。


---
**由此我们引出了第二个问题：饥饿**

如何写出一个不会出现饥饿问题的解法呢？

---

**回顾信号量**
- 信号量可以被用来解决一些传统的同步问题
- 但信号量有一些缺陷：
  - 它们本质上是共享变量，可以在程序的任何地方使用
  - 在共享变量和信号量之间其实没有什么逻辑联系 
  - 既可以用作临界区（互斥机制）又可以用于协调（schedulling）
  - 没有办法控制和保证正确的使用
- 有的时候很难使用，而且易于出错。
