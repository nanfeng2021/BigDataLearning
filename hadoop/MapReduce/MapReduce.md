# MapReduce

分布式计算框架

优点：1、易于编程 2、良好扩展性 3、高容错性 4、适合PB级以上海量数据的离线处理

缺点：1、不擅长实时计算 2、不擅长流失计算 3、不擅长DAG计算



**WordCount**

![](E:\github\BigDataLearning\hadoop\MapReduce\MapReduce_WordCount.png)



### MapReduce框架原理

![](E:\github\BigDataLearning\hadoop\MapReduce\MapReduce_框架原理.png)

#### FileInputFormat数据输入

首先，需要解决的是，一个job会生成几个Task去执行？

##### 切片与MapTask并行度决定机制

MapTask的并行度决定Map阶段的任务处理并发度，进而影响到整个Job的处理速度。

思考：MapTask并行度是由什么决定的？

**数据块与数据切片**

数据块：Block是HDFS把数据一块一块存储在物理机器上。

数据切片：在逻辑上对输入进行切片，并不会在磁盘上将其切分成片进行存储。**数据切片是MapReduce程序计算输入数据的单位。**一个切片会对应启动一个MapTask。

![](E:\github\BigDataLearning\hadoop\MapReduce\数据切片与MapTask并行度.png)

##### Job提交流程





#### Map

#### Reduce

#### FileOutputFormat数据输出