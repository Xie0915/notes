# Java并发编程

### 2、线程安全性

​	对状态访问操作进行管理，包括共享以及可变状态；同步操作包括sychronized（内置锁）显示锁 原子操作 不可变对象

​	封装性越好，越容易实现程序的线程安全性



**什么是线程安全性**

​	正确性：某个类的行为与其规范完全一致。良好的规范通常会定义不变性状态来约束对象的状态，以及后验条件来描述对象操作的结果

​	线程安全性：多个线程访问同一个类时，这个类始终能够表现出正确的行为，这个类就是线程安全的

​	**无状态的对象一定是线程安全的**

**原子性**

​	**竞态条件**

​		某个计算的正确性取决于多个线程交替执行的时序，就会发生竞态条件，最简单的竞态条件就是check-then-act，或者读取-修改-写入操作

​	**复合操作**

​		上述先检查后执行以及读取-修改-写入都是复合操作，这类操作可以调用`java.util.concurrent.atomic`中的原子变量，例如`AtomicLong`这能表示对状态的访问操作都是原子的

​		在无状态的对象中添加**一个**状态时，如果这个状态完全由线程安全的对象来管理，这个类也是线程安全的

**加锁机制**

​	用多个线程安全的变量来表示对象的状态并不是完全的线程安全的，因为这些对象的状态并不是完全独立的，例如如果有两个状态变量需要同时进行更新（用AtomicRefference来表示缓存）

​	**因此，要保持状态的一致性，就需要在单个原子操作中更新所有相关的状态变量**

​	**内置锁**

​		内置锁相当于一种互斥锁，获取锁的唯一方法是进入同步代码块

​	**重入**

​		设置一个计数值以及一个所有者线程，请求一个未被持有的锁，JVM将记下锁的持有者，并将计数值+1，当同一个线程重复获得锁时，计数值递增，计数值为0时，锁释放。这也表明获取锁操作的粒度为“线程”，而不是调用

​	**活跃性与性能**

​		不应将同步代码块分得过细，因为获取和释放锁都需要一定的开销。

​		通常，简单性与性能之间会相互制约，但盲目地牺牲简单性可能会带来安全性上的问题



### 3、对象的共享

内存可见性：当一个线程修改了对象的状态时，其他线程能够看到发生的状态变化





**可见性**

​	在没有同步的情况下，编译器、处理器以及运行时都可能对执行次序进行一定地调整

**失效数据**

​	除非进行同步，否则可能获得失效值；并且，有的线程可能获得最新值，有的可能获得失效值

​	get set 方法需要同时进行同步

**非原子的64位操作**

​	没有同步情况下的失效值是之前某个线程设置的值，这称作**最低安全性**

​	但是对于非volatile类型的64位数值变量，jvm允许将其分为两个32位操作

**加锁与可见性**

​	加锁可以保证一个线程以一种可预见的形式查看另一线程的执行结果

​	因此，加锁并不仅仅是保证互斥，还包括内存可见性，为了确保所有线程都能看到共享变量的最新值，所有执行的写操作与读操作都必须在同一个锁上同步

**Volatile**

​	编译器与运行时会注意到这个变量是共享的，因此不会将该变量上的操作与其他操作进行重排序

​	**访问volatile不会加锁，也不会使线程阻塞**，是一种轻量级的同步机制

​	volatile变量通常用作某个操作完成、发生中断、或者状态的标志；但是其语义并不足以保证count++的原子性





**发布与逸出**

​	发布一个对象：使对象能够在当前作用域之外的代码中使用

​	逸出：某个不应该被发布的对象被发布

​	发布对象最简单的方法是将对象的引用保存到一个公有的静态变量中

​	发布某一个对象时，有可能间接发布其他的对象，例如集合中的对象

​	对一个类来说，外部方法指行为不按C规定的方法：包括其他类中定义的方法以及C中可以被改写的方法。无论其他类是否对发布的对象执行操作都没有影响，因为误引用可能性一直存在。

**安全的对象构造过程**

​	当且仅当构造函数返回时，对象才处于可预测以及一致状态

​	**不要在构造函数内让this逸出**

​	this逸出一个常见的错误是，在构造函数里启动一个新的线程，this引用会被新线程所共享，对象尚未构造完成时，新线程就能看见

​	如果想在构造函数内部注册一个事件监听装置，可以使用私有的构造函数和公有的工厂方法





**线程封闭**

​	避免使用同步的方法是不共享数据，线程封闭是指仅在单线程内访问数据

​	JDBC的connection使用了线程封闭的技术

**ad-hoc封闭**

​	维持线程封闭的职责完全由程序来承担

**栈封闭**

​	只能通过局部变量才能访问对象

​	对于基本类型的局部变量，任何方法都无法获得对基本类型的引用，因此不会破坏栈封闭性

​	对于引用对象的局部变量，需要确保该对象不会逸出

**ThreadLocal类**

​	使线程中的某个值与保存值的对象关联起来，提供get、set访问接口，每个使用该变量的线程都存有一个副本

**不变性**

​	如果一个对象在创建之后状态就不能被修改，这个对象就称为不可变对象，线程安全性是不可变对象的固有属性。不可变对象只有一种状态，该状态由构造函数来确定

​	不可变对象满足以下条件：

​		1、对象创建之后其状态就不能修改

​		2、对象的所有域都是final

​		3、创建期间this指针未逸出

​	可以使用包含多个状态变量的容器对象（不可变）来维持不变性条件，并使用一个volatile类型引用来确保可见性



**安全发布**

**安全发布的常用模式**

​	安全发布一个对象，对象的引用以及状态必须同时对其他线程可见

​	

​	

​	

### 5、并发容器类



**并发容器**

**ConcurrentHashMap**

​    用一种粒度更细的加锁机制来实现更大程度的共享（分段锁），任意数量的读线程可以并发访问Map，执行读取与执行写入的线程可以并发访问容器，一定数量的写线程可以并发修改Map，并发访问下实现更高的吞吐量，单线程中损失一小部分的性能

​	不会抛出`ConcurrentModificationException`因此不需要对容器进行加锁，ConcurrentHashMap返回的迭代器具有**弱一致性**，迭代器可以容忍并发修改，创建时会遍历所有的元素，在迭代器构造后将修改的操作返回给容器

​	ConcurrentHashMap中提供了一些原子操作（因为不能加锁）

```java
V putIfAbsent(K key, V value);
boolean remove(K key, V value);
boolean replace(K key, V oldValue, V newValue);
V replace(K key, V newValue)
```



**CopyOnWriteArrayList**

​	用于替代List，在同步过程中不需要对容器进行加锁。

​	对其同步时需要保证底层数组的可见性，每次修改时，都会复制底层数组。





**阻塞队列**

​	提供阻塞的put和take方法，以及支持定时的offer和poll方法

​	如果数据项不能被正确加入到队列，offer将会返回一个失败的状态，可以根据这个调整策略

​	类库中有很多BlockingQueue的实现：LinkedBlockingQueue和ArrayBlockingQueue是FIFO队列，分别于LinkedList于ArrayList类似，但是比同步的List拥有更好的并发性能。PriorityBlockingQueue是按优先级排序的队列，还有一种实现是SynchronousQueue，这不是一个队列，它不会为队列中的元素维护存储空间，它维护一组线程。





**阻塞方法与中断方法**

​	三种状态：`BLOCKED` `WAITING` `TIMED_WAITING`

​	阻塞操作必须等待某个不受它控制的事件发生后才能继续执行，如等待IO 等待锁等，当外部事件发生后，重回`Runnable`状态

​	如果一个方法抛出`InterruptException`,表明该方法是一个阻塞方法，如果调用了一个将抛出`InterruptException`方法，该方法也成为了一个阻塞方法，需要处理这个异常

​	1、传递`InterruptException`:不做任何处理，抛出；或者简单处理，抛出

​	2、恢复中断：捕获异常，调用`Interrupt`方法

​	Thread提供了`Interrupt`方法，用于中断线程或者是查询线程是否中断



**同步工具类**：

​	根据自身状态来协调线程的控制流，如阻塞队列，信号量，栅栏，闭锁等

​	特点：

​		1、封装了一些状态    2、提供了一些方法对状态进行操作   3、提供一些方法用于高效等待同步工具类

​	**闭锁：**

​		闭锁到达结束状态之前，所有线程都阻塞，下述应用

​	        确保计算在所有资源都加载完成后才进行

​			确保某个服务在其依赖的所有服务启动后启动

​			等待某个操作所有参与者都就绪

​		`CountDownLatch`包括一个计数器，初始化为一个正数，表示需要等待的事件数量，`countDown`方法递减计数器，`await`方法阻塞至计数器变为零



### 6、任务执行

**线程中执行任务**

​	**无限制创建新线程**

​		不足：

​			1、线程生命周期开销非常高，创建需要时间，且取药JVM与操作系统一些辅助操作

​			2、资源消耗：消耗系统资源，尤其是内存，如果运行线程数大于处理器数量，会闲置

​			3、稳定性：可创建线程的数量随着平台的不同而不同

**Executor框架**

​	java类库中，任务执行的主要抽象不是Thread，而是Executor

```java
public interface Executor{
    void execute(Runnable command);
}

//同步进行
pulbic execute(Runnable r){
    new Thread(r).start();
}

//串行进行
public execute(Runnable r){
    new Thread(r).run();
}
```

​	基于生产者、消费者模式，提供任务的为生产者，执行任务的线程为消费者

​	可以设置执行策略，包括执行顺序，并发数，过载时操作，执行前后操作等

​	**线程池**

​		工作者线程（Worker Thread）：从工作队列（Work Queue）中获得一个任务，执行任务，然后返回线程池

​		优势：1、分摊线程创建和销毁的开销  2、减少创建线程的延迟

```java
//Executor中静态工厂方法创建线程池

newFixedThreadPool;
newCachedThreadPool;
...
```

​	**Executor 生命周期**

​		ExecutorService 一共有三种运行状态：运行、关闭、已终止；关闭后提交的任务将由拒绝执行处理器处理，它将抛弃任务

```java
pulbic interface ExecutorService {
    void shutdown();  //平稳关闭
    List<Runnable> shutdownNow();  //粗暴关闭
    boolean isShutdown();
    boolean isTerminated();
    boolean awaitTermination(long timeout, TimeUnit unit);
}
```

​		

**Callable与Future**

​	Runnable不能返回一个值或者说抛出一个异常

​	Future表示一个任务的生命周期，可以判断任务是否完成或者是取消，以及获取任务的结果或者取消任务

```java
public inteface Future<V>{
    boolean cancel(...);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException, CancelledException;
    V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, CancelledException, TimeoutException
}
```

​	get()方法行为取决于任务状态，任务完成则返回一个值或者是ExecutionException（该异常封装了原有异常）

​	在Runnable与Callable任务提交中包含一个安全发布的过程，提交->运行线程；运行线程->get的线程



**CompletionService：Executor和BlockingQueue**

​	将callable提交给`CompletionServic`e执行，并表用take和poll方法来获得已完成的结果

​	`ExecutorCompletionService`实现了`CompletionService`，在构造函数中创建一个`BlockingQueue`保存计算结果，计算完成调用`FutureTask`中的Done方法。提交一个任务时，首先将其包装成一个`QueueingFuture`（`FutureTask`的子类），并改写子类的done方法，将结果放在`BlockingQueue`中



**为任务设定时限**

```java
V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, CancelledException, TimeoutException
```

结果可用时，立即返回；指定时限中没有计算出结果，抛出`TimeoutException`

也可以由任务本身管理时限，超时时抛出TimeoutException，并由Future来取消任务

​	**invokeall方法：**

![1564713285745](C:\Users\bingmeishi\AppData\Roaming\Typora\typora-user-images\1564713285745.png)



### 8、线程池使用

​	









### 11、显式锁 Lock

**为什么显示锁**？

​	内置锁局限性：无法中断一个正在获取锁的线程；无法获取锁时无限等待下去；必须在获取该锁的代码中释放，因此无法实现非阻塞的加锁规则

**轮询锁与定时锁**

​	`trylock`方法获取锁，如果不能获取锁将释放所有的锁并尝试重新获得锁，会进行一段时间的休眠，包含固定部分和随机部分

​	如果在带有时间限制的操作中调用了一个阻塞方法，会根据剩余时间提供一个时限，操作不能在指定时间结束的话会提前终止程序。而内置锁则无法取消

**可中断的锁获取操作**

​	`lockInterruptibly`获得锁的同时保持对中断的相应

**非块结构的加锁**

​	降低锁的粒度，例如对一个链表中的节点加锁





**公平性**

- 公平的锁：按照请求的顺序来获得锁，维护一个队列
- 非公平锁：允许插队，如果发出请求的同时一个锁的状态变为可用，则跳过队列直接获得锁

大多数情况下，非公平锁的性能优于公平锁，有一个重要的原因：**恢复一个被挂起的线程与该线程真正运行之间存在严重的延迟**





**选择sychronized或是Reentrantlock**

在内置锁无法满足要求的时候使用Reentrantlock，包括：可定时、轮询、中断的锁获取操作，公平队列以及非块结构的锁，否则优先使用synchronized

Reentrantlock还能给出哪些调用帧获得了哪些锁，能够检测和识别石锁发生，但是不能和特定的栈帧联系起来







**读写锁**

`ReentrantReadWriteLock`

读取锁和写入锁只是读-写锁对象的不同视图