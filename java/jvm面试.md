### 分代回收

对象的生命周期不同，回收方式不同，提高回收效率

新生代：eden,s0,s1。minor gc，只有少量存活对象，复制算法。eden满，复制到 survivor中的一个，当这个survivor满时，复制到另一个。对象每经历一次minor gc，年龄加1。达到晋升年龄（在serial和parNew gc中MaxTenuringThreshold默认15），去老年代。

老年代：old generation。经历N次gc后，仍然存在，老年代。major gc。标记清理，标记整理。

持久代：perm。存放元数据（一种特殊的 Java 结构，用来修饰类、方法、字段、参数、变量、构造器或包。它是 JSR-175 选择用来提供元数据的工具。），如Class、Method元信息。

### gc 日志

gc参数：https://blog.csdn.net/yinni11/article/details/102591431

ps gc日志：https://blog.csdn.net/weixin_39889214/article/details/114232566

cms gc日志：https://www.jianshu.com/p/ba768d8e9fec

~~~
gc参数
-XX:+PrintGC 输出简要GC日志 
-XX:+PrintGCDetails 输出详细GC日志 
-Xloggc:gc.log  输出GC日志到文件
-XX:+PrintGCTimeStamps 输出GC的时间戳（以JVM启动到当期的总时长的时间戳形式） 
-XX:+PrintGCDateStamps 输出GC的时间戳（以日期的形式，如 2013-05-04T21:53:59.234+0800） 
-XX:+PrintHeapAtGC 在进行GC的前后打印出堆的信息
-verbose:gc
-XX:+PrintReferenceGC 打印年轻代各个引用的数量以及时长
~~~

### 空间分配

分析活跃数据：稳定存活的对象们占堆的空间（full gc后的大小）

总大小：3-4倍活跃数据空间

新生代：1-1.5倍活跃数据空间

老年代：2-3倍活跃数据空间

永久代：1.2-1.5倍，full gc后永久代占用空间

### 优化

https://tech.meituan.com/2017/12/29/jvm-optimize.html

高可用：多少个9

低时延：请求必须多少毫秒内完成响应

高吞吐：每秒完成多少事务

如果eden区较小，则会频繁发生minor gc。但增大一倍eden，不一定会减少一半minor gc。增加T1时间，减少T2时间。单次minor gc更多取决与gc后存活的对象大小（单次minor gc时间：T1（扫描新生代），T2(复制存活对象到survivor)）。

动态年龄计算（为了防止过早晋升和过晚晋升）：晋升年龄=min(累积的某个年龄大小超过一半的年龄，maxTenuringThreshold)

**如果短命的多，增大新生代大小，如果活得久，增大老年代**

### cms gc

init mark(stw) ：可达性分析(gc root 直连对象)，新生代minor gc使用根搜索算法。

concurrent-mark：由前面的节点，再扩大一下存活对象（bfs？）

remark(stw)：修正在concurrent-mark期间的修改。全堆扫描：跨代引用（新生代引用老年代），导致无法仅仅扫描老年代。优化：先并发预清除cms-concurrent-abortable-prelean（minor gc，默认eden = 2M启动），再cms。为了避免无限等等待minor gc，增加超时时间CMSMaxAortablePrecleanTime=5s。如果minor gc确实炒股5s，增加CMSScavengeBeforeRemark，强制minor gc。

并发清理。

优化：使用卡表，避免全堆扫描。

### full gc

触发因素：

perm不足。

cms gc出现promotion failed和concurrent model failure(old不足)。（关键字排查）

统计得到young gc晋升大小大于老年代的剩余空间。主动触发。（代码排查）

### 高性能建议

MaxPermSize和MinPermsize设置成一致（jdk1.8开始，Perm消失，使用元空间，直接内存中）

Xms和xmn相等。避免自动扩缩容

### 新生代为什么两个survivor(from to)

优点：效率，不会内存碎片

缺点：空间变小

### jvm 内存结构，运行时数据区

代码段，数据段(静态，常量)，bss（未初始化全局变量），stack（局部变量），heap（动态分配内存，先进先出）

线程：栈帧

堆（  字面量和符号引用）

metaSpace(直接内存)

虚拟机栈帧

本地方法栈

程序计数器（无omm）：线程私有，执行native方法，值为空，

### 内存泄露

无法回收不使用的对象

内存溢出：stackOverflow，outofMemory

### java 内存模型

通过共享内存实现多线程互相通信，

主内存：放共享变量

本地内存：线程私有

### 栈帧

jstack

### 双亲委派模型

解决安全性问题：始终加载正确的类路径

自定义ClassLoader，重写父类loadClass

