## kafka总结

### kafka能解决什么问题

1. 消息系统：系统解耦（生产，消费提供统一的中间层接口），冗余存储，流量削峰，缓冲，异步通信
2. 存储系统：消息持久化到磁盘，多副本机制
3. 流式处理平台
4. 分区：可伸缩、可水平扩展
5. 多副本：数据冗余、高可靠

### kafka基本组件和概念
#### producer



问题：

1. 消息发送后怎么确认broker确实收到了：send方法是异步的，但是会返回future，可以通过判断future的返回值确认消息的状态。发送策略：发后即忘、同步、异步。
2. 重试避免直接抛出异常：retries
3. 发送流程：kafkaProducer->ProducerInterceptions->Serializer->Partitioner->消息累加器RecordAccumulator(缓存批量，避免网络开销，大小配置buffer.memory)
4. RecordAccumulator结构：双端队列，Deque<ProducerBatch>。byteBuffer内存池，实现高效缓存。ProducerBatch是多个ProducerRecord（大小由batch.size控制）。<Node, List<ProducerBatch>>。没有收到响应的请求保存在InFlightRequests里
5. 如何保证发送消息分区有序的：消息序列号（类似tcp的序列号）
6. 如何保证消息的高可靠的：acks，多少副本写入成功，消息才算发送成功。1：主副本成功，0：发送即成功，-1/all：所有副本成功，严重影响吞吐量。这里的写入应该包括刷新到脏页了，不仅仅是落盘。
7. 缓存系统无处不在：将消息缓存层批，然后呢，socket里也有缓存，还有脏页缓存...

#### broker

问题：

1. 选主：
2. 保存了那些数据：
3. 分区副本与broker的对应：副本随机均等，机架
4. 减少分区：不能，保证分区有序性要付出代价。

#### consumer



问题：

1. 消费者组存在的意义：可以让消费者与分区形成m:n(m<n)的读取关系，加快读取速度（考虑网络瓶颈，磁盘io瓶颈）。一个分区只能对应一个消费者。
2. 负载问题：再均衡策略。前一个消费者的最新一次位移可能丢失，造成重复消费。再均衡期间，消费者组不能进行消费。可通过再均衡监听器缓解ConsumerRebalanceLisenter
3. 位移提交如何避免重复消费和消息丢失：5s自动提交（逻辑在poll里）显然无法满足。可配enable_auto_commit_config为false。手动同步提交会阻塞线程，手动异步提交失败导致重复消费：通过位移->序列号递增解决。
4. seek设置位移
5. 再均衡：重新分配分区给消费者？期间，暂停消费。再均衡监听器。
6. 消费者拦截器。
7. 多线程实现：将处理逻辑放到线程中，有点eventloop的味道。加锁提交位移。
8. 分区分配策略：rangeAssignor,randomRbinAssinger,stickyAssignor。

#### 分区

leader副本：读写只操作leader副本

follower副本：为了可靠性，当leader副本所在的机器挂了，方便恢复

消息追加：只保证分区的有序性

ar:assinged replicas

isr:in-sync replicas

osr:out-of-sync replicas

hw:high water，一个offset，大数据里也有。只能消费到之前的消息

leo:log end offset

调整副本：副本数，不能超过broker的数量。这个嘛，否则会冗余到一个机器，没有意义。

~~~
replication-factor.json：
{"version":1,
  "partitions":[
     {"topic":"t1","partition":0,"replicas":[0,1,2]},
     {"topic":"t1","partition":1,"replicas":[0,1,2]},
     {"topic":"t1","partition":2,"replicas":[0,1,2]},
     {"topic":"t1","partition":3,"replicas":[0,1,2]}
]}
.\kafka-reassign-partitions.bat --zookeeper localhost:2181 --reassignment-json-file ..\..\replication-factor.json --execute
~~~

负载均衡：leader副本集中在一个broker上，会造成produce过程负载不均衡。优先副本，均匀分布，选leader，偏向优先副本。kafka-perferred-replica-election.sh

分区重分配：节点下线，新增节点。kafka-reassign-partitions.bat。数据的复制。先增加新的副本，然后数据同步，删除老数据。注意对资源的占用，尽量小批量操作。有参数调整。

失效副本：

isr伸缩：定时任务

leader副本切换：leader epoch 通信，解决一致性问题

读写分离：有一致性，延时问题。kafka可以通过多种方式确保负载均衡。



#### 日志存储系统：

顺序写系统

配置存储位置：server.properties->log.dirs

调整分区数为4：

~~~
 bin/windows/kafka-topics.bat --alter --topic t1 --partitions 4 --bootstrap-server localhost:9092
~~~

![1621063333063](C:\Users\wqkant\AppData\Roaming\Typora\typora-user-images\1621063333063.png)

文件存储：t1为主题名，后缀为分区编号



运行期间，发现生产的消息并不是立即落盘，在结束kafka进程后落盘。检查缓存配置，刷盘策略：server.properties。这个强制刷盘是否应该开启，存疑。消息的可靠性，本质应该由应用自身保证。



问题：

1. 假设主机突然掉电，消息是否会因为缓存而导致丢失：消息确认机制，缓存的消息发送端怎么应对，mysql也有类似问题吧。操作系统的页缓存机制：vm.dirty_background_ratio设置脏页百分比，超过刷回磁盘（通过后台进程kdmflush）。脏页刷新机制可以了解一下。页缓存减少了系统调用，也将落盘任务委托给了操作系统。
2.  日志过大怎么加快磁盘读取：日志分段策略logJSegement（同内存分页），超过这个大小会生成新的.log及配套文件。其二，通过.index存储offset。offset格式：relativeOffset（偏移量）:position（磁盘位置）。只能写activeLogSegment。
3. 磁盘系统的选择：磁盘i/o流程：应用层->内核层-》块层->设备层。磁盘调度算法。文件格式：ext4，xfs。i/o策略：零拷贝sendfile，绕过用户态，通过dma，直接读到内核态

### 问题

**kafka分区的必要性**

负载均衡，水平扩展，效率（一个文件太大了，查询时间也大），并发

**kafka高可靠的原因**

副本嘛。leader副本，follower副本。消息确认机制，少数多数原则。

