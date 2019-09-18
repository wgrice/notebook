# Volatile

volatile 是 Java 虚拟机提供的轻量级的同步机制。

## Volatile特点

1. 保证可见性
2. 不保证原子性
3. 禁止指令重排

## Volatile 内存语义

volatile在并发编程中很常见，但也容易被滥用，现在我们就进一步分析 volatile 关键字的语义。volatile关键字有如下两个作用：

- 保证被volatile修饰的共享变量对所有线程总数可见的，也就是当一个线程修改了一个被 volatile 修饰共享变量的值，新值总是可以被其他线程立即得知。
- 禁止指令重排序优化。

## Volatile 的可见性

关于volatile的可见性作用，我们必须意识到被 volatile 修饰的变量对所有线程总数立即可见的，对 volatile 变量的所有写操作总是能立刻反应到其他线程中，但是对于 volatile 变量运算操作在多线程环境并不保证安全性，如下：

```java
public class VolatileVisibility {
    public static volatile int i =0;

    public static void increase(){
        i++;
    }
}
```

正如上述代码所示，i变量的任何改变都会立马反应到其他线程中，但是如此存在多条线程同时调用 increase() 方法的话，就会出现线程安全问题，毕竟 `i++` ；操作并不具备原子性，该操作是先读取值，然后写回一个新值，相当于原来的值加上1，分两步完成，如果第二个线程在第一个线程读取旧值和写回新值期间读取 i 的域值，那么第二个线程就会与第一个线程一起看到同一个值，并执行相同值的加1操作，这也就造成了线程安全失败，因此对于increase方法必须使用 synchronized 修饰，以便保证线程安全，需要注意的是一旦使用 synchronized 修饰方法后，由于 synchronized 本身也具备与 volatile 相同的特性，即可见性，因此在这样种情况下就完全可以省去volatile 修饰变量。

```java
public class VolatileVisibility {
    public static int i =0;

    public synchronized static void increase(){
        i++;
    }
}
```

现在来看另外一种场景，可以使用volatile修饰变量达到线程安全的目的，如下：

```java
public class VolatileSafe {

    volatile boolean close;

    public void close(){
        close=true;
    }

    public void doWork(){
        while (!close){
            System.out.println("safe....");
        }
    }
}
```

由于对于boolean变量close值的修改属于原子性操作，因此可以通过使用volatile修饰变量close，使用该变量对其他线程立即可见，从而达到线程安全的目的。那么JMM是如何实现让volatile变量对其他线程立即可见的呢？实际上，当写一个volatile变量时，JMM会把该线程对应的工作内存中的共享变量值刷新到主内存中，当读取一个volatile变量时，JMM会把该线程对应的工作内存置为无效，那么该线程将只能从主内存中重新读取共享变量。volatile变量正是通过这种写-读方式实现对其他线程可见（但其内存语义实现则是通过内存屏障，稍后会说明）。

## Volatile 禁止重排优化

volatile关键字另一个作用就是禁止指令重排优化，从而避免多线程环境下程序出现乱序执行的现象，关于指令重排优化前面已详细分析过，这里主要简单说明一下volatile是如何实现禁止指令重排优化的。先了解一个概念，内存屏障(Memory Barrier）。 

内存屏障，又称内存栅栏，是一个CPU指令，它的作用有两个，一是保证特定操作的执行顺序，二是保证某些变量的内存可见性（利用该特性实现volatile的内存可见性）。由于编译器和处理器都能执行指令重排优化。如果在指令间插入一条Memory Barrier则会告诉编译器和CPU，不管什么指令都不能和这条Memory Barrier指令重排序，也就是说通过插入内存屏障禁止在内存屏障前后的指令执行重排序优化。Memory Barrier的另外一个作用是强制刷出各种CPU的缓存数据，因此任何CPU上的线程都能读取到这些数据的最新版本。总之，volatile变量正是通过内存屏障实现其在内存中的语义，即可见性和禁止重排优化。下面看一个非常典型的禁止重排优化的例子DCL，如下：

```java
public class DoubleCheckLock {
    private static DoubleCheckLock instance;
    
    private DoubleCheckLock(){}
    
    public static DoubleCheckLock getInstance(){
        //第一次检测
        if (instance==null){
            //同步
            synchronized (DoubleCheckLock.class){
                if (instance == null){
                    //多线程环境下可能会出现问题的地方
                    instance = new DoubleCheckLock();
                }
            }
        }
        return instance;
    }
}
```

上述代码一个经典的单例的双重检测的代码，这段代码在单线程环境下并没有什么问题，但如果在多线程环境下就可以出现线程安全问题。原因在于某一个线程执行到第一次检测，读取到的instance不为null时，instance的引用对象可能没有完成初始化。因为instance = new DoubleCheckLock();可以分为以下3步完成(伪代码)

```java
memory = allocate(); //1.分配对象内存空间
instance(memory);    //2.初始化对象
instance = memory;   //3.设置instance指向刚分配的内存地址，此时instance！=null
```

由于步骤1和步骤2间可能会重排序，如下：

```java
memory = allocate(); //1.分配对象内存空间
instance = memory;   //3.设置instance指向刚分配的内存地址，此时instance！=null，但是对象还没有初始化完成！
instance(memory);    //2.初始化对象
```

由于步骤2和步骤3不存在数据依赖关系，而且无论重排前还是重排后程序的执行结果在单线程中并没有改变，因此这种重排优化是允许的。但是指令重排只会保证串行语义的执行的一致性(单线程)，但并不会关心多线程间的语义一致性。所以当一条线程访问instance不为null时，由于instance实例未必已初始化完成，也就造成了线程安全问题。那么该如何解决呢，很简单，我们使用volatile禁止instance变量被执行指令重排优化即可。

```java
//禁止指令重排优化
private volatile static DoubleCheckLock instance;
```

到此相信我们对Java内存模型和volatile应该都有了比较全面的认识，总而言之，我们应该清楚知道，JMM就是一组规则，这组规则意在解决在并发编程可能出现的线程安全问题，并提供了内置解决方案（happen-before原则）及其外部可使用的同步手段(synchronized/volatile等)，确保了程序执行在多线程环境中的应有的原子性，可视性及其有序性。
