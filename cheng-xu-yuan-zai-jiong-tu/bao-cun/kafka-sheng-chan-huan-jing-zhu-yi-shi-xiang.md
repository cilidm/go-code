# Kafka生产环境注意事项

**目录**

* 前提
* 硬件配置
* 参数设置
* 优化
* 测试
* 疑难杂症
* 其他

**前提**

**数据规划**

* 数据采用28法则。
* 一天有24小时,凌晨12点-8点没有太多用户,所以一天按照16小时计算。
* 用户流量峰值1w人10分钟为800w条数据,三小时则为9600w条数据
* 其余时间十三小时,10分钟为200w条数据,十三小时为1.5亿条数据
* 所以大概得出一天产出的数据为2.4亿
* 数据保留7天

**硬件配置**

**磁盘方案**

**磁盘选择**

* 因为kafka为顺序写,所以使用机械硬盘足够

**硬盘容量**

* 如果每条消息的数据为5kb,保存两份,则数据需要的磁盘空间为 2.4亿5kb2/1000/1000=2tb,数据保留7天,也就是2tb\*7=为14t,kafka支持压缩,比例为0.75，所以最后规划的存储空间为10.5tb硬盘

**QPS**

* 假定说一天需要消费2.4亿数据,按照单机的标准来说,所以需要机器的吞吐量为2.4亿/24/60/60 所以单机的的qps在3000q/s 就足够。

**内存**

* kafka分配的jvm堆内存，6-10G足够,其他的内存留给os cache。
* 如果说topics一共有100个,单机的情况下,一个topics设置一个partition,就会有100个partition,每个partition假定大小是是1g，就需要100G内存，但是数据不需要全部放内存，所以0.25 为，25G内存，所以在选择kafka机器时候，单机情况下，32G内存。

**cpu**

* 建议8-16H,可以Hold住100-200个线程。

**网卡**

* 如果使用的千兆网卡,在我们高峰期,每秒需要处理13333条数据，每条消息5kb,也就是说每秒需要处理65m。
* 按照宽带的计算,kafka应该只能占用所有机器网络的2/3，所以则为700m,也不是一直占用全部的700m的高峰期，所以为1/3,所以为240M,所以可以得出。在现在业务压力下，单机应该是能满足。

**小结**

* 所以如果在使用单服务器的情况下，为了抗住一天有2.4亿的日志，日志每条为5kb，并且，日志保存7天的情况下,需要的配置如下
* 16h32g 硬盘11t,千兆网卡 单机

**参数设置**

**kafka参数设置**

* [broker.id](http://broker.id/) 每台node的borker值唯一,可以从0开始。
* log.dirs Kafka的日志目录,可以多个。
* zookeeper.connect 连接kafka底层的zookeeper集群
* listeners：学名叫监听器，其实就是告诉外部连接者要通过什么协议访问指定主机名和端口开放的 Kafka 服务,如果需要配置外网和内网都能访问,建议这里配置成0.0.0.0:端口
* advertised.listeners 主要是为外网访问用的。如果clients在内网环境访问Kafka不需要配置这个参数。
* unclean.leader.election.enable 这个是是否落后的副本还能参加选举吗?默认为false。
* auto.create.topics.enable 是否自动创建topic,生产环境建议设置为false。
* auto.leader.rebalance.enable 是否自动重新选举leader,生产环境建议为false。
* log.retention.{hour\|minutes\|ms}：控制一条消息数据被保存多长时间。从优先级上来说 ms 设置最高、minutes 次之、hour 最低,默认为7天,一般使用log.retention.hour
* log.retention.bytes 这是指定 Broker 为消息保存的总磁盘容量大小,如果设置为-1,则为默认无限制存储。
* message.max.bytes 控制 Broker 能够接收的最大消息大小
* delete.topic.enable 允许删除topic
* min.insync.replicas 这依然是 Broker 端参数，控制的是消息至少要被写入到多少个副本才算是“已提交”。设置成大于 1 可以提升消息持久性。在实际环境中千万不要使用默认值 1。
* log.cleaner.enable 是否开启日志压缩
* kafka-start-server.sh中的jvm设置\(生产环境配置\)

export KAFKA\_HEAP\_OPTS=”-Xmx6g -Xms6g -XX:MetaspaceSize=96m -XX:+UseG1GC - XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 - XX:G1HeapRegionSize=16M -XX:MinMetaspaceFreeRatio=50 - XX:MaxMetaspaceFreeRatio=80”

**生产者参数设置**

* buffer.memory 设置发送消息的缓冲区，默认值是33554432，就是32MB 如果发送消息出去的速度小于写入消息进去的速度，就会导致缓冲区写满，此时生产消息就会阻塞住，所以说 这里就应该多做一些压测，尽可能保证说这块缓冲区不会被写满导致生产行为被阻塞住
* compression.type 默认是none，不压缩，但是也可以使用lz4压缩，效率还是不错的，压缩之后可以 减小数据量，提升吞吐量，但是会加大producer端的cpu开销
* batch.size 设置每个batch的大小，如果batch太小，会导致频繁网络请求，吞吐量下降；如果 batch太大，会导致一条消息需要等待很久才能被发送出去，而且会让内存缓冲区有很大压力，过多数据缓冲 在内存里 默认值是：16384，就是16kb，也就是一个batch满了16kb就发送出去，一般在实际生产环境，这个batch 的值可以增大一些来提升吞吐量
* [linger.ms](http://linger.ms/) 这个值默认是0，意思就是消息必须立即被发送，但是这是不对的，一般设置一个100毫秒之 类的，这样的话就是说，这个消息被发送出去后进入一个batch，如果100毫秒内，这个batch满了16kb，自 然就会发送出去。但是如果100毫秒内，batch没满，那么也必须把消息发送出去了，不能让消息的发送延迟 时间太长，也避免给内存造成过大的一个压力
* max.request.size 这个参数用来控制发送出去的消息的大小，默认是1048576字节，也就1mb，这个 一般太小了，很多消息可能都会超过1mb的大小，所以需要自己优化调整，把他设置更大一些
* [request.timeout.ms](http://request.timeout.ms/) 这个就是说发送一个请求出去之后，他有一个超时的时间限制，默认是30秒， 如果30秒都收不到响应，那么就会认为异常，会抛出一个TimeoutException来让我们进行处理
* request.required.acks=0,发送了就不管是否插入成功。
* request.required.acks=1,只要leader写入成功,就认为消息写入成功了。
* request.required.acks=all/-1,这个leader写入成功以后，必须等待其他ISR中的副本都写入成功，才可以 返回响应说这条消息写入成功了，此时你会收到一个回调通知
* max.in.flight.requests.per.connection=1 为了保证发出消息是顺序的,设置同一时间只能发出一条。
* retries 重试次数
* [retry.backoff.ms](http://retry.backoff.ms/) 重试间隔时间

**消费者参数设置**

* [heartbeat.interval.ms](http://heartbeat.interval.ms/) consumer心跳时间，必须得保持心跳才能知道consumer是否故障了，然后如果故障之后，就会通过心跳下 发rebalance的指令给其他的consumer通知他们进行rebalance的操作
* [session.timeout.ms](http://session.timeout.ms/) kafka多长时间感知不到一个consumer就认为他故障了，默认是10秒
* [max.poll.interval.ms](http://max.poll.interval.ms/) 如果在两次poll操作之间，超过了这个时间，那么就会认为这个consume处理能力太弱了，会被踢出消费 组，分区分配给别人去消费
* fetch.max.bytes 获取一条消息最大的字节数
* max.poll.records 一次poll返回消息的最大条数，默认是500条
* [connection.max.idle.ms](http://connection.max.idle.ms/) consumer跟broker的socket连接如果空闲超过了一定的时间，此时就会自动回收连接，但是下次消费就要 重新建立socket连接，这个建议设置为-1
* auto.offset.reset 生产环境latest
* enable.auto.commit 开启自动提交
* [auto.commit.ineterval.ms](http://auto.commit.ineterval.ms/) 多久条件一次偏移量

**优化**

**操作系统调优**

* 文件系统使用ext4/xfs
* 禁用swap空间
* ulimit -n 设置大一些。
* vm.max\_map\_count 设置大一些。

**JVM** **层调优**

* 堆大小设置6-8G
* 使用G1收集器
* 如果遇到大对象，在启动参数增加-XX:+G1HeapRegionSize=N

**性能指标优化**

* 适当增加num.replica.fetchers的参数，不能超过cpu核心数
* 避免经常性的full gc
* 适当增加batch.size数值
* [适当增加linger.ms](http://xn--linger-2e4j090brvo308i.ms/)
* 设置compression.type=lz4/zstd
* 设置acks=0/1
* 设置retries=0
* 如果多线程共享一个producer 就增加buffer.memory数值
* 多consumer进程线程同事消费数据
* 增加fetch,min.bytes参数值

**应用层优化**

* 不要频繁创建producer和consummer对象实例，用完及时关闭。合理利用多线程改善性能

**调优延时**

* 适当增加num.replica.fetchers的参数
* 设置linger.ms=0
* 不启用压缩
* 设置acks=1
* 设置fetch,min.bytes=1

**测试**

* 使用kafka-producer-perf-test.sh和kafka-consumer-perf-test.sh进行压力测试。

**疑难杂症**

**怎么保证product数据一定能够发送成功?**

* 不要使用 producer.send\(msg\)，而要使用 producer.send\(msg, callback\)。
* 设置 retries 为一个较大的值
* 设置 unclean.leader.election.enable = false。这是 Broker 端的参数，它控制的是哪些 Broker 有资格竞选分区的 Leader。
* 设置 replication.factor &gt;= 3。这也是 Broker 端的参数。其实这里想表述的是，最好将消息多保存几份，毕竟目前防止消息丢失的主要机制就是冗余
* 设置 min.insync.replicas &gt; 1。这依然是 Broker 端参数，控制的是消息至少要被写入到多少个副本才算是“已提交”。设置成大于 1 可以提升消息持久性。在实际环境中千万不要使用默认值 1。
* 确保 replication.factor &gt; min.insync.replicas。如果两者相等，那么只要有一个副本挂机，整个分区就无法正常工作了。我们不仅要改善消息的持久性，防止数据丢失，还要在不降低可用性的基础上完成。推荐设置成 replication.factor = min.insync.replicas + 1。
* 确保消息消费完成再提交。Consumer 端有个参数 enable.auto.commit，最好把它设置成 false，并采用手动提交位移的方式。就像前面说的，这对于单 Consumer 多线程处理的场景而言是至关重要的
* 可以先采用异步发送，如果失败再采用同步发送，如果还失败，就报错，手动提交的方式。

**重平衡危害很大,怎么避免它?**

**重平衡出现的原因?**

* 组成员的数量发送变化
* 订阅主题数量发生变化
* 订阅主题的分区数发生变化

**导致的问题?**

* 重平衡会导致consumer重新开始消费，并且会等重平衡完毕之后才会继续处理数据。

**解决方案**

* 除开组成员之外，其他2个都会是运维的主动操作，所以我们需要避免组成员的数量变化。
* [session.timeout.ms](http://session.timeout.ms/) = 6s ,[heartbeat.interval.ms](http://heartbeat.interval.ms/) = 2s 保证一定不要是消费者没有及时发送心跳导致的重平衡。
* 可能是消费者的下游程序消费时间过长导致的重平衡，[所以根据业务情况设置max.poll.interval.ms](http://xn--max-o48d89aq9igybp31dm1clxdvytc11ezp7a.poll.interval.ms/)

**如何保证消息不重复消费?**

* 业务中设置前置条件,使用幂等方法。
* 保存数据的时候先查询是否数据已经保存。

**怎么保证数据是顺序发送？**

* max.in.flight.requests.per.connection =1 同一时间只能发送一个

**怎么保证数据是顺序消费？**

* 单消费者，单分区
* 多消费者，多分区,指定topic的key,一个消费者只能消费一个分区。

**其他**

* [kafka和zookeeper安装连接](http://note.youdao.com/s/Kf7Ac9TK)

