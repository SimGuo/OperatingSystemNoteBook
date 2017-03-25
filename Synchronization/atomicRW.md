#Critical Sections with atomic read/write
---

###下面是利用原子读写操作的两种算法
>在下面的算法中对数据的读写都是原子的

---

**算法一**
两段程序如下：

（1）
```
while(true){
    while(turn == 2);
    //critical section
    turn = 2;
    //outside of critical section
}
```
（2）
```
while(true){
    while(turn == 1);
    //cirtical section
    turn = 1;
    //outside of critical section
}
```
这个解法存在的问题是：
如果（1）在临界区之外进入了一个无限的循环，然后（2）在一次进入临界区之后，将turn修改为了1，则（2）之后就永远进不了临界区了。

---
**Peterson的算法**
```
while(true){
    ready1 = true;
    turn = 2;
    while(turn == 2 && ready2);
    //critical section
    ready1 = false;
    //outside of critical section
}
```
```
while(true) {
    ready2 = true;
    turn = 1;
    while(turn == 1 && ready1);
    //critical section
    ready2 = false;
    //outside of critical section
}
```
Peterson算法是可行的，也可以扩展到N个线程。（在Wiki中有相关的扩展：filter算法）
