### jps

```
jps -l 输出jar包路径，类全名
jps -m 输出main参数
jps -v 输出JVM参数
```

### jinfo

查看JVM参数和系统配置

jinfo -flags

jinfo -flag PrintGC

jinfo -flag PrintGCDetails

打开GC日志参数

jinfo -flag +PrintGC

jinfo -flag +PrintGCDetails

关闭GC日志参数

jinfo -flag -PrintGC

jinfo -flag -PrintGCDetails

### jstat

option参数解释：

- -class class loader的行为统计
- -compiler HotSpt JIT编译器行为统计
- -gc 垃圾回收堆的行为统计
- -gccapacity 各个垃圾回收代容量(young,old,perm)和他们相应的空间统计
- -gcutil 垃圾回收统计概述
- -gccause 垃圾收集统计概述（同-gcutil），附加最近两次垃圾回收事件的原因
- -gcnew 新生代行为统计
- -gcnewcapacity 新生代与其相应的内存空间的统计
- -gcold 年老代和永生代行为统计
- -gcoldcapacity 年老代行为统计
- -gcpermcapacity 永生代行为统计
- -printcompilation HotSpot编译方法统计

字段解释:

- S0 survivor0使用百分比
- S1 survivor1使用百分比
- E Eden区使用百分比
- O 老年代使用百分比
- M 元数据区使用百分比
- CCS 压缩使用百分比
- YGC 年轻代垃圾回收次数
- YGCT 年轻代垃圾回收消耗时间
- FGC 老年代垃圾回收次数
- FGCT 老年代垃圾回收消耗时间
- GCT 垃圾回收消耗总时间

### jstack

jstack是用来查看JVM线程快照的命令，线程快照是当前JVM线程正在执行的方法堆栈集合。使用jstack命令可以定位线程出现长时间卡顿的原因，例如死锁，死循环等。jstack还可以查看程序崩溃时生成的core文件中的stack信息。

option参数解释：

- -F 当使用jstack <pid>无响应时，强制输出线程堆栈。
- -m 同时输出java和本地堆栈(混合模式)
- -l 额外显示锁信息

https://www.hollischuang.com/archives/110

### jmap

jmap是用来生成堆dump文件和查看堆相关的各类信息的命令，例如查看finalize执行队列，heap的详细信息和使用情况

option参数解释：

- <none> to print same info as Solaris pmap
- -heap 打印java heap摘要
- -histo[:live] 打印堆中的java对象统计信息
- -clstats 打印类加载器统计信息
- -finalizerinfo 打印在f-queue中等待执行finalizer方法的对象
- -dump:<dump-options> 生成java堆的dump文件

　　　　　　dump-options:

　　　　　　live 只转储存活的对象，如果没有指定则转储所有对象

　　　　　　format=b 二进制格式

　　　　　　file=<file> 转储文件到 <file>

- -F 强制选项

### jhat

jhat是用来分析jmap生成dump文件的命令，jhat内置了应用服务器，可以通过网页查看dump文件分析结果，jhat一般是用在离线分析上。

option参数解释：

- -stack false: 关闭对象分配调用堆栈的跟踪
- -refs false: 关闭对象引用的跟踪
- -port <port>: HTTP服务器端口，默认是7000
- -debug <int>: debug级别

　　 0: 无debug输出

 

　　 1: Debug hprof file parsing

 

　　 2: Debug hprof file parsing, no server

 

- -version 分析报告版本

### jconsloe,jvisualvm

