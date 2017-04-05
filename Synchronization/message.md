#Messages

---

---

`messages` `barrier`

适用于分布式系统

----
####Message

**消息机制**是利用 原子的消息传递 解决进程间通信和同步问题的机制。

**两个原语：**
 - send(destination, &message);
 - receive(source, &message);

**相应会有的问题：**
 - 信息和动作的丢失
 - 消息的顺序
 - 命名
 - 消息的真实性和可靠性

----
####使用message解决生产者消费者问题

```
#define N 100
void producer(void){
    int item;
    message m;

    while(TRUE) {
        item = produce_item();
        receive(consumer, &m);
        build_message(&m, item);
        send(consumer, &m);
    }
}

void consumer(void){
    int item, i;
    message m;
    for(i = 0; i < N; i++) send(producer, &m);
    while(TRUE) {
        receive（producer, &m);
        item = extract_item(&m);
        send(producer, &m);
        consume_item(item);
    }
}
```

---
####Barrier机制

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4984976-de50e5b99b62320d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

barrier机制是为了实现如上图所示，所有进程到达barrier才能继续进行的机制。
