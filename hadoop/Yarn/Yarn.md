---
typora-root-url: ./
---

# Yarn

### Yarn基础架构

![](/yarn基础架构.png)

### Yarn工作机制

![](/yarn工作机制.png)

1）MR程序提交到客户端所在节点;

2）YarnRunner（集群环境下）向ResourceManager申请一个Application;

3）RM将该应用资源路径返回给YarnRunner;

4）该程序将运行所需资源（split文件、xml文件、jar包）提交到HDFS上；

5）资源提交完毕，再次申请运行mrAppMaster；

6）RM这次，将用户的请求初始化成一个Task，并提交到**调度队列**中等待；

7）等到，其中一个NodeManager领到Task任务；

8）该NodeManager创建容器Container，并产生MRAppmaster；

9）Container从HDFS上拷贝资源到本地；

10）MRAppmaster向RM申请运行MapTask资源；

11）RM将运行MapTask任务分配给另外的NodeManager，它们领到任务并创建容器；

12）MRAppmaster向接收到任务的NodeManager发送启动脚本，它们分别启动MapTask，MapTask对数据分区排序；

13）MRAppmaster等待所有的MapTask运行完毕后，向RM申请容器，运行ReduceTask。

14）ReduceTask从MapTask获取相应分区的数据。

15）程序运行完毕后，MR会向RM申请注销自己。



### Yarn调度器

FIFO、容量（CapacityScheduler）、公平（FairScheduler）

Apache Hadoop 3.x默认Capacity Scheduler。

#### 先进先出调度器（FIFO）

![](/FIFO.png)

#### 容量调度器（Capacity Scheduler）

![](/Capacity Scheduler.png)

容量保证：每个队列可设置资源最低保证和使用上限

灵活性：一个队列中有空闲资源，可暂时共享给其它需要资源的队列。而一旦该队列有新的应用程序提交，则其它队列借调的资源会归还。



![](/Capacity Scheduler 调度算法.png)

##### 资源分配算法

###### 1）队列资源分配

从root开始，使用深度优先算法，**优先选择资源占用率最低**的队列分配资源。

###### 2）作业资源分配

默认按照提交**作业的优先级**和**提交时间顺序**分配资源。

###### 3）容器资源分配

按照容器的优先级分配。

若优先级相同，按照数据本地性原则：

（1）任务和数据在同一节点

（2）任务和数据在同一机架

（3）任务和数据不同节点、不同机架



#### 公平调度器（Fair Scheduler）

![](/Fair Scheduler.png)

与容量调度器的区别：

1）公平调度器——优先选择对资源的缺额比例大的；容量调度器——优先选择资源利用率低的队列

2）每个队列可以单独设置资源分配方式：

​		公平：FIFO、FAIR、DRF

​		容量：FIFO、DRF；

公平调度器设计目标：在时间尺度上，所有作业获得公平的资源。某一时刻一个作业应获取资源与实际获取资源的差距叫 “**缺额**”。因此，优先为缺额大的作业分配资源。

##### 资源分配算法

###### 1）队列资源分配

需求：集群总资源100，有三个队列，对资源的需求：queueA->20，queueB->50，queueC->30

|                                | queueA  | queueB  | queueC |
| ------------------------------ | ------- | ------- | ------ |
| 资源需求                       | 20      | 50      | 30     |
| 第1次                          | 33.33   | 33.33   | 33.33  |
|                                | 多13.33 | 少16.67 | 多3.33 |
| 第2次（多出的资源/少资源队列） | 20      | 16.66   | 30     |
| 最终                           | 20      | 50      | 30     |



###### 2）作业资源分配

（a）不加权（关注Job个数）

需求：有一条队列上总资源12，有4个Job，对资源的需求：job1->1,job->2,job3->6,job4->5

|                                   | Job1 | Job2 | Job3 | Job4 |
| --------------------------------- | ---- | ---- | ---- | ---- |
| 资源需求                          | 1    | 2    | 6    | 5    |
| 第1次                             | 3    | 3    | 3    | 3    |
|                                   | 多2  | 多1  | 少3  | 少2  |
| 第2次（多出资源数 / 少资源job数） | 1    | 2    | +1.5 | +1.5 |
| 最终                              | 1    | 2    | 4.5  | 4.5  |



（b）加权（关注Job的权重）

需求：有一条队列，总资源16，有4个job，

对资源需求：job1->4,job2->2,job->10,job->4

权重：job1->5,job->8,job3->1,job4->2

|                            | Job1 | Job2 | Job3     | Job4   |
| -------------------------- | ---- | ---- | -------- | ------ |
| 资源需求                   | 4    | 2    | 10       | 4      |
| 权重                       | 5    | 8    | 1        | 2      |
| 第1次（按权重分）          | 5    | 8    | 1        | 2      |
|                            | 多1  | 多6  | 少9      | 少2    |
| 第2次（多出资源/少的权重） | 4    | 2    | +2.33    | +4.66  |
|                            |      |      | 少6.67   | 多2.66 |
| 第3次                      | 4    | 2    | +2.66    | 4      |
|                            | 4    | 2    | 少4.01   | 4      |
| 最终                       | 4    | 2    | 6（少4） | 4      |

一直到没有空闲资源



###### 3）DRF策略

DRF（Domainant Resource Fairness）

对不同应用进行不同资源(CPU和内存)的一个不同比例的限制。