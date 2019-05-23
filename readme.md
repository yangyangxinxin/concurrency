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
