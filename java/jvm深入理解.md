### java内存区域与内存溢出

### 程序启动

打包使用lifecycle的package

启动  java -Xmx512M -Xms512M -XX:+PrintGCDetails  -cp .\xx.jar bio.BIOServer ，cp是classpath的意思

#### 发生了什么

向用户空间申请了一片内存空间，最大为-Xmx512M 最小为-Xms512M的jvm内存空间，启动一个jvm进程

jinfo 查看

![1615704442802](C:\Users\wqkant\AppData\Roaming\Typora\typora-user-images\1615704442802.png)

#### 如何启动进程

**Java → JVM → glibc → 内核**

https://zhuanlan.zhihu.com/p/202768672

JVM 的入口：java.base/share/native/launcher/main.c

首先，调用 CreateExecutionEnvironment 查找设置环境变量

其次，调用 LoadJavaVM 加载 JVM，就是 libjvm.so 文件，然后找到创建 JVM 的函数赋值给 InvocationFunctions 的对应字段

最后，调用 JVMInit 初始化 JVM，load Java 程序

#### linux 内存的三个层面

![img](http://elsef.com/img/in-post/hardware-os-user-mem.png)

BIN:引导系统

从进程的角度来看，进程能直接访问的用户内存（虚拟内存空间）被划分为5个部分：代码区、数据区、堆区、栈区、未使用区。（冯诺依曼结构）

- 代码区中存放应用程序的机器代码，运行过程中代码不能被修改，具有只读和固定大小的特点。
- 数据区中存放了应用程序中的全局数据，静态数据和一些常量字符串等，其大小也是固定的。
- 堆是运行时程序动态申请的空间，属于程序运行时直接申请、释放的内存资源。
- 栈区用来存放函数的传入参数、临时变量，以及返回地址等数据。
- 未使用区是分配新内存空间的预备区域。

#### jvm进程和普通进程的比较

![img](http://elsef.com/img/in-post/process-jvm.png)

#### jvm 的内存区域

1. 程序计数器

2. 线程栈：

   局部变量（基本类型，对象引用，returnAddress类型）

   槽位大小确定，变量大小->槽位大小（long，double占2，其余占1）->字节大小（32，64）

3. 本地方法栈：调用本地方法的

4. 堆

   垃圾回收是自动行为，普通程序不具备。

   自动回收主要发生在堆空间，所以相应的内存布局（逻辑划分）就不同。如果采用不同的回收算法，相应的逻辑划分也会不同。

5. 方法区（被元空间替代，使用直接内存，因为之前容易内存溢出）：类型信息、常量、静态变量等

6. 运行时常量池：放在方法区的，运行时也可以加入数据(String.intern)

7. 直接内存：java nio

注：默认情况下元空间大小是无限的，但是JVM同样提供了参数来控制它的使用

```text
-XX:MetaspaceSize
    class metadata的初始空间配额，以bytes为单位，达到该值就会触发垃圾收集进行类型卸载，
    同时GC会对该值进行调整：如果释放了大量的空间，就适当的降低该值；如果释放了很少的空间，
    那么在不超过MaxMetaspaceSize（如果设置了的话），适当的提高该值。
-XX：MaxMetaspaceSize
    可以为class metadata分配的最大空间。默认是没有限制的。
-XX：MinMetaspaceFreeRatio
    在GC之后，最小的Metaspace剩余空间容量的百分比，减少为class metadata分配空间导致的垃圾收集。
-XX:MaxMetaspaceFreeRatio
    在GC之后，最大的Metaspace剩余空间容量的百分比，减少为class metadata释放空间导致的垃圾收集。
```

#### 对象、头、域

很多消息设计都具有这样的特征，分为，信息头，信息体，值得借鉴

头：相当于描述信息

​	hash码，gc分代年龄，锁状态，线程持有的锁，偏向锁id...

​	类型指针：不一定要有，可以有其他方式获得，如果是数组，还得记录大小

域：相当于具体内容

​	指针：指向内存中的某块区域

​	非指针：值本身

