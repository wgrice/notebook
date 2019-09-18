## AQS 原理概述

AQS 的全称为（ AbstractQueuedSynchronizer ），这个类在 java.util.concurrent.locks 包下面。

AQS是一个用来构建锁和同步器的框架，使用 AQS 能简单且高效地构造出应用广泛的大量的同步器，比如我们提到的 ReentrantLock、 Semaphore 。当然，我们自己也能利用AQS非常轻松容易地构造出符合我们自己需求的同步器。

AQS 维护了一个 `volatile int state` （代表共享资源）和一个 FIFO 线程等待队列（**用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中** ）。

> CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点（Node）来实现锁的分配。

AQS使用一个int成员变量来表示同步状态，通过内置的FIFO队列来完成获取资源线程的排队工作。AQS使用CAS对该同步状态进行原子操作实现对其值的修改。

```java
private volatile int state; // 共享变量，使用 volatile 修饰保证线程可见性
```

状态信息通过 procted 类型的 getState，setState，compareAndSetState 进行操作。

```java
// 返回同步状态的当前值
protected final int getState() {  
        return state;
}
// 设置同步状态的值
protected final void setState(int newState) { 
        state = newState;
}
// 原子的（ CAS操作 ）将同步状态值设置为给定值 update 如果当前同步状态的值等于 expect （期望值）
protected final boolean compareAndSetState(int expect, int update) {
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

## AQS 对资源的共享方式

AQS定义两种资源共享方式

- **Exclusive（独占）**：只有一个线程能执行，如 ReentrantLock 。又可分为公平锁和非公平锁：
  - 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
  - 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的
- **Share（共享）**：多个线程可同时执行，如 Semaphore、CountDownLatCh、 CyclicBarrier、ReentrantReadWriteLock 。 ReentrantReadWriteLock 可以看成是组合式，因为 ReentrantReadWriteLock 也就是读写锁允许多个线程同时对某一资源进行读。

不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源 state 的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在上层已经帮我们实现好了。

## AQS底层使用了模板方法模式

同步器的设计是基于模板方法模式的，如果需要自定义同步器一般的方式是这样（模板方法模式很经典的一个应用）：

使用者继承 AbstractQueuedSynchronize r并重写指定的方法。（这些重写方法很简单，无非是对于共享资源 state 的获取和释放）
将 AQS 组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。

这和我们以往通过实现接口的方式有很大区别，这是模板方法模式很经典的一个运用，下面简单的给大家介绍一下模板方法模式，模板方法模式是一个很容易理解的设计模式之一。

> 模板方法模式是基于”继承“的，主要是为了在不改变模板结构的前提下在子类中重新定义模板中的内容以实现复用代码。举个很简单的例子假如我们要去一个地方的步骤是：购票 buyTicket() -> 安检 securityCheck() -> 乘坐某某工具回家 ride() ->  到达目的地 arrive() 。我们可能乘坐不同的交通工具回家比如飞机或者火车，所以除了ride()方法，其他方法的实现几乎相同。我们可以定义一个包含了这些方法的抽象类，然后用户根据自己的需要继承该抽象类然后修改 ride()方法。

AQS使用了模板方法模式，自定义同步器时需要重写下面几个AQS提供的模板方法：

```java
// 该线程是否正在独占资源。只有用到 condition 才需要去实现它。
isHeldExclusively()
// 独占方式。尝试获取资源，成功则返回 true ，失败则返回 false 。
tryAcquire(int)
// 独占方式。尝试释放资源，成功则返回 true ，失败则返回 false 。
tryRelease(int)
// 共享方式。尝试获取资源。负数表示失败，0 表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryAcquireShared(int)
// 共享方式。尝试释放资源，成功则返回 true ，失败则返回 false 。
tryReleaseShared(int)
```

默认情况下，每个方法都抛出 UnsupportedOperationException 。 这些方法的实现必须是**内部线程安全**的，并且通常应该简短而不是阻塞。AQS类中的其他方法都是 final ，所以无法被其他类使用，只有这几个方法可以被其他类使用。

以 ReentrantLock 为例，state 初始化为 0，表示未锁定状态。A 线程 `lock()` 时，会调用 `tryAcquire()` 独占该锁并将 `state+1` 。此后其他线程再 `tryAcquire()` 时就会失败，直到 A 线程 `unlock()` 到 `state=0` （即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A 线程自己是可以重复获取此锁的（ state 会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证 state 是能回到零态的。

再以 CountDownLatch 以例，任务分为 N 个子线程去执行，state 也初始化为 N （注意 N 要与线程个数一致）。这 N 个子线程是并行执行的，每个子线程执行完后 `	countDown()` 一次，state 会 `CAS(Compare and Swap) ` 减 1 。等到所有子线程都执行完后（即 `state=0` ），会  `unpark()` 主调用线程，然后主调用线程就会从 `await()` 函数返回，继续后余动作。

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现 tryAcquire/tryRelease、tryAcquireShared/tryReleaseShared 中的一种即可。但 AQS 也支持自定义同步器同时实现独占和共享两种方式，如ReentrantReadWriteLock 。

## AQS 同步组件

### Semaphore （信号量）

synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源，Semaphore （信号量）可以**指定多个线程同时访问某个资源**。

执行 `acquire` 方法阻塞，直到有一个许可证可以获得然后拿走一个许可证；每个 `release` 方法增加一个许可证，这可能会释放一个阻塞的 `acquire` 方法。然而，其实并没有实际的许可证这个对象，Semaphore 只是维持了一个可获得许可证的数量。 Semaphore 经常用于限制获取某种资源的线程数量。

当然一次也可以一次拿取和释放多个许可，不过一般没有必要这样做：

```java
semaphore.acquire(5);// 获取5个许可
semaphore.release(5);// 释放5个许可
```
除了 `acquire` 方法之外，另一个比较常用的与之对应的方法是 `tryAcquire` 方法，该方法如果获取不到许可就立即返回false。

Semaphore 有两种模式，公平模式和非公平模式。

- 公平模式： 调用 acquire 的顺序就是获取许可证的顺序，遵循 FIFO ；

- 非公平模式： 抢占式的。

### CountDownLatch （倒计时器）

CountDownLatch 是一个同步工具类，用来协调多个线程之间的同步。这个工具通常用来控制线程等待，它可以让某一个线程等待直到倒计时结束，再开始执行。

#### CountDownLatch 的两种典型用法

1. 某一线程在开始运行前等待 n 个线程执行完毕。将 CountDownLatch 的计数器初始化为 n （ `new CountDownLatch(n)` ），每当一个任务线程执行完毕，就将计数器减 1 （ `countdownlatch.countDown()` ），当计数器的值变为 0 时，在CountDownLatch 上 `await()` 的线程就会被唤醒。一个典型应用场景就是启动一个服务时，主线程需要等待多个组件加载完毕，之后再继续执行。

2. 实现多个线程开始执行任务的最大并行性。注意是并行性，不是并发，强调的是多个线程在某一时刻同时开始执行。类似于赛跑，将多个线程放到起点，等待发令枪响，然后同时开跑。做法是初始化一个共享的 CountDownLatch 对象，将其计数器初始化为 1 （ `new CountDownLatch(1)` ），多个线程在开始执行任务前首先 `coundownlatch.await()`，当主线程调用  `countDown()` 时，计数器变为 0 ，多个线程同时被唤醒。

#### CountDownLatch 的不足

**CountDownLatch 是一次性的**，计数器的值只能在构造方法中初始化一次，之后没有任何机制再次对其设置值，当 CountDownLatch 使用完毕后，它不能再次被使用。

### CyclicBarrier （循环栅栏）

CyclicBarrier 和 CountDownLatch 非常类似，它也可以实现线程间的技术等待，但是它的功能比 CountDownLatch 更加复杂和强大。主要应用场景和 CountDownLatch 类似。

CyclicBarrier 的字面意思是可循环使用（ Cyclic ）的屏障（ Barrier ）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。CyclicBarrier 默认的构造方法是 `CyclicBarrier(int parties)` ，其参数表示屏障拦截的线程数量，每个线程调用 `await` 方法告诉 CyclicBarrier 我已经到达了屏障，然后当前线程被阻塞。

CyclicBarrier 还提供一个更高级的构造函数`CyclicBarrier(int parties, Runnable barrierAction)`，用于在线程到达屏障时，优先执行`barrierAction`，方便处理更复杂的业务场景。

#### CyclicBarrier 的应用场景

CyclicBarrier 可以用于多线程计算数据，最后合并计算结果的应用场景。比如我们用一个Excel保存了用户所有银行流水，每个Sheet保存一个帐户近一年的每笔银行流水，现在需要统计用户的日均银行流水，先用多线程处理每个 sheet 里的银行流水，都执行完之后，得到每个 sheet 的日均银行流水，最后，再用 barrierAction 用这些线程的计算结果，计算出整个 Excel 的日均银行流水。

#### CyclicBarrier和CountDownLatch的区别

CountDownLatch 是计数器，只能使用一次，而 CyclicBarrier 的计数器提供 reset 功能，可以多次使用。但是我不那么认为它们之间的区别仅仅就是这么简单的一点。我们来从 jdk 作者设计的目的来看，javadoc 是这么描述它们的：

> CountDownLatch: A synchronization aid that allows one or more threads to wait until a set of operations being performed in other threads completes.
> 一个或者多个线程，等待其他多个线程完成某件事情之后才能执行；
> 
> CyclicBarrier : A synchronization aid that allows a set of threads to all wait for each other to reach a common barrier point.
> 多个线程互相等待，直到到达同一个同步点，再继续一起执行。

对于 CountDownLatch 来说，重点是“一个线程（多个线程）等待”，而其他的 N 个线程在完成“某件事情”之后，可以终止，也可以等待。而对于CyclicBarrier，重点是多个线程，在任意一个线程没有完成，所有的线程都必须等待。

CountDownLatch 是计数器，线程完成一个记录一个，只不过计数不是递增而是递减，而 CyclicBarrier 更像是一个阀门，需要所有线程都到达，阀门才能打开，然后继续执行。

| CountDownLatch | CyclicBarrier |
| :-----: | :-----: |
|                          减计数方式                          |                          加计数方式                          |
|                 计数为 0 时释放所有等待线程                  |               计数达到指定值时释放所有等待线程               |
|                     计数为 0 时无法重置                      |            计数达到指定值时，计数置为 0 重新开始             |
| 调用 `CountDown()` 方法计数减一，调用 `await()`方法只进行阻塞，不影响计数 | 调用 `await() `方法计数加1，若加1后的值不等于指定值则线程阻塞 |
|                         不可重复利用                         |                          可重复利用                          |

### ReentrantLock 

 Lock 对象锁还提供了 synchronized 所不具备的其他同步特性，如**可中断锁的获取**（ synchronized 在等待获取锁时是不可中的），**超时中断锁的获取**，**等待唤醒机制的多条件变量 Condition** 等，这也使得 Lock 锁在使用上具有更大的灵活性。

### ReentrantReadWriteLock 

#### ReentrantReadWriteLock  原理

实现读写锁与实现普通互斥锁的主要区别在于需要分别记录读锁状态及写锁状态，并且等待队列中需要区别处理两种加锁操作。 
 `Sync `使用 `state` 变量同时记录读锁与写锁状态，将 `int` 类型的 `state` 变量分为高 16 位与第 16 位，高 16 位记录读锁状态，低 16 位记录写锁状态。

`Sync` 使用不同的 `mode` 描述等待队列中的节点以区分读锁等待节点和写锁等待节点。`mode `取值包括 `SHARED` 及`EXCLUSIVE `两种，分别代表当前等待节点为读锁和写锁。

读写所允许同一时刻被多个读线程访问，但是在写线程访问时，所有的读线程和其他的写线程都会被阻塞。

- 读-读不互斥：读读之间不阻塞
- 读-写互斥：读堵塞写，写也阻塞读
- 写-写互斥：写写阻塞

#### ReentrantReadWriteLock 锁降级

锁降级算是获取读锁的特例，如在`t0`线程已经获取写锁的情况下，再调取读锁加锁函数则可以直接获取读锁，但此时其他线程仍然无法获取读锁或写锁，在`t0`线程释放写锁后，如果有节点等待则会唤醒后续节点，后续节点可见的状态为目前有`t0`线程获取了读锁。 
所降级有什么应用场景呢？引用读写锁中使用示例代码

```java
class CachedData {
    Object data;
    volatile boolean cacheValid;
    final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();

    void processCachedData() {
        rwl.readLock().lock();
        if (!cacheValid) {
            // Must release read lock before acquiring write lock
            rwl.readLock().unlock();
            rwl.writeLock().lock();
            try {
            // Recheck state because another thread might have
            // acquired write lock and changed state before we did.
            if (!cacheValid) {
                data = ...
                cacheValid = true;
            }
            // Downgrade by acquiring read lock before releasing write lock
            rwl.readLock().lock();
            } finally {
            rwl.writeLock().unlock(); // Unlock write, still hold read
            }
        }
        try {
            use(data);
        } finally {
            rwl.readLock().unlock();
        }
    }
 }
```

其中针对变量`cacheValid`的使用主要过程为加读锁、读取、释放读锁、加写锁、修改值、加读锁、释放写锁、使用数据、释放读锁。其中后续几步（加写锁、修改值、加读锁、释放写锁、使用数据、释放读锁）为典型的锁降级。如果不使用锁降级，则过程可能有三种情况：

- 第一种：加写锁、修改值、释放写锁、使用数据，即使用写锁修改数据后直接使用刚修改的数据，这样可能有数据的不一致，如当前线程释放写锁的同时其他线程（如`t0`）获取写锁准备修改（还没有改）`cacheValid`变量，而当前线程却继续运行，则当前线程读到的`cacheValid`变量的值为`t0`修改前的老数据；
- 第二种：加写锁、修改值、使用数据、释放写锁，即将修改数据与再次使用数据合二为一，这样不会有数据的不一致，但是由于混用了读写两个过程，以排它锁的方式使用读写锁，减弱了读写锁读共享的优势，增加了写锁（独占锁）的占用时间；
- 第三种：加写锁、修改值、释放写锁、加读锁、使用数据、释放读锁，即使用写锁修改数据后再请求读锁来使用数据，这是时数据的一致性是可以得到保证的，但是由于释放写锁和获取读锁之间存在时间差，则当前想成可能会需要进入等待队列进行等待，可能造成线程的阻塞降低吞吐量。

因此针对以上情况提供了锁的降级功能，可以在完成数据修改后尽快读取最新的值，且能够减少写锁占用时间。 

最后注意，读写锁**不支持锁升级**，即获取读锁、读数据、获取写锁、释放读锁、释放写锁这个过程，因为读锁为共享锁，如同时有多个线程获取了读锁后有一个线程进行锁升级获取了写锁，这会造成同时有读锁（其他线程）和写锁的情况，造成其他线程可能无法感知新修改的数据（此为逻辑性错误），并且在JAVA读写锁实现上由于当前线程获取了读锁，再次请求写锁时必然会阻塞而导致后续释放读锁的方法无法执行，这回造成死锁（此为功能性错误）。

#### ReentrantReadWriteLock  特点

1. 创建，两种 fair 和 unfair
   - unfair：read 和 write 锁获取依赖于 Reentrancy 规则，不存在先后顺序
   - fair：各个线程按照时间顺序进行锁的获取，当一个线程试图获取 read 锁时，如果当前没有线程获取 write 锁且试图获取 write 锁的线程请求时间没有请求read锁的时间久，则获取 read 锁。当一个线程试图获取 write 锁时，当且仅当当前所有的 read 锁和 write 锁全部释放才能成功（ `ReadLoac.tryLock()` 和 `WriteLock.tryLock()` 不受该约束）

2. ReentrantReadWriteLock 提供了两个锁，readLock 和 writeLock ，readLock 可多线程并发执行，writeLock 只能单线程执行（类似于synchronized），线程获取 writeLock 锁之后可以继续获取 readLock ，反过来不行；

3. 降级：writeLock 变成 readLock 成为锁降级，具体操作为 `writeLock.lock` -> `readLock.lock` -> `writeLock.unlock` ，这样就把 writeLock 变成了 readLock ；

4. 两种锁再获取锁期间都支持 interruption ；

5. 只有 writeLock 支持 condition ，通过 writeLock.newCondition 获得 condition 对象，对 writeLock 进行 await 和 signal 、signalAll 控制，因为 readLock本 身支持多线程迸发访问，所以 condition 控制对 readLock 没什么用 ；