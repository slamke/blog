学习资料为：http://ifeve.com/zookeeper-sharedcount/

# zookeeper的功能列表
ZooKeeper官方给出了使用zookeeper的几种用途。
* Leader Election
* Barriers
* Queues
* Locks
* Two-phased Commit
* 其它应用如Name Service, Configuration, Group Membership

在实际使用ZooKeeper开发中，我们最常用的是Apache Curator。 它由Netflix公司贡献给Apache。
Curator的主要组件为：
* Recipes， ZooKeeper的系列recipe实现, 基于 Curator Framework.
* Framework， 封装了大量ZooKeeper常用API操作，降低了使用难度, 基于Zookeeper增加了一些新特性，对ZooKeeper链接的管理，对链接丢失自动重新链接。
* Utilities，一些ZooKeeper操作的工具类包括ZK的集群测试工具路径生成等非常有用，在Curator-Client包下org.apache.curator.utils。
* Client，ZooKeeper的客户端API封装，替代官方 ZooKeeper class，解决了一些繁琐低级的处理，提供一些工具类。
* Errors，异常处理, 连接异常等
* Extensions，对curator-recipes的扩展实现，拆分为 curator-:stuck_out_tongue_closed_eyes:iscovery和 curator-:stuck_out_tongue_closed_eyes:iscovery-server提供基于RESTful的Recipes WEB服务.

## leader选举
* Leader latch：必须启动LeaderLatch: leaderLatch.start(); 一旦启动， LeaderLatch会和其它使用相同latch path的其它LeaderLatch交涉，然后随机的选择其中一个作为leader。 你可以随时查看一个给定的实例是否是leader。一旦不使用LeaderLatch了，必须调用close方法。 如果它是leader,会释放leadership， 其它的参与者将会选举一个leader。异常处理 LeaderLatch实例可以增加ConnectionStateListener来监听网络连接问题。 当 SUSPENDED 或 LOST 时, leader不再认为自己还是leader.当LOST 连接重连后 RECONNECTED,LeaderLatch会删除先前的ZNode然后重新创建一个. LeaderLatch用户必须考虑导致leadershi丢失的连接问题。 强烈推荐你使用ConnectionStateListener。
* Leader Election
	* LeaderSelector
	* LeaderSelectorListener
	* LeaderSelectorListenerAdapter
	* CancelLeadershipException

	与LeaderLatch， 通过LeaderSelectorListener可以对领导权进行控制， 在适当的时候释放领导权，这样每个节点都有可能获得领导权。 而LeaderLatch一根筋到死， 除非调用close方法，否则它不会释放领导权。
## 计算器
* SharedCount
* SharedCountReader
* SharedCountListener
支持监听，强制set，trySet，leguansuo
## 临时节点
 临时节点驻存在ZooKeeper中，当连接和session断掉时被删除。
 PersistentEphemeralNode   节点必须调用start方法启动。 不用时调用close方法。
## 分布式Barrier
 分布式Barrier是这样一个类： 它会阻塞所有节点上的等待进程，知道某一个被满足， 然后所有的节点继续进行。
 * DistributedBarrier
 * DistributedDoubleBarrier 双栅栏允许客户端在计算的开始和结束时同步。当足够的进程加入到双栅栏时，进程开始计算， 当计算完成时，离开栅栏。 类似于百米赛跑
 ---
 首先你需要设置栅栏，它将阻塞在它上面等待的线程:
```java
setBarrier();
```
然后需要阻塞的线程调用“方法等待放行条件:
```java
public void waitOnBarrier()
```
当条件满足时，移除栅栏，所有等待的线程将继续执行：
```java
removeBarrier();
```
异常处理 DistributedBarrier 会监控连接状态，当连接断掉时waitOnBarrier()方法会抛出异常。
## 队列
Curator也提供ZK Recipe的分布式队列实现。 利用ZK的 PERSISTENTSEQUENTIAL节点， 可以保证放入到队列中的项目是按照顺序排队的。 如果单一的消费者从队列中取数据， 那么它是先入先出的，这也是队列的特点。 如果你严格要求顺序，你就的使用单一的消费者，可以使用leader选举只让leader作为唯一的消费者。

但是， 根据Netflix的Curator作者所说， ZooKeeper真心不适合做Queue，或者说ZK没有实现一个好的Queue，详细内容可以看 Tech Note 4， 原因有五：
* ZK有1MB 的传输限制。 实践中ZNode必须相对较小，而队列包含成千上万的消息，非常的大。
* 如果有很多节点，ZK启动时相当的慢。 而使用queue会导致好多ZNode. 你需要显著增大 initLimit 和 syncLimit.
* ZNode很大的时候很难清理。Netflix不得不创建了一个专门的程序做这事。
* 当很大量的包含成千上万的子节点的ZNode时， ZK的性能变得不好
* ZK的数据库完全放在内存中。 大量的Queue意味着会占用很多的内存空间。

---
* DistributedQueue  队列是与path绑定的
	* QueueBuilder
	* QueueConsumer
	* QueueSerializer
	* DistributedQueue
* DistributedIdQueue   可以为队列中的每一个元素设置一个ID。 可以通过ID把队列中任意的元素移除
* DistributedPriorityQueue 优先级队列对队列中的元素按照优先级进行排序。 Priority越小， 元素月靠前， 越先被消费掉。 
* DistributedDelayQueue  元素有个delay值， 消费者隔一段时间才能收到元素。
* SimpleDistributedQueue  实现类似JDK一样的接口

## 缓存
利用ZooKeeper在集群的各个节点之间缓存数据。 每个节点都可以得到最新的缓存的数据。 Curator提供了三种类型的缓存方式：Path Cache,Node Cache 和Tree Cache。

* Path Cache
Path Cache用来监控一个ZNode的子节点. 当一个子节点增加， 更新，删除时， Path Cache会改变它的状态， 会包含最新的子节点， 子节点的数据和状态。 这也正如它的名字表示的那样， 那监控path。
* Node Cache
Path Cache用来监控一个ZNode. 当节点的数据修改或者删除时，Node Cache能更新它的状态包含最新的改变。
* Tree Node
这种类型的即可以监控节点的状态，还监控节点的子节点的状态， 类似上面两种cache的组合。 这也就是Tree的概念。 它监控整个树中节点的状态。 

##分布式锁
- 可重入锁Shared Reentrant Lock
首先我们先看一个全局可重入的锁。 Shared意味着锁是全局可见的， 客户端都可以请求锁。 Reentrant和JDK的ReentrantLock类似， 意味着同一个客户端在拥有锁的同时，可以多次获取，不会被阻塞。 
	* InterProcessMutex
- 不可重入锁Shared Lock
- 可重入读写锁Shared Reentrant Read Write Lock  一个拥有写锁的线程可重入读锁，但是读锁却不能进入写锁。 这也意味着写锁可以降级成读锁， 比如请求写锁 —>读锁 —->释放写锁。 从读锁升级成写锁是不成的。
- 信号量Shared Semaphore
上面说讲的锁都是公平锁(fair)。 总ZooKeeper的角度看， 每个客户端都按照请求的顺序获得锁。 相当公平。
- 多锁对象 Multi Shared Lock

Multi Shared Lock是一个锁的容器。 当调用acquire， 所有的锁都会被acquire，如果请求失败，所有的锁都会被release。 同样调用release时所有的锁都被release(失败被忽略)。 基本上，它就是组锁的代表，在它上面的请求释放操作都会传递给它包含的所有的锁。

#Curator
Framework
事务
