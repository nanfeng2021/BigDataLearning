# Kafka面试题

1.Kafka 中的 ISR(InSyncRepli)、 OSR(OutSyncRepli)、 AR(AllRepli)代表什么？
2.Kafka 中的 HW、 LEO 等分别代表什么？
3.Kafka 中是怎么体现消息顺序性的？

4.Kafka 中的分区器、序列化器、拦截器是否了解？它们之间的处理顺序是什么？

5.Kafka 生产者客户端的整体结构是什么样子的？使用了几个线程来处理？分别是什么？

6.“消费组中的消费者个数如果超过 topic 的分区，那么就会有消费者消费不到数据”这句话是否正确？

7.消费者提交消费位移时提交的是当前消费到的最新消息的 offset 还是 offset+1？

offset+1

表示下一条待消费数据



8.有哪些情形会造成重复消费？

先处理再提交offset



9.那些情景会造成消息漏消费？  

先提交offset后处理



10.当你使用 kafka-topics.sh 创建（删除）了一个 topic 之后， Kafka 背后会执行什么逻辑？
1）会在 zookeeper 中的/brokers/topics 节点下创建一个新的 topic 节点，如：/brokers/topics/first

2）触发 Controller 的监听程序
3） kafka Controller 负责 topic 的创建工作，并更新 metadata cache



11.topic 的分区数可不可以增加？如果可以怎么增加？如果不可以，那又是为什么？



12.topic 的分区数可不可以减少？如果可以怎么减少？如果不可以，那又是为什么？

不可减少。



13.Kafka 有内部的 topic 吗？如果有是什么？有什么所用？

0.9之后，__consumer_offsets，给普通消费者存offset



14.Kafka 分区分配的概念？

Range（按主题）、RandRobin（按消费者组）



15.简述 Kafka 的日志目录结构？

index、log



16.如果我指定了一个 offset， Kafka Controller 怎么查找到对应的消息？

在Index二分查找法，找到对应的Log文件，再在对应log文件找到具体记录



17.聊一聊 Kafka Controller 的作用？

与zk集群打交道，kafka集群中临时选取了一个老大，干活。元数据更新之类的



18.Kafka 中有那些地方需要选举？这些地方的选举策略又有哪些？

Controller、Leader

Controller：抢资源

Leader：ISR，同步时间，同步条数（0.9干掉）



19.失效副本是指什么？有那些应对措施？





20.Kafka 的哪些设计让它有如此高的性能？  

分布式，分区

顺序写磁盘、零拷贝、页缓存