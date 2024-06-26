## 参考链接
https://blog.csdn.net/Javachichi/article/details/131871627

## 总结
```
1： Spark的宽窄依赖
       宽依赖： 一个Rdd的分区依赖父RDD的多个分区，必须等待父RDD执行完毕后才可以执行下一个RDD 分区之间无法并行操作 一般会触发shuffle
       窄依赖： 一个Rdd的分依赖于父RDD的一个分区，这样各个分区之间就可以并行执行

2： Spark的两种操作
     transformation ： 惰性求值，就是我们只有执行action的时候才会真正触发RDD   reducebykey它不是一个action操作 会对相同的key 规约但是
      返回的是一个新的RDD
     action  : 一般是stage之间action就是对数据进行处理会触发transformation的操作

3. Spark的两种Shuffle   shuffle 就是在transform 和 action 之间的执行  也就是在两个stage之间进行执行 action需要将相同的key进行处理比如reducebykey 那么我们肯定是要把相同的key放入同一个分区进行计算的这也就是shuffle的作用
    SortShuffleManager ：
    HashShuffleManager
    shufflewrite :我们将相同的key写入不同的磁盘文件中这个磁盘文件的数量是根据下游task数决定的一个上游的task要为每一个下游的task创建一个磁盘文件
    shufflereader: 我们下游的task需要去拉取所有属于自己的文件，这个过程是通过buffer缓冲区实现的

4： Spark如何构建DAG
Spark Context 构建有向无环图
   1: 通过回溯算法，当遇到了宽依赖的时候就将这两个依赖分别置于两个stage，也就是进行一次划分，当遇到窄依赖我们就放到同一个Stage中
   2： 我们构建DAG不是立刻就执行的，而是action操作触发的时候才去按照DAG,提前去优化好路线
   3： 效率的提升就是在窄依赖的并行操作去提升效率的

5： Spark的具体执行流程
SparkContext 向资源管理器注册并向资源管理器申请运行 Executor
资源管理器分配 Executor，然后资源管理器启动 Executor
Executor 发送心跳至资源管理器
SparkContext 构建 DAG 有向无环图
将 DAG 分解成 Stage（TaskSet）
把 Stage 发送给 TaskScheduler
Executor 向 SparkContext 申请 Task
TaskScheduler 将 Task 发送给 Executor 运行
同时 SparkContext 将应用程序代码发放给 Executor
Task 在 Executor 上运行，运行完毕释放所有资源

6: RDD
R 代表的就是弹性的意思就是可以存放在内存中和磁盘中
D 代表的是可分布式计算，就是可以并行计算
D 代表Dataset
1: 是一个可以容错且并行的数据结构（其实可以理解成分布式的集合，操作起来和操作本地集合一样简单)，它可以让用户显式的将中间结果数据集保存在 内存 中，
并且通过控制数据集的分区来达到数据存放处理最优化
2: RDD的前后依赖关系使得我们只要知道它的父RDD我们就可以获取即使丢失也没有关系，我们不用计算所有的分区只需计算对应依赖的分区
3：RDD的五个属性
分区列表、分区函数、最佳位置，这三个属性其实说的就是数据集在哪，在哪计算更合适，如何分区；
计算函数、依赖关系，这两个属性其实说的是数据集怎么来的。
4： 持久化  persist cache checkpoint
cahe调用了persist peiesit可以设置缓存的级别
 
7： SparkStraming
采用的是Dstream
就是微型批处理，我么让一个小的时间段的数据集当作一个RDD然后处理

8 :structedStreming 可以处理更加细微的流式数据 他是使用无界表的方式
遇到一个数据就可以将其加入无界表的尾部，然后进行实时sql的查询
-采用DataFrame

9: spark的优点
  1： 调度算法 基于DAG 的调度算法实现了spark
  2.  lineage 提供了容错机制
```

10: Spark为什么要作持久化
1： 由于linage系统的存在Spark其实需要在复杂步骤中进行persist暂存到磁盘中，如果丢失数据可以恢复
2： 具体来说 我们需要在一些复杂的操作前后及逆行persist,如shuffle操作要在前后进行pressit进行存储

11： join操作调优
通过广播连接的方式进行连接，主要是用于小表连接大表即mapper-join 通过将小表放入内存中广播到大表中去，与大表的每个部分进行连接
而如果是reduce-join shuffle join 的话将两个相同的键进行排序，然后将相同的键发送到reduce中进行处理，这种操作会有耗时

12 block 和 partition的区别
block 是 hdfs 上的最小分块单位 存储的视觉去看
partition 是RDD的组成部分 大小不一 从计算的视角去看

13： sortedshuffle 和 hashshuffle 的区别
sortedshuffle比较适合处理大型数据集
  sorted shuffle产生的磁盘文件的较少有合并的过程并且它传输到分区中的数据是有序的  方便某些任务的执行
hashshuffle 比较适合处理小型或中型数据集
  hashshuffle 会产生大量的小的磁盘空间因为上一个任务的每一个task都要为下一个任务的所有task进行创造一休哥小的磁盘文件
14 ： RDD 不可变性 分部性 弹性
