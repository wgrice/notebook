# Interrupt 线程中断

在我们的程序中经常会有一些不达到目的不会退出的线程，例如：我们有一个下载程序线程，该线程在没有下载成功之前是不会退出的，若此时用户觉得下载速度慢，不想下载了，这时就需要用到我们的线程中断机制了，告诉线程，你不要继续执行了，准备好退出吧。当然，线程在不同的状态下遇到中断会产生不同的响应，有点会抛出异常，有的则没有变化，有的则会结束线程。本篇将从以下两个方面来介绍Java中对线程中断机制的具体实现：

- Java中对线程中断所提供的API支持
- 线程在不同状态下对于中断所产生的反应

## Java中对线程中断所提供的API支持

 在以前的 jdk 版本中，我们使用 stop 方法中断线程，但是现在的 jdk 版本中已经不再推荐使用该方法了，反而由 Thread 类的以下三个方法完成对线程中断的支持。

```java
// 中断线程（实例方法）
public void interrupt();
// 判断线程是否被中断（实例方法）
public boolean isInterrupted();
// 判断是否被中断并清除当前中断状态（静态方法）
public static boolean interrupted();
```

每个线程都一个状态位用于标识当前线程对象是否是中断状态。isInterrupted 是一个实例方法，主要用于判断当前线程对象的中断标志位是否被标记了，如果被标记了则返回 true 表示当前已经被中断，否则返回 false 。我们也可以看看它的实现源码：

```java
public boolean isInterrupted() {
	return isInterrupted(false);
}
private native boolean isInterrupted(boolean ClearInterrupted);
```

底层调用的本地方法 isInterrupted ，传入一个 boolean 类型的参数，用于指定调用该方法之后是否需要清除该线程对象的中断标识位。从这里我们也可以看出来，调用 isInterrupted 并**不会清除线程对象的中断标识位**。

interrupt 是一个实例方法，该方法用于设置当前线程对象的中断标识位。

interrupted 是一个静态的方法，用于返回当前线程是否被中断。

```java
public static boolean interrupted() {
    return currentThread().isInterrupted(true);
}
private native boolean isInterrupted(boolean ClearInterrupted);
```

该方法用于判断当前线程是否被中断，并且该方法调用结束的时候会**清空中断标识位**。

> 实例方法 isInterrupted 不会清楚线程的中断标识；
>
> 静态方法 interrupted 会清楚线程的中断标识；

## 线程在不同状态下对于中断所产生的反应

线程一共6种状态，分别是**NEW**，**RUNNABLE**，**BLOCKED**，**WAITING**，**TIMED_WAITING**，**TERMINATED**（ Thread 类中有一个 State 枚举类型列举了线程的所有状态）。下面我们就将把线程分别置于上述的不同种状态，然后看看我们的中断操作对它们的影响。

### NEW 

线程的 new 状态表示还未调用 start 方法，还未真正启动。该状态下调用中断方法来中断线程的时候，Java 认为毫无意义，所以并不会设置线程的中断标识位，什么事也不会发生。

```java
public class InterruptOnThreadStateNEW {
    public static void main(String[] args) {
        testNew();
    }

    private static void testNew() {
        Thread thread = new MyThread();
        // 此时线程状态
        System.out.println("Thread state: " + thread.getState());
        // 中断
        thread.interrupt();
        // 中断后线程状态
        System.out.println("Thread interrupt state: " + thread.isInterrupted());
    }

    private static class MyThread extends Thread {
    }
}

// Thread state: NEW
// Thread interrupt state: false
```

> 处于NEW 状态的线程对于中断是屏蔽的，也就是说中断操作对 NEW 状态下的线程是无效的。

### RUNNABLE

如果线程处于运行状态，那么该线程的状态就是 RUNNABLE ，但是不一定所有处于 RUNNABLE 状态的线程都能获得CPU运行，在某个时间段，只能由一个线程占用 CPU ，那么其余的线程虽然状态是 RUNNABLE ，但是都没有处于运行状态。而我们处于 RUNNABLE 状态的线程在遭遇中断操作的时候只会设置该线程的中断标志位，并不会让线程实际中断，想要发现本线程已经被要求中断了则需要用程序去判断。

我们定义的线程始终循环做一些事情，主线程启动该线程并输出该线程的状态，然后调用中断方法中断该线程并再次输出该线程的状态。

```java
public class InterruptOnThreadStateRUNNABLE1 {
    public static void main(String[] args) {
        testRunnable();
    }

    private static void testRunnable() {
        Thread thread = new MyThread();
        thread.start();
        System.out.println("Thread state: " + thread.getState());

        thread.interrupt();

        System.out.println("Thread interrupt state: " + thread.isInterrupted());

        System.out.println("Thread state: " + thread.getState());
    }

    private static class MyThread extends Thread {
        @Override
        public void run() {
            while(true){
                // do something
            }
        }
    }
}

// Thread state: RUNNABLE
// Thread interrupt state: true
// Thread state: RUNNABLE
```

可以看到在我们启动线程之后，线程状态变为 RUNNABLE ，中断之后输出中断标志，显然中断位已经被标记，但是当我们再次输出线程状态的时候发现，线程仍然处于 RUNNABLE 状态。很显然，处于 RUNNBALE 状态下的线程即便遇到中断操作，也只会设置中断标志位并不会实际中断线程运行。那么问题是，既然不能直接中断线程，我要中断标志有何用处？
这里其实 Java 将这种权力交给了我们的程序，Java 给我们提供了一个中断标志位，我们的程序可以通过 if 判断中断标志位是否被设置来中断我们的程序而不是系统强制的中断。

```java
public class InterruptOnThreadStateRUNNABLE2 {
    public static void main(String[] args) {
        testRunnable();
    }

    private static void testRunnable() {
        Thread thread = new MyThread();
        thread.start();
        System.out.println("Thread state: " + thread.getState());

        thread.interrupt();

        System.out.println("Thread interrupt state: " + thread.isInterrupted());
        // 等待 MyThread 判断中断标识后退出
        try { Thread.sleep(1000); } catch (InterruptedException e) { e.printStackTrace(); }

        System.out.println("Thread state: " + thread.getState());
    }

    private static class MyThread extends Thread {
        @Override
        public void run() {
            while(true){
                // if (Thread.interrupted()) {
                if (this.isInterrupted()) {
                    System.out.println("[MyThread]Thread interrupt state: " + this.isInterrupted());
                    System.out.println("[MyThread]exit MyThread");
                    break;
                }
            }
        }
    }
}

// 实例方法：this.isInterrupted()
// Thread state: RUNNABLE
// Thread interrupt state: true
// [MyThread]Thread interrupt state: true
// [MyThread]exit MyThread
// Thread state: TERMINATED

// 静态方法： Thread.interruped()
// Thread state: RUNNABLE
// Thread interrupt state: true
// [MyThread]Thread interrupt state: false
// [MyThread]exit MyThread
// Thread state: TERMINATED
```

线程一旦发现自己的中断标志位被设置了，立马跳出死循环。这样的设计好处就在于给了我们程序更大的灵活性。

> 处于 RUNNBALE 状态下的线程即便遇到中断操作，也只会设置中断标志位并不会实际中断线程运行；
>
> 线程可以通过循环判断当前线程的中断标识被置位了，立马跳出死循环，实现线程真正的中断；

### BLOCKED

当线程处于 BLOCKED 状态说明该线程由于竞争某个对象的锁失败而被挂在了该对象的阻塞队列上了。那么此时发起中断操作不会对该线程产生任何影响，依然只是设置中断标志位。

在我们的主线程中，我们定义了两个线程并按照定义顺序启动他们，显然 thread1 启动后便占用 MyThread 类锁，此后 thread2 在获取锁的时候一定失败，自然被阻塞在阻塞队列上，而我们对 thread2 进行中断。

```java
public class InterruptOnThreadStateBLOCKED {
    public static void main(String[] args) {
        testBlocked();
    }

    private static void testBlocked() {
        Thread thread1 = new MyThread();
        thread1.start();
        // 后启动的线程 2 由于无法获取锁将处于 BLOCKED 状态
        Thread thread2 = new MyThread();
        thread2.start();

        try { Thread.sleep(1000); } catch (InterruptedException e) { e.printStackTrace(); }
        System.out.println("Thread1 state: " + thread1.getState());
        System.out.println("Thread2 state: " + thread2.getState());
        // 中断处于 BLOCKED 的线程 2
        thread2.interrupt();

        System.out.println("Thread1 interrupt state: " + thread1.isInterrupted());
        System.out.println("Thread2 interrupt state: " + thread2.isInterrupted());

        try { Thread.sleep(1000); } catch (InterruptedException e) { e.printStackTrace(); }

        System.out.println("Thread1 state: " + thread1.getState());
        System.out.println("Thread2 state: " + thread2.getState());
    }

    private static class MyThread extends Thread {

        public synchronized static void doSomething() {
            while(true){
                //do something
            }
        }

        @Override
        public void run() {
            doSomething();
        }
    }
}

// Thread1 state: RUNNABLE
// Thread2 state: BLOCKED
// Thread1 interrupt state: false
// Thread2 interrupt state: true
// Thread1 state: RUNNABLE
// Thread2 state: BLOCKED
```

从输出结果看来，thread2 处于 BLOCKED 状态，执行中断操作之后，该线程仍然处于 BLOCKED 状态，但是中断标志位却已被修改。这种状态下的线程和处于 RUNNABLE 状态下的线程是类似的，给了我们程序更大的灵活性去判断和处理中断。

> 处于 BLOCKED 状态下的线程遇到中断操作不会对该线程产生任何影响，依然只是设置中断标志位。

### WAITING

处于 WAITING 状态的线程是由于在运行的过程中缺少某些条件而被挂起在某个对象的等待队列上，需要其他线程调用notify方法释放自己。当这些线程遇到中断操作的时候，**会抛出一个 InterruptedException 异常，并清空中断标志位**。

```java
public class InterruptOnThreadStateWAITING {
    public static void main(String[] args) {
        testWaiting();
    }

    private static void testWaiting() {
        Thread thread = new MyThread();
        thread.start();
        // 当前线程状态
        System.out.println("Thread state: " + thread.getState());
        // 中断线程
        thread.interrupt();
        // 等待线程处理 InterruptedException 异常
        try { Thread.sleep(1000); } catch (InterruptedException e) { e.printStackTrace(); }
        // 最终线程状态
        System.out.println("Thread interrupt state: " + thread.isInterrupted());
    }

    private static class MyThread extends Thread {
        @Override
        public void run() {
            synchronized (this){
                try {
                    wait();
                } catch (InterruptedException e) {
                    System.out.println("[MyThread]Catch InterruptedException, thread interrupt state: " + this.isInterrupted());
                }
            }
        }
    }
}

// Thread state: RUNNABLE
// [MyThread]Catch InterruptedException, thread interrupt state: false
// Thread state: TERMINATED
```

从运行结果看，当前程thread启动之后就被挂起到该线程对象的条件队列上，然后我们调用interrupt方法对该线程进行中断，输出了我们在catch中的输出语句，显然是捕获了InterruptedException异常，接着就看到该线程的中断标志位被清空。

> 处于 WAITING 状态下的线程遇到中断操作会抛出一个 InterruptedException 异常，并清空中断标志位。

### TIMED_WAITING

TIMED_WAITING 与 WAITING 两种状态本质上是同一种状态，只不过 TIMED_WAITING 在等待一段时间后会自动释放自己，而WAITING则是无限期等待，其他线程调用 notify/notifyAll 方法时会释放自己。因此处于 TIMED_WAITING 状态下的线程遇到中断操作时与 WAITING 状态一致。

> 处于 TIMED_WAITING 状态下的线程遇到中断操作会抛出一个 InterruptedException 异常，并清空中断标志位。

### TERMINATED

​     线程的 terminated 状态表示线程已经运行终止。该状态下调用中断方法来中断线程的时候，Java 认为毫无意义，所以并不会设置线程的中断标识位，什么事也不会发生。

```java
public class InterruptOnThreadStateTERMINATED {
    public static void main(String[] args) {
        testTerminated();
    }

    private static void testTerminated() {
        Thread thread = new MyThread();
        thread.start();
        try {
            thread.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 此时线程状态
        System.out.println("Thread state: " + thread.getState());
        // 中断
        thread.interrupt();
        // 中断后线程状态
        System.out.println("Thread interrupt state: " + thread.isInterrupted());
    }

    private static class MyThread extends Thread {
    }
}

// Thread state: TERMINATED
// Thread interrupt state: false
```

> 处于TERMINATED 状态的线程对于中断是屏蔽的，也就是说中断操作对 TERMINATED 状态下的线程是无效的。

## Thread.interrupt() 和 Thread.stop() 的区别

- 相同点：当线程在等待内置锁或IO时，stop 跟 interrupt 一样，不会中止这些操作；当 catch 住 stop 导致的异常时，程序也可以继续执行

- 不同点：中断需要程序自己去检测然后做相应的处理，对象状态一致性就是可控的；而 Thread.stop 会直接在代码执行过程中抛出 ThreadDeath 错误，这是一个 java.lang.Error 的子类。程序运行时被 stop 的位置可能出现在很多地方，不可控，极易造成对象状态的不一致。