## 原子性对比

- synchronize：不可中断锁，适合竞争不激烈，可读性好
- Lock : 可中断锁，多样化同步，竞争激烈时能维持常态
- Atomic：竞争激烈时能维持常态，比Lock性能好；但是只能同步一个值。


## 可见性

- 导致共享变量在线程间不可见的原因

1. 线程交叉执行
2. 重排序结合线程交叉执行
3. 共享变量更新后的值没有在工作内存与主内存及时更新

## 可见性 --- synchronize

- Java内存模型关于synchronize的两条规定

1. 线程解锁前，必须把共享变量的最新值刷新到主内存中
2. 线程加锁前，将清空工作内存中的共享变量的值，从而使用共享变量时需要从主内存重新读取最新的值（注意：加锁和解锁是同一把锁）

### 可见性 --- volatile 

- 通过加入内存屏障和禁止重排序优化来实现

1. 对volatile变量写操作时，会在写操作后加入一条store屏障指令，将本地内存中的共享变量刷新到主内存中
2. 对volatile变量读操作时，会在读操作之前加入一条load屏障指令，从主内存中读取共享变量的值

## 发布对象

- 发布对象：
> 1. 使一个对象能够被当前范围之外的代码所使用

- 对象逸出：
> 一种错误的发布。当一个对象还没有构造完成的时候，就使它被其他线程所见

### 安全发布对象

1. 在静态初始化函数中创建一个对象引用
2. 将对象的引用保存到volatile类型域或者AtomicReference对象中
3. 将对象的引用保存到某个正确构造的final类型域中
3. 将对象的引用保存到一个由锁保护的域中

## 不可变对象

- 不可变对象满足的条件：

1. 对象创建以后其状态就不能修改
2. 对象所有域都是final类型
3. 对象是正确创建的（在对象创建期间，this引用没有逸出）

> 可以把类声明为final，表示类不能被继承，所有的成员都是private final ,不提供setter方法；get方法不返回对象本身，而是返回对象的拷贝，可以参考String类

> Collections.unmodifiableXXX 也是不可变的.Guava:ImmutableXXX 

## 线程封闭

- Ad-hoc线程封闭：程序控制实现，最糟糕的，忽略
- 堆栈封闭：使用局部变量，无并发问题
- ThreadLocal线程封闭：特别好的封闭方法

## 线程不安全的写法和类

- StringBuilder -> StringBuffer
- SimpleDateFomat -> JodaTime
- ArrayList HashSet HashMap
- 先检查再执行:if(condititon(a))(handler(a))

## 线程安全-同步容器

- ArrayList -> Vector Stack
- HashMap -> HashTable(key value 不能为null)
- Collections.synchronizedXXX(List,Set,Map)

## 线程安全 - 并发容器 J.U.C

- ArrayList -> CopyOnWriteArrayList 读多写少 读写分离 最终一致性 使用时另外开辟空间 读的时候没加锁，写的时候加了锁
- HashSet TreeSet -> CopyOnWriteArraySet(HashSet) 读多写少 ConcurrentSkipListSet(TreeSet，不允许null).批量操作还是要加锁，比如removeAll，addAll,containsAll等方法
- HashMap TreeMap -> ConcurrentHashMap,不允许null. ConcurrentSkipListMap,key是有序的,支持更高的并发

## AbstractQueuedSynchronizer - AQS

- 使用Node 实现FIFO队列，可以用于构建锁或者其他同步装置的基础框架
- 利用了一个int类型表示状态 state 字段 ReerentLock中的state表示获取锁的线程数量
- 使用方法是继承 子类通过继承并通过实现它的方法管理其状态（acquire 和 release）的方法操作状态
- 可以同时实现排它锁和共享锁模式（独占、共享）

## CyclicBarrier CountDownLunch
> CountDownLunch只能使用一次，CyclicBarrier可以使用reset方法重置，再次使用
> CountDownLunch主要是实现一个或多个线程需要等待其他线程 完成某项操作之后才能再执行其他操作，描述的是线程等待其他线程的关系。
CyclicBarrier实现了多个线程之间相互等待，直到所有线程都满足了条件之后才能继续执行后面的操作，描述的是各个线程内部相互等待的关系


## ReentrantLock (可重入锁) 与 锁

1. ReentrantLock 和 synchronized 的区别
- 可重入性 都是可重入锁
- 锁的实现 synchronized是基于jvm实现的 ReentrantLock基于JDK实现的
- 性能的区别 官方更推荐synchronized
- 功能区别1.synchronized更加方便和间接 2.ReentrantLock需要手动操作 3. ReentrantLock更具有灵活性

2. ReentrantLock独有的功能
- 可指定是公平锁还是非公平锁 synchronized只能是非公平锁. 公平锁指先等待的线程先获取锁。
- 提供了一个Condition类，可以分组唤醒需要唤醒的线程。synchronized要么唤醒随机一个线程，要么唤醒全部的线程
- 提供能够中断等待锁的线程的机制，lock.lockInterruptibly();

## 锁总结 
1. 当只有少量竞争者的时候，synchronized是比较好的选择，
2. 竞争者不少，但是线程增长的趋势是可预估的，ReentrantLock较好
3.synchronized不会造成死锁，其他的锁使用不当会造成死锁，因为某些情况没有执行Unlock操作

## 线程池

- new Thread 弊端

1. 每次new Thread新建对象，性能差
2. 线程缺乏统一管理，可能无限制的新建线程，相互竞争，有可能占用过多系统资源导致死机或OOM
3. 缺少更过功能，如更过执行、定期执行、线程中断等

- 线程池的好处

1. 重用存在的线程，减少对象创建、消亡的开销，性能好
2. 可有效控制最大并发线程数，提高系统资源利用率，同时可以避免过多资源竞争，避免阻塞
3. 提供定时执行、定期执行、单线程、并发数控制等功能

- 线程池 - ThreadPoolExecutor

1. corePoolSize : 核心线程数量
2. maximumPoolSize: 线程最大线程数
3. workQueue:阻塞队列，存储等待执行的任务，很重要，会对线程池运行过程产生重大影响
4. keepAliveTime：线程没有任务执行时最多保持多长时间终止
5. unit：keepAliveTime的时间单位
6. threadFactory:线程工厂，用来创建线程
7. rejectHandler：当拒绝处理任务时的策略


