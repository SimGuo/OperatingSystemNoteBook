###Hardware Features for OS

 - 安全性（用户态和内核态）
 - 对指令的保护
 - 对内存的保护
 - 错误
 - 系统调用
 - 中断（时钟中断，I/O中断，等等）
 - 同步

----
操作系统是**事件驱动的（event-driven）**

----
**OS Control Flow**
操作系统启动之后，所有只有通过event才能进入内核态
 - event直接停止当前在执行的操作
 - 将系统的状态切换到内核态，然后调用与event相关的处理函数

内核对每一种事件类型准备了处理函数

当处理器遇到一种给定的事件类型，会
 - 先将控制权转移给OS里面的处理函数
 - 处理函数先保存程序的运行状态（PC，寄存器，等等）
 - 处理函数的具体功能被调用
 - 处理函数恢复系统状态，返回原来的程序

---
###Events

---

---


![EVENT](http://upload-images.jianshu.io/upload_images/4984976-03911f7b0f1c3f00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**分类**
 - 中断 （由外部事件引起的，unexpected）（异步的）
 - 异常（由正在执行的指令引起的，CPU解决一个fault或者trap需要软件的介入，deliberate）（同步的）
   - 缺页
   - 除零

---
相应的处理函数
 - 对用户态的fault
> 当用户进程出现不可恢复的错误的时候，操作系统通过杀死当前线程来解决
> - 如没有相应的Handler时
> - 先halt这个进程，然后将进程的状态写到文件中，接着杀死这个进程
> - 在Unix中，这个是很多信号的默认的处理函数

 - 对系统态的fault
> - 遇到的类型可能是：对NULL的访问，除零，没有定义的指令
> - 这些是致命的使系统崩溃的错误。
> - Windows会出现蓝屏（内核halt，状态写入core file，机器被锁住）

- 更进一步的fault的解决
> - 有一些错误可以修复（比如缺页），这种时候我们会返回导致错误的上下文
> - 有一些错误可以通知进程
>   - 内核态的处理函数会修改PC的值，使得从处理函数返回时，会跳转到用户程序自己写的处理函数
>   - 用户的处理函数也是要登记的
>   - 可以使用signal和APC实现

----
###系统调用

---

---

>只有操作系统可以直接使用物理设备，那用户态的程序该如何使用呢？
—— OS需要提供给用户程序有限的、使用硬件设备的间接的接口。

当一个用户程序想做一些超出自己权限的事情的时候，必须通过系统调用。

**系统调用的过程**是这样：
 - 传递一个参数，表示自己要调用什么
 - 保存调用者的状态，从而能返回
 - 引起一个异常
 - 从系统调用回来，恢复用户程序的状态

这样的流程需要：
 - 确定输入的参数（放在buffer固定的位置）
 - 恢复保存的状态，恢复到用户态，重新开始执行

---
**系统调用的函数**
 - 与进程控制有关的：
   - create process, allocate memory
 - 文件管理
   - create, read, delete file
 - 设备管理
   - open device, read/write device, mount device
 - 信息维护
   - get time, get system data/parameters
 - 交互
   - create/delete channel, send/receive message


---
程序员一般是不会直接写系统调用的，而是使用一些库函数（如C,java的库）。
—— 因为系统调用的具体实现与硬件的架构相关，而使用这些库函数的时候程序员就不需要知道在具体的架构上的系统调用的指令是什么了。只需要直接使用这个同一的接口。

---
**系统调用与函数调用的区别**

系统调用是通过int 80H的指令进行陷入的，而函数调用则是直接call即可。系统调用会发生上下文切换，函数调用则不会、

---
###中断

---

---
中断是异步的，有
 - 来自I/O等设备的硬件中断
 - 软件和硬件的计时器

两种风格的中断：
 - 明确的：只会在一条指令执行之后发生（更受OS设计者喜欢，因为这样他们编程的不确定性就少很多?）
 - 不明确的：可以在指令执行的期间发生。（CPU设计者喜欢，因为这样中断设计更及时？）

----
###Timer & I/O

**timer**对操作系统而言很重要，每个一段时间发送一个中断，timer中断的处理函数在内核态。

**I/O**是硬件和进程间的同步。设备与机器独立运行，在完成任务时，发送中断给CPU，OS的处理函数来处理这个中断。

---
**Interrupt Questions**
1. 中断中止了程序继续运行，并将控制权交给了操作系统。
那么操作系统可以被打断么？

2. 还有什么方法可以代替中断，这些方法又有什么劣势？

---

###软件中断（略）

---
