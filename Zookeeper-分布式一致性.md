## Zookeeper

zookeeper 底层提供两个功能： 1、管理用户提交的数据； 2、为用户提交数据节点监听服务



#### 会话

Session指Zookeeper服务器与客户端的会话。**一个客户端连接是指服务器与客户端之间的一个TCP长连接**

通过该连接，客户端可以：

- 通过心跳检测与服务器保持有效的会话
- 向Zookeeper发送请求并接收相应
- 接收来自服务器的Watch事件通知

断联时只要在sessionTimeout前连接上任意一台机器，之前的会话仍然有效

SessionID 全局唯一

















## 分布式一致性原理与实践

### 一、分布式架构

**分布式特点**：

​	分布式系统是一个硬件或软件组件分布在不同的网络计算机上，彼此之间仅仅通过消息传递进行通信和协调的系统

​	**分布性**：多台计算机在空间上随意分布，分布情况随时变动

​	**对等性:**计算机没有主从之分，所有节点都是对等的，提供数据、服务的冗余

​	并发性：

​	缺乏全局时钟

​	故障总会发生：组成分布式系统的所有计算机，都有可能发生各种形式的故障



**CAP理论**

一个分布式系统不可能同时满足一致性，可用性和分区容错性

一致性：数据的多个副本之间保持一致的特性

可用性：服务一直处于可用的状态，每一个操作请求总是在有限时间返回

分区容错性：遇到网络分区故障的时候，仍然能够保证对外提供一致性和可用性服务



**BASE理论**

基本可用 软状态 最终一致性

基本可用：响应时间增加 功能损失

软状态：数据同步的过程存在延时

最终一致性：系统中所有的数据副本，经过一段时间同步，最终能达到一个一致性状态



### 二、Paxos算法

两阶段提交协议：事务的提交过程分为两个阶段来进行处理

阶段一：提交事务请求 阶段二：执行事务提交



拜占庭问题：军队之间是分隔开来的，需要进行统一行动计划，但是所有通信员中可能存在叛徒，可以篡改信息。

（分布式系统内，异步系统和不可靠信道中达到一致性是不可能的），但是实际上可以保证消息的完整性，也可以基于校验算法来判断消息的真实性



问题描述：一组可以提出提案的进程集合，那么对于一致性算法来说：

1、这些提案中，只有一个被选定 2、如果没有提案选出，就不会选定 3、当一个提案被选定，进程能够获得选定信息

问题抽象：多个Acceptor，多个Proposer，多个Learner

P1: 一个Acceptor必须批准它收到的第一个提案

P2C：对于任意的Mn和Vn，如果[Mn,Vn]被提出，那么存在一个超过半数的Acceptor子集，满足下列中一个条件：

​	1、S中不存在任何批准过编号小于Mn的Acceptor（此时Vn取任意值）

​	2、S中所有Acceptor批准的编号小于Mn提案，其中编号最大值为Vn



生成提案过程：

1、Proposer选择一个新的编号Mn，向某个Acceptor集合成员发送请求，Acceptor做出以下回应

​	不再批准编号小于Mn的提案， 如果批准过，则反馈最大编号的value

2、如果收到了超过半数acceptor请求，则可以制定提案



批准提案：

任意时候相应一个Prepare请求，不违背现有承诺条件下，相应accept请求





### 四、zookeeper基本概念及ZAB协议

什么是**Zookeeper**

​	==一个分布式数据一致性的解决方案==，能够保证如下的分布式一致性

​		1、顺序一致性：同一个客户端发送的请求，能够按照发起顺序应用到Zookeeper中

​		2、原子性：一个事务在集群上的应用情况是一致的

​		3、单一视图：不管哪个服务器提供的视图是一致的

​		4、可靠性：成功应用某一事务，服务器的状态变更会保留下来

​		5、实时性：一定时间内，可以读到最新的数据状态



**ZooKeeper基本概念**

集群角色

​	Leader、Follower、Observer： Leader提供读写服务，Follower与Observer提供读服务

会话

​	客户端与服务端的一个TCP长连接。这个链接会执行心跳检测，发送并回应，watcher事件通知

数据节点

​	树型的数据节点，一个ZNode包含数据与属性信息，分为临时节点与永久节点

版本

​	version cversion aversion

Watcher

​	允许用户在指定节点上注册watcher



**ZAB协议**（ZooKeeper原子广播协议）

**核心**

​	主备架构，单一主进程接收并处理所有的事务请求，将服务器数据的状态变更同步到所有的副本上；需要保证事务的顺序执行；需要保证集群的高可用（Leader挂掉）

​	核心是定义那些改变服务器状态的事务请求的处理方式：用全局唯一的一个服务器来接收请求，服务器将这个请求转换成Proposal广播，然后过半即提交

​	分为 崩溃恢复+消息广播

**消息广播**

​	类似于两阶段的提交，proposal提出（这里所有的follower要么正常反馈，要么就抛弃leader），超过半数的follower回应ack后即发出commit信息。

​	整个消息广播过程中，Leader服务器为每一个事务分配一个全局递增的唯一ID，follower按照ID来对事务进行排序和处理，Leader为每一个Follower建立一个FIFO的队列，follower先将事务写入磁盘日志，并返回一个ack。

**崩溃恢复**

​	由于网络故障（Leader与超过半数的follower失去联系）或者是Leader挂掉的话，会进行崩溃恢复

​	两个不一致隐患：

​	1、一个事务已经在Leader上提交，但是广播commit前，挂了

​	2、丢弃只在Leader上被提出的事务

​	方法：选举出的新Leader拥有集群中最大的ZXID  



### 六、Zookeeper应用场景

**数据发布\订阅**

​	推、拉模式，定时轮询拉，Zookeeper采用推拉相结合的方式。通常， 应用启动时会到zookeeper服务端进行一次拉取，并配置一个watcher监听

​	**数据库切换：**

​		1、初始化配置存储到Zookeeper上的一个数据节点上

​		2、集群中每台机器在初始化阶段，会从上面的配置节点拉取数据库信息，并注册一个watcher监听

​		3、对配置节点进行更新，Zookeeper推送到每一个订阅的客户端

**负载均衡（Zookeeper在软负载均衡中的应用）：**

​	分布式系统具有对等性，且数据、服务具有冗余，需要在对等的提供方中选取一个来执行相应的业务逻辑

​	**动态DNS服务：**

​		1、域名配置：创建一个节点来进行域名配置，每个应用都可以创建一个属于自己的数据节点来作为域名配置的根节点

​		2、域名解析：域名解析是由每一个应用自己负责，首先从域名节点中获取一份IP地址和端口的配置，进行自行解析，同时注册一个Watcher监听

​		3、域名变更：对域名节点进行更新操作

**命名服务**

​	ZooKeeper提供命名服务帮助应用系统通过一个资源引用的方式来实现对资源的定位和使用

​	1、调用create()接口来创建一个顺序节点，例如`job-`，此基础上，拼接type类型

**分布式协调/通知**

​	注册watcher机制

**集群管理**

​	集群监控：对集群的工作状态进行收集

​	集群管理：对集群进行操作与控制

​	**分布式日志收集系统**

​		日志系统会把所有需要收集的日志机器分为多个组别，每个组别对应一个收集器

​			每个组别中的日志源机器通常是在不断变化的，日志收集器系统也会出现变更或者扩容，因此，如何快速，合理地为每个收集器分配对应的日志源机器

​			1、注册收集器节点，`logs/collectors/[Hostname]`

​			2、系统根据收集器节点数将日志源机器进行分组，并将相应的列表写到对应的子节点上

​			3、收集器创建一个状态子节点，`/logs/collectors/[Hostname]/status`，定期写入状态信息，日志系统根据最后更新时间判断收集器状态

​			4、**动态分配：**收集器挂掉或者扩容时，进行动态分配

​					全局动态分配：对所有日志源系统重新分组

​					局部动态分配：收集器在写入状态信息的时候，同时写入负载，分配时考虑负载较高的收集器

​		注意事项：

​			1、节点类型：持久节点+status状态

​			2、日志系统节点监听：日志系统主动轮询收集器节点

​	**在线云主机管理**

​		传统通过定时检测（如主机某个端口）来判断主机存货

**MASTER选举**

​	如果只想实现master选举，只需要有一个能够保证数据唯一性的组件就可以了，如果是动态master选举，可以基于zookeeper的watcher机制（Zookeeper在写入节点的时候强一致性）

**分布式锁**

​	通过一个数据节点来表示一个锁，用create()来创建一个节点，ZooKeeper会保证只有一个进程创建成功，然后没有创建成功的进程会注册Watcher监听





**Kafka中Zookeeper**：



​	**Broker注册：**

​		Kafka是一个分布式的消息系统，Broker、Producer、Consumer也是分开部署的，Zookeeper将所有的Broker服务器管理起来，Broker的节点路径`broker/ids/[0...N]`，创建完节点后，Broker将自己的ip和端口写入。这个节点是一个**临时节点**，下线或宕机后节点消失

​	**Topic注册：**

​		Kafka中会将同一个Topic分成多个分区并将其分布到多个Broker上，分区消息以及与Broker的对应关系也是由Zookeeper进行维护的，节点路径为`broker/topics`，例如`broker/topics/login`，Broker服务器会到相应的Topic节点下注册自己的brokerID，并写入分区的总数



​	**生产者负载均衡：**

​		生产者需要将消息合理地发送到分布式的Broker上

​		**kafka传统的四层负载均衡：**根据生产者的IP地址和端口为其确定唯一的关联Broker

​		**使用Zookeeper进行负载均衡**：当一个Broker启动时，会首先完成注册过程。生产者可以通过这个节点来动态感知到Broker服务器列表的变更。实现上，生产者订阅一些"Broker新增与减少""Topic新增与减少"''Broker与Topic的关联关系''



​	**消费者的负载均衡:**

​		每个消费者分组内有若干个消费者，每一条消息只会发给一个消费者，因此消费者的负载均衡是同一个消费者内部的消息消费策略

​		**消费分区与消费者关系：**对于每一个消费者分组，Zookeeper会分配一个GroupID，组内所有消费者共享，同时每个消费者也有一个ConsumerID，采用“Hostname:UUID”形式表示。每个消息分区有且只能同时有一个消费者进行消息消费，因此消费者需要写入消息分区的一个临时节点：`consumers/[group_id]/owners/[topic]/[broker_id-partition_id]`，写入的内容就是消费者的ConsumerID

​		**消费进度Offset记录**：需要定时将Offset记录到Zookeeper上去，Zookeeper上有一个专门记录offset的节点



​	**消费者注册：**

​		1、注册到消费者分组，到Zookeeper上创建一个属于自己的消费者节点`consumers/[group_id]/ids/consumer_id`，完成创建后，将自己订阅的Topic写入节点中，该节点也为**临时节点**

​		2、对消费者ids子节点进行监听，消费者新增或减少会触发负载均衡策略

​		3、对Broker服务器变化注册监听

​		4、进行消费者的负载均衡：为了能让同一个Topic下不同分区的消息尽量均衡地被多个消费者消费而进行的一次消费者与消费分区分配的过程

​	

​	***负载均衡：***官方文档



**Dubbo与Zookeeper：**

​	Zookeeper可以作为Dubbo的服务注册中心。Dubbo注册中心整体架构：

​	1、/dubbo:Dubbo在Zookeeper上的根节点

​	2、/dubbo/com.foo.BarService：服务节点，代表一个dubbo服务

​	3、/dubbo/com.foo.BarService/providers：服务提供者的根节点，子节点代表每一个服务的真正提供者

​	4、/dubbo/com.foo.BarService/consumers：服务消费者的根节点，子节点代表每一个服务的真正消费者

​	

​	**注册中心的一次工作流程**（Barservice）

​	服务提供者：启动时，首先在`/dubbo/com.foo.BarService/providers`下创建一个子节点，并写入自己的URL地址

​	服务消费者：启动时读取并订阅`/dubbo/com.foo.BarService/providers`，并解析url地址作为服务的地址列表

​	监控中心：监控中心会获取提供者和消费者所有的URL





### 七、Zookeeper技术内幕

**系统模型**



​	**数据模型：**ZNode是Zookeeper中最小的数据单元，每个ZNode可以保存数据，也可以挂载子节点

​	**事务ID:** ZooKeeper中事务指能改变服务器状态的操作，对于每一个事务请求，ZooKeeper分配一个全局唯一的ID，用ZXID表示



​	**节点类型**：**持久节点、临时节点、顺序节点**，可以组合，对于顺序节点，每个父节点会为其第一级子节点维护一份顺序（创建先后）；临时节点的生命周期与客户端会话绑定在一起，客户端失效，节点会被清理掉。

​		状态信息：每一个数据节点除了存储数据内容以外，还存储节点本身的一些状态信息，Stat类存储

![1564367920719](C:\Users\bingmeishi\AppData\Roaming\Typora\typora-user-images\1564367920719.png)

​	

**版本--保证分布式数据原子性**

​		Zookeeper为每个数据节点引入了版本信息，包含三个版本，**这里版本是指对数据节点的修改次数**

| 版本类型 | 说明           |
| -------- | -------------- |
| version  | 数据内容版本号 |
| cversion | 子节点版本号   |
| aversion | ACL变更版本号  |

​		Zookeeper中版本表示对数据内容，子节点列表，ACL信息的修改次数

​		Zookeeper中版本实际上时对乐观锁中写入校验的实现

版本与分布式锁：

​		乐观锁三个阶段：读取数据、写入校验、数据写入

​		版本是实现**写入校验**这一步的，在写入数据时，会比较当前请求的版本version（应该是读取数据时一起读取的，`int version= setDataRequest.getVersion`），与当前数据节点的version进行对比

​		

**Watcher--数据变更通知**

​		Zookeeper提供分布式的发布/订阅功能，一个典型的发布/订阅模型定义了一种一对多的订阅关系，Zookeeper通过watcher来实现多订阅者订阅一个主题

​		Watcher机制主要包括：**客户端线程、客户端WatcherManager、Zookeeper服务器**

​		客户端注册Watcher时，会将Watcher对象存储在WatcherManager中，Zookeeper服务器触发Watcher事件后，会发送通知，客户端取出对应的Watcher对象执行回调逻辑

​		**Watcher接口：**

​			定义事件通知的相关逻辑，包含KeeperState和EventType两个枚举类，表示通知状态和事件类型，同时定义了一个事件的回调方法，`process(WatchedEvent event)`

​		**Watcher事件中封装了通知状态，事件类型以及path**

​		NodeDataChanged：数据内容或版本号发生改变时触发（用同一数据更新节点）

​		NodeChildrenChanged：子节点列表发生变更时触发

​		WatchedEvent 包含三个属性：通知状态，事件类型，节点路径

​		服务端会将WatchedEvent封装成一个可序列化的WatcherEvent，用于网络传输



**工作机制**

​		分为三步：客户端注册Watcher，服务端处理watcher，客户端回调watcher

​		**客户端注册watcher：**

​			可以在客户端对象的构造函数中传入默认watcher，也可以通过getData等接口注册watcher

​			在利用getdata等方法注册Watcher时，客户端会封装一个WatcheRegistration对象（保存数据节点路径与Watcher的对应关系），WatcheRegistration会被封装到Packet对象中（这个对象是最小通信单元），然后被发送

​			客户端SendThread线程的readResponse对象会接收响应，并且取出Watcher（还是在WatcherRegistration中）注册到ZKWatcherManager中，最终保存在dataWatches中

```java
dataWatches: 映射数据节点和Watcher对象
Map<String, Set<Watcher>>
```

​			底层序列化过程，并没有将Watcher对象完全封进去（否则注册的watcher一多，服务端内存压力就会很大）



​	

​		**服务端处理watcher：**

​			**ServerCnxn存储（将对应的ServerCnxn存储到WatcherManager中）**

​				判断需要存储Watcher时，将当前的ServerCnxn对象（可以理解为一个Watcher对象，实现了watcher接口）以及数据节点路径传入到getData方法中，最终被存到WatcherManager的watchTable和watch2Path中。watchTable	从数据节点路径的粒度来管理watcher，watch2Path从Watcher粒度来控制事件触发需要触发的数据节点

​			**Watcher触发**

​				对指定的数据节点完成更新后，调用WatchManager的TriggerWatch方法来触发相关的事件

​				处理逻辑：1、封装WatchedEvent	2、查询Watcher	3、调用process来触发Watcher（这里是ServerCnxn的process方法，这里process的主要逻辑为：1、请求头里标志-1，代表是一个通知；WatchedEvent包装为WatcherEvent，以便序列化传输，向客户端发送通知）



​		**客户端回调Watcher**

​				对于来自服务端的响应，客户端都调用SendThread中的readResponse来处理

​				当一个通知事件来临时，典型的处理逻辑为：1、反序列化	2、处理chrootPath（绝对路径转为相对路径）	3、还原watchedEvent	4、回调Watcher（将WatchedEvent对象交给EventThread线程）

​				**EventThread处理事件通知：**

​					客户端识别EventType后，从相应的ZKWatchManager中获取相关的watcher（放到WaitingEvents中），并从Watcher存储中去除，将所有的watcher放到waitingEvents队列中

​					EventThread的run方法不断对该队列进行处理（每次取出一个watcher，并调用其process方法）



​		**Watcher特性总结：**

​			**一次性**：Watcher需要反复注册，用以减少服务端的压力（因为一次注册一直有效的话，更新频繁的节点对服务器压力较大）

​			**客户端串行执行：**客户端处理watcher是一个串行过程

​			**轻量：**WatchedEvent是Zookeeper中最小通知单元，只包含事件类型，通知状态和节点路径，传输的时候也不会将发送对象传送至服务端，只是通过**boolean对象**进行标记（服务端调用boolean getWatch()判断是否需要注册watcher）





**ACL机制（Access Control List）**

​	访问控制列表，细粒度的权限管理

​	权限模式（Scheme）授权对象（ID）权限（Permission），使用scheme id permission 来标志一个acl信息

​		**权限模式（Scheme）**

​			确定权限验证过程中使用的策略：1、IP	2、Digest	3、World	4、Super

​		**授予对象（ID）**

​			权限赋予的用户或是一个指定实体，例如IP或是机器

​		**权限（Permission）：**

​			对数据的操作权限分为五类：1、CREATE（允许在该节点下创建子节点）	2 DELETE（允许删除该数据节点的子节点）	3 READ	4 WRITE	5 ADMIN（允许对数据节点进行ACL相关的设置操作）



**客户端**

​	客户端由以下核心组件组成：

​		1、Zookeeper实例	2、ClientWatchManager	3、HostProvider	4、ClientCnxn（包含SendThread EventThread）

​	客户端实例化与启动过程主要可以分为三个步骤：

​		1、设置默认的Watcher	2、设置Zookeeper服务器地址列表	3、创建ClientCnxn



**会话**

​		客户端会话可能有多个状态，CONNECTING CONNECTED RECONNECTING RECONNECTED CLOSE等

​		Session是ZooKeeper中的会话实体，代表一个客户端会话，包含四个属性

​		1、SessionID：会话ID，唯一标志一个会话

​		2、TimeOut：会话超时时间

​		3、TickTime：

​		4、isClosing：一个会话是否关闭，一旦超时，服务端设为true，就不在接受该客户端请求

​		

​		SessionTracker 服务端的会话管理器 ，管理会话