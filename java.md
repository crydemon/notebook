## 多线程



### 守护线程



### 非守护线程



### interruptedException，interrupt

能够被中断的阻塞称为轻量级阻塞，对应的线程状态是WAITING或者TIMED_WAITING；而像synchronized 这种不能被中断的阻塞称为重量级阻塞，对应的状态是BLOCKED

除了常用的阻塞/唤醒函数，还有一对不太常见的阻塞/唤醒函数，LockSupport.park（）/unpark（）。这对函数非常关键，Concurrent包中Lock的实现即依赖这一对操作原语。故而t.interrupted（）的精确含义是“唤醒轻量级阻塞”，而不是字面意思“中断一个线程”。

因为t.interrupted（）相当于给线程发送了一个唤醒的信号，所以如果线程此时恰好处于WAITING或者TIMED_WAITING状态，就会抛出一个InterruptedException，并且线程被唤醒。而如果线程此时并没有被阻塞，则线程什么都不会做。但在后续，线程可以判断自己是否收到过其他线程发来的中断信号，然后做一些对应的处理，这也是本节要讲的两个函数。这两个函数都是线程用来判断自己是否收到过中断信号的，前者是非静态函数，后者是静态函数。二者的区别在于，前者只是读取中断状态，不修改状态；后者不仅读取中断状态，还会重置中断标志位。

### wait，notify

在wait（）的内部，会先释放锁obj1，然后进入阻塞状态，之后，它被另外一个线程用notify（）唤醒，去重新拿锁！其次，wait（）调用完成后，执行后面的业务逻辑代码，然后退出synchronized同步块，再次释放锁。

### volatile，dcl

### happen-before

### 内存屏障

根据注释，我们知道：

loadFence=LoadLoad+LoadStore

storeFence=StoreStore+LoadStore

fullFence=loadFence+storeFence+StoreLoad

### final

### synchronized

在对象头里，有一块数据叫Mark Word。在64位机器上，Mark Word是8字节（64位）的，这64位中有2个重要字段：锁标志位和占用该锁的thread ID

### 缓存一致性

cpu缓存：l1，l2，l3

### cas

同内存屏障一样，CAS（Compare And Set）也是CPU提供的一种原子指令

### atomic类

Unsafe类是整个Concurrent包的基础，里面所有函数都是native的

当一个线程拿不到锁的时候，有以下两种基本的等待策略。

策略1：放弃CPU，进入阻塞状态，等待后续被唤醒，再重新被操作系统调度。

策略2：不放弃CPU，空转，不断重试，也就是所谓的“自旋”。

AtomicInteger的实现就用的是“自旋”策略，如果拿不到锁，就会一直重试。有一点要说明：这两种策略并不是互斥的，可以结合使用。如果拿不到锁，先自旋几圈；如果自旋还拿不到锁，再阻塞，synchronized关键字就是这样的实现策略。

要解决Integer或者Long型变量的ABA问题，为什么只有AtomicStampedReference，而没有AtomicStampedInteger或者AtomictStampedLong呢？因为这里要同时比较数据的“值”和“版本号”，而Integer型或者Long型的CAS没有办法同时比较两个变量，于是只能把值和版本号封装成一个对象，也就是这里面的Pair 内部类，然后通过对象引用的CAS来实现

### lock、condition

Condition本身也是一个接口，其功能和wait/notify类似。在讲多线程基础的时候，强调wait（）/notify（）必须和synchronized一起使用，Condition也是如此，必须和Lock一起使用。因此，在Lock的接口中，有一个与Condition相关的接口

lock.newCondition()。

首先，读写锁中的ReadLock 是不支持Condition 的，读写锁的写锁和互斥锁都支持Condition。

每一个Condition对象上面，都阻塞了多个线程。因此，在ConditionObject内部也有一个双向链表组成的队列。





![1622043365618](C:\Users\wqkant\AppData\Roaming\Typora\typora-user-images\1622043365618.png)



### AQS原理

https://www.cnblogs.com/waterystone/p/4920797.html

![1622043783108](C:\Users\wqkant\AppData\Roaming\Typora\typora-user-images\1622043783108.png)

Sync的父类AbstractQueuedSynchronizer经常被称作队列同步器（AQS），这个类非常关键，下面会反复提到，该类的父类是AbstractOwnableSynchronizer

state取值不仅可以是0、1，还可以大于1，就是为了支持锁的可重入性。例如，同样一个线程，调用5次lock，state会变成5；然后调用5次unlock，state减为0。

在当前线程中调用park（），该线程就会被阻塞；在另外一个线程中，调用unpark（Thread t），传入一个被阻塞的线程，就可以唤醒阻塞在park（）地方的线程。尤其是unpark（Thread t），它实现了一个线程对另外一个线程的“精准唤醒”。前面讲到的wait（）/notify（），notify也只是唤醒某一个线程，但无法指定具体唤醒哪个线程。

### 读写锁

![1622044251755](C:\Users\wqkant\AppData\Roaming\Typora\typora-user-images\1622044251755.png)

readerLock和writerLock实际共用同一个sync对象

也就是把state 变量拆成两半，低16位，用来记录写锁。但同一时间既然只能有一个线程写，为什么还需要16位呢？这是因为一个写线程可能多次重入。例如，低16位的值等于5，表示一个写线程重入了5次。这是因为无法用一次CAS 同时操作两个int变量，所以用了一个int型的高16位和低16位分别表示读锁和写锁的状态。

### StampedLock

![1622045646931](C:\Users\wqkant\AppData\Roaming\Typora\typora-user-images\1622045646931.png)

在AQS里面，当一个线程CAS state失败之后，会立即加入阻塞队列，并且进入阻塞状态。但在StampedLock中，CAS state失败之后，会不断自旋，自旋足够多的次数之后，如果还拿不到锁，才进入阻塞状态。为此，根据CPU的核数，定义了自旋次数的常量值。如果是单核的CPU，肯定不能自旋，在多核情况下，才采用自旋策略。

### 同步工具类

#### Semaphore

信号量，提供了资源数量的并发访问控制

考虑一个场景：一个主线程要等待10个Worker 线程工作完毕才退出，就能使用CountDownLatch来实现

## io

### 同步异步，阻塞非阻塞

https://leetcode-cn.com/circle/discuss/uHGOZo/

阻塞，非阻塞是指线程的状态

线程阻塞情况：

sleep

wait

等待锁、资源、

五种io模型

https://zhuanlan.zhihu.com/p/115912936


阻塞io

非阻塞io

io多路复用：由一个线程(select、poll、epoll)监控fd，询问线程询问监控线程好了没。

信号驱动io：select的轮询方式太暴力了，在调用sigaction时候建立一个SIGIO的信号联系，当内核数据准备好之后再通过SIGIO信号通知线程数据准备好后的可读状态，当线程收到可读状态的信号后，此时再向内核发起recvfrom读取数据的请求

![1622215077981](C:\Users\wqkant\AppData\Roaming\Typora\typora-user-images\1622215077981.png)

异步io：上面两个，是要主动去问询数据是否好了，为什么不能简简单单给内核发个read？

![1622215087373](C:\Users\wqkant\AppData\Roaming\Typora\typora-user-images\1622215087373.png)

