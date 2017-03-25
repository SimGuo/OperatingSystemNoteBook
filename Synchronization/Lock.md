#Locks

---
---

**与锁有关的操作**
- ```Acquire()``` ：在进入临界区前调用
- ```Release()```：在离开临界区后调用

      有的操作系统可能会用```Lock()/Unlock()```来代替

- 在```Acquire()```和```Release()```之间，线程将一直占有这个锁。只有当占有锁的线程释放锁的时候，线程才能获得锁。
- ```Acquire()```和```Release()```一定是成对出现的。
---
**一个简单的锁的使用**
应用在银行取款：
```
withdraw(account, amount) {
    acquire(lock);
    balance = get_balance(account);
    balance = balance - amount;
    put_balance(account, balance);
    release(lock);
    return balance;
}
```

---
###锁可以是**自旋的**（a spinlock），也可以是**阻塞的**（a mutex）

---

---

**那如何实现自旋锁呢？**
- 实现一：
 ```
struct lock{
int held = 0;
}
void acquire(lock) {
    while(lock->held);
    lock->held = 1;
}
void release(lock){
    lock->held = 0;
}
```
之所以被称为自旋锁，是因为线程在锁没有被释放的时候并不会进入睡眠状态，而是不停地检查锁是否被释放了。
但这样的实现是**存在问题**的。因为两个自旋中的线程可能同时发现held变成了0，而跳出循环。

实现一给我们的启示：
 - 在实现一中```lock->held```其实是一个新的共享变量，会带来新的临界区。这样无限循环，无法解决问题。
 - ```Acquire()```和```Release()```操作必须是原子的。

---
---

**那么又该如何实现原子操作呢？**  ——————我们需要硬件的支持：

---
硬件支持（一）：一些原子的指令（如：test-and-set指令）
      ```
      bool test_and_set (bool *flag) {
            bool old = *flag;
            *flag = TRUE;
            return old;
      }
      ```
- test-and-set的语义：
>①记录旧值 ②将值设置为TRUE ③返回旧值
- 这是由硬件自动实现的
- Swap/SCHG指令
      ```
      void Swap (char* x,* y) { // All done atomically
            char temp = *x;
            *x = *y;
            *y = temp
      }
      ```
- 使用Swap实现test-and-set
     ```
      bool test_and_set (bool *flag) {
      bool X = TRUE;
      Swap(&X, &flag);
      return X;
      }
     ```
---

开关中断
  - 对应代码：
    ```
    structlock { };
    void acquire (lock) {
        disable interrupts;
    }
    void release (lock) {
        enable interrupts;
    }
    ```
  - 开关中断在实际系统中是不具有可行性的。因为在实际系统中，只有kernel才有资格开关中断，而且随意开关中断也会导致很多严重的后果。所以只有在很高要求的同步才会使用中断。
  - 在有多处理器时，关中断也是不足够的

---
**根据上述实现的自旋锁**
根据test-and-set指令：
```
struct lock {
    int held = 0;
}
void acquire(lock) {
    while(test_and_set(&lock->held);
}
void release(lock) {
    lock->held = 0;
}
```

- 只有当一开始```lock->held == 0```的时候才不会进入忙等
- 这在多处理器上也是可行的！

**因为同时只有一个CPU可以占有内存总线，只有一个CPU能够读到```lock->held == 0```的情况**

---
**自旋锁存在的问题**
- 使用自旋锁会使得线程在占用CPU的时候，只是在不停地忙等而不做任何事情。所以是开销很大的。
- 解决方法：
  - 没有锁就主动thread_yield()，来下CPU
  - 没有锁就进入睡眠状态，直到可以的时候再上CPU

---
**在锁的使用中可能存在如下问题：**
- 临界区可能很长，其他线程可能等待的时间很长，而且持有锁的线程随时可能下CPU。
