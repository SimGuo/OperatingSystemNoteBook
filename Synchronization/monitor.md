#Monitor

---

---

`monitor` `Condition variable`

---
###管程

---
**管程的定义**

管程是对共享数据的访问进行控制的特殊的一段程序：
- 这是编译器的一段代码，在运行的时候执行

管程是一个压缩了如下内容的模块：
- 共享的数据结构
- 对共享数据的操作
- 在不同线程之间的同步操作

它能够：
- 保证数据不会有非同步的访问
- 只允许线程以合理的方式访问共享数据

---
**管程的语义**

- 管程保证互斥访问：
  - 一次只能有一个线程“在管程中”（也就是执行管程提供的程序）
  - 如果一个线程在管程中的时候，有第二个线程调用管程中的程序，则这第二个线程会被阻塞。
  - 如果在管程中的线程被阻塞了，则另外一个线程就可以进入管程

- 问题：
  - 管程和我们了解的临界区有什么区别呢？
  >我的理解：管程包括了对共享数据的处理函数。用户只能使用管程提供的方式，不能自定义；但在临界区中用户可以对共享数据做任何事情。临界区是指会对共享数据产生data race的代码段，无论是否采取保护措施临界区都是存在的。而管程可以被认为是对临界区进行保护的一段代码，包装了临界区和对其互斥访问的管理。
  - 管程的并行是什么含义呢？

---
**管程的使用实例**
```
Monitor account {
  double balance;
  double withdraw(amount) {
  balance = balance –amount;
  return balance;
  }
}
```

![例子](http://upload-images.jianshu.io/upload_images/4984976-44d3d3849cd8f1e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

----

---
###如果一个线程想在管程中等待呢？——条件变量（condition variables）

**Attention：**
>共享变量并不是`if`语句中的判断条件，所以不能`if(condition) { ... }`

**条件变量是：**
- `Sleep`:线程需要的资源无法获取的时候，让线程等待
- `Wake(or signal)`:资源可以访问的时候叫醒线程
- 'Wakeall (or signalAll)':叫醒所有等待线程
条件变量就是实现上述操作的一种方式

---
**条件变量 & 锁**

条件变量不能代替锁，而是和锁互补的一种机制
- Wait(condition, lock)
> 首先释放锁，将线程放到`condition`的等待队列中，如果线程再醒过来，将重新获得锁。
在放在等待队列的时候，会让线程`sleep`，等`sleep`返回的时候，它是被另外一个持有锁的线程叫醒的。

- Signal(condition)
> 叫醒一个在condition的等待队列上的线程

- Broadcast(condition)
> 叫醒所有在 `condition`的等待队列上的线程

---
**条件变量 & 管程：**

管程中的一个条件变量一般表示的含义是：一个线程要在管程中继续运行所需要的条件（比如消费者需要资源不为空才能消费）

```
Monitor M {
    ...monitored variables;
    Condition c, d;
    
    void enter_monitor(...) {
        if( extra property C not true) wait(c); //等待管程的锁
        do what you have to do 
        if(extra property true D) signal(d);//叫起在d上等待的线程
    }
}
```
一般条件变量对应的操作为：
`Wait( ) :` 等待管程的锁，或者等待条件变量被signal了
`Signal( ):’ 叫醒一个等待线程
'Broadcase( ):' 叫醒所有等待线程

---
**条件变量 != 信号量**

有条件变量的管程 != 信号量
> 但它们可以相互实现

其区别在于，对管程访问用锁控制，下面是具体分析：
- `wait()`操作会阻塞当前线程，并放开锁
  - 线程要能进行`wait()`操作，必须在管程内部，也就是必须持有锁
  - Semaphore::P只是阻塞了在队列上的线程，线程还没有持有信号量
- `Signal()`使得一个等待线程被唤醒
  - 如果没有等待的线程，这个信号其实没有用
  - 但对Semaphore::V()会增加信号量的值，使得将来其他线程能访问
  - 也就是Condition没有历史，但Semaphore是有的。


---

---
###使用管程和条件变量解决实际问题

---

**（一）有限缓冲区问题描述：**

 有一个被producer和consumer共享的资源池
 - producer向其中放入资源
 - consumer从其中拿出资源消耗。

producer和consumer执行速度不同：
 - 没有严格的串行化
 - 任务是独立执行的
 - 在buffer上不会发生线程切换

安全要求：
 - 如果nc是消费了的资源数，np是生产了的资源数，N是buffer的大小。则`0 ≤ np - nc ≤ N`

**有限缓冲区代码**
```
Monitor bounded_buffer {
    Resource buffer[N];
    //Variables for indexing buffer
    Condition not_full;  //space in buffer;
    Condition not_empty; //value in buffer
    
    void put_resource (Resource R){
        if(buffer array is full) 
            wait(not_full);
        Add R to buffer array;
        signal(not_empty);
    }
    Resource get_resource() {
        if(buffer array is empty)
            wait(not_empty);
        Get resource R from buffer array;
        signal(not_full);
        return R;
    }
}
```

---
**（二）读者写者问题**

分析：
- 使用Mesa管程，有四个函数`StartRead` `StartWrite` `EndRead` `EndWrite`
- 用`nr` `nw`分别表示读者和写者的数量
- 在`nw = 0`的时候可以读
- 在`(nw = 0) && (nr = 0)`的时候可以写

实现：
```
Monitor RW {
    int nr = 0, nw = 0;
    Condition canRead, canWrite;
    
    void StartRead() {
        while(nw != 0) wait(canRead);
        nr++;
    }
    void EndRead() {
        nr--;
        if(nr == 0) signal(canWrite);
    }
    void Start Write() {
        while(nr != 0 || nw != 0) wait(canRead);
        nw++;
    }
    void EndWrite() {
        nw--;
        signal(canWrite);
        signal(canRead);
    }
}
```

----
###两种不同的管程实例——Hoare管程和Mesa管程

---
两者的区别主要是`signal()`的语义不一样

**Hoare monitors（original）**
- `signal()`立刻就会进行线程切换，即将调用`signal()`的线程切换下CPU，让被`wakeup`的线程运行
- `condition`一定会被这个被`wakeup`的线程一直持有

**Mesa monitors（Mesa，Java）**
- `signal()`的时候，是把一个线程叫醒然后放入`readyList`，然后继续执行当前线程
- `condition`在不一定被这个被`wakeup`的线程持有，线程再一次进入管程的时候，必须检查`condition`是否满足。（因为可能被`readyList`上前面的线程用掉了）

---
**具体使用的区别：**

Hoare：
```
if(empty)
    wait(condition)
```
Mesa：
```
while(empty)
    wait(condition)
```

---
**比较：**

Mesa管程更易于使用，也更有效
- 因为上下文的切换比较少，而且容易支持broadcast

Hoare管程则相对不够灵活
- 但更容易理解
