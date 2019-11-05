## 对 ThreadLocal 的理解

ThreadLocal 很多地方叫做线程本地变量，也有些地方叫做线程本地存储，其实意思差不多。可能很多朋友都知道 ThreadLocal为变量在每个线程中都创建了一个副本，那么每个线程可以访问自己内部的副本变量。

我们还是先来看一个例子：

```java
class ConnectionManager {  

    private static Connection connect = null;  

    public static Connection openConnection() {  
        if(connect == null){  
            connect = DriverManager.getConnection();  
        }  
        return connect;  
    }  

    public static void closeConnection() {  
        if(connect!=null)  
            connect.close();  
    }  
}  
```

假设有这样一个数据库链接管理类，这段代码在单线程中使用是没有任何问题的，但是如果在多线程中使用呢？很显然，在多线程中使用会存在线程安全问题：第一，这里面的 2 个方法都没有进行同步，很可能在 openConnection 方法中会多次创建connect；第二，由于 connect 是共享变量，那么必然在调用 connec t的地方需要使用到同步来保障线程安全，因为很可能一个线程在使用 connect 进行数据库操作，而另外一个线程调用 closeConnection 关闭链接。

所以出于线程安全的考虑，必须将这段代码的两个方法进行同步处理，并且在调用connect的地方需要进行同步处理。

这样将会大大影响程序执行效率，因为一个线程在使用connect进行数据库操作的时候，其他线程只有等待。

那么大家来仔细分析一下这个问题，这地方到底需不需要将connect变量进行共享？事实上，是不需要的。假如每个线程中都有一个connect变量，各个线程之间对connect变量的访问实际上是没有依赖关系的，即一个线程不需要关心其他线程是否对这个connect进行了修改的。

到这里，可能会有朋友想到，既然不需要在线程之间共享这个变量，可以直接这样处理，在每个需要使用数据库连接的方法中具体使用时才创建数据库链接，然后在方法调用完毕再释放这个连接。比如下面这样：

```java
class ConnectionManager {  

    private Connection connect = null;  

    public Connection openConnection() {  
        if(connect == null){  
            connect = DriverManager.getConnection();  
        }  
        return connect;  
    }  

    public void closeConnection() {  
        if(connect!=null)  
            connect.close();  
    }  
}  

class Dao{  
    public void insert() {  
        ConnectionManager connectionManager = new ConnectionManager();  
        Connection connection = connectionManager.openConnection();  

        //使用connection进行操作  

        connectionManager.closeConnection();  
    }  
}  
```

这样处理确实也没有任何问题，由于每次都是在方法内部创建的连接，那么线程之间自然不存在线程安全问题。但是这样会有一个致命的影响：导致服务器压力非常大，并且严重影响程序执行性能。由于在方法中需要频繁地开启和关闭数据库连接，这样不仅严重影响程序执行效率，还可能导致服务器压力巨大。

那么这种情况下使用 ThreadLocal 是再适合不过的了，因为 ThreadLocal 在每个线程中对该变量会创建一个副本，即每个线程内部都会有一个该变量，且在线程内部任何地方都可以使用，线程之间互不影响，这样一来就不存在线程安全问题，也不会严重影响程序执行性能。

但是要注意，虽然 ThreadLocal 能够解决上面说的问题，但是由于在每个线程中都创建了副本，所以要考虑它对资源的消耗，比如内存的占用会比不使用 ThreadLocal 要大。

## 深入解析 ThreadLocal 类

在上面谈到了对ThreadLocal的一些理解，那我们下面来看一下具体 ThreadLocal 是如何实现的。

先了解一下 ThreadLocal 类提供的几个方法：

```java
public T get() { }  
public void set(T value) { }  
public void remove() { }  
protected T initialValue() { }  
```

`get()` 方法是用来获取 ThreadLocal 在当前线程中保存的变量副本，`set()` 用来设置当前线程中变量的副本，`remove()` 用来移除当前线程中变量的副本，`initialValue()` 是一个 protected 方法，一般是用来在使用时进行重写的，它是一个延迟加载方法，下面会详细说明。

```java
/**
 * Returns the value in the current thread's copy of this
 * thread-local variable.  If the variable has no value for the
 * current thread, it is first initialized to the value returned
 * by an invocation of the {@link #initialValue} method.
 *
 * @return the current thread's value of this thread-local
 */
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```
第一句是取得当前线程，然后通过 `getMap(t)` 方法获取到一个 map，map 的类型为 ThreadLocalMap 。然后接着下面获取到 <key,value> 键值对，注意这里获取键值对传进去的是 this，而不是当前线程 t。

如果获取成功，则返回 value 值。

如果 map 为空，则调用 `setInitialValue` 方法返回 value 。

我们上面的每一句来仔细分析：

首先看一下 `getMap` 方法中做了什么：

```java
/**
 * Get the map associated with a ThreadLocal. Overridden in
 * InheritableThreadLocal.
 *
 * @param  t the current thread
 * @return the map
 */
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```
可能大家没有想到的是，在 getMap 中，是调用当期线程 t，返回当前线程 t 中的一个成员变量 threadLocals。

那么我们继续取 Thread 类中取看一下成员变量 threadLocals 是什么：

```java
 /* ThreadLocal values pertaining to this thread. This map is maintained
  * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;
```


实际上就是一个 ThreadLocalMap，这个类型是 ThreadLocal 类的一个内部类，我们继续取看 ThreadLocalMap 的实现：

```java
/**
 * ThreadLocalMap is a customized hash map suitable only for
 * maintaining thread local values. No operations are exported
 * outside of the ThreadLocal class. The class is package private to
 * allow declaration of fields in class Thread.  To help deal with
 * very large and long-lived usages, the hash table entries use
 * WeakReferences for keys. However, since reference queues are not
 * used, stale entries are guaranteed to be removed only when
 * the table starts running out of space.
 */
static class ThreadLocalMap {
 
    /**
     * The entries in this hash map extend WeakReference, using
     * its main ref field as the key (which is always a
     * ThreadLocal object).  Note that null keys (i.e. entry.get()
     * == null) mean that the key is no longer referenced, so the
     * entry can be expunged from table.  Such entries are referred to
     * as "stale entries" in the code that follows.
     */
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;
 
        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
//...其他
}
```

可以看到 ThreadLocalMap 的 Entry 继承了 WeakReference，并且使用 ThreadLocal 作为键值。

然后再继续看 `setInitialValue()`方法的具体实现：

```java
/**
 * Variant of set() to establish initialValue. Used instead
 * of set() in case user has overridden the set() method.
 *
 * @return the initial value
 */
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```

很容易了解，就是如果 map 不为空，就设置键值对，为空，再创建 Map，看一下 `createMap()` 的实现：

```java
/**
 * Create the map associated with a ThreadLocal. Overridden in
 * InheritableThreadLocal.
 *
 * @param t the current thread
 * @param firstValue value for the initial entry of the map
 */
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```


至此，可能大部分朋友已经明白了 ThreadLocal 是如何为每个线程创建变量的副本的：

首先，在每个线程 Thread 内部有一个 ThreadLocal.ThreadLocalMap 类型的成员变量 threadLocals ，这个 threadLocals 就是用来存储实际的变量副本的，键值为当前 ThreadLocal 变量，value 为变量副本（即T类型的变量）。

初始时，在 Thread 里面，threadLocals 为空，当通过 ThreadLocal 变量调用 `get()` 方法或者 `set()` 方法，就会对 Thread 类中的 threadLocals 进行初始化，并且以当前 ThreadLocal 变量为键值，以 ThreadLocal 要保存的副本变量为 value ，存到 threadLocals 。

然后在当前线程里面，如果要使用副本变量，就可以通过 get 方法在 threadLocals 里面查找。

下面通过一个例子来证明通过 ThreadLocal 能达到在每个线程中创建变量副本的效果：

```java

package com.milo.jdk.lang;
/**
 * You need to setting before getting it, otherwise there will be a NullPointerException
 * @author MILO
 *
 */
public class MiloTheadLocal {
	  	ThreadLocal<Long> longLocal = new ThreadLocal<Long>();  
	    ThreadLocal<String> stringLocal = new ThreadLocal<String>();  
	  
	    public void set() {  
	    	longLocal.set(Thread.currentThread().getId());
	    	stringLocal.set(Thread.currentThread().getName());
	    }  
	  
	    public long getLong() {  
	        return longLocal.get();  
	    }  
	  
	    public String getString() {  
	        return stringLocal.get();  
	    }  
	  
	    public static void main(String[] args) throws InterruptedException {  
	        final MiloTheadLocal test = new MiloTheadLocal();  
	  
	        test.set();  
	        System.out.println(test.getLong());  
	        System.out.println(test.getString());  
	  
	        
	        Thread thread=new Thread() {
	        	public void run() {
	        		test.set();
	        		 System.out.println(test.getLong());  
	        		 System.out.println(test.getString());  
	        	}
	        };
	        thread.start();
	        //thread.join():用来指定当前主线程等待其他线程执行完毕后,再来继续执行Thread.join()后面的代码
	        thread.join();
	        System.out.println(test.getLong());  
	        System.out.println(test.getString());  
	    }  
}
```

运行结果：

```shell
1
main
10
Thread-0
1
main
```

从这段代码的输出结果可以看出，在 main 线程中和 thread1 线程中，longLocal 保存的副本值和 stringLocal 保存的副本值都不一样。最后一次在 main 线程再次打印副本值是为了证明在 main 线程中和 thread1 线程中的副本值确实是不同的。

总结一下：

1. 实际的通过 ThreadLocal 创建的副本是存储在每个线程自己的 threadLocals 中的；

2. 为何 threadLocals 的类型 ThreadLocalMap 的键值为 ThreadLocal 对象，因为每个线程中可有多个 threadLocal 变量，就像上面代码中的 longLocal 和 stringLocal ；

3. 在进行 get 之前，必须先 set ，否则会报空指针异常。如果想在 get 之前不需要调用 set 就能正常访问的话，必须重写 `initialValue()` 方法。

因为在上面的代码分析过程中，我们发现如果没有先 set 的话，即在 map 中查找不到对应的存储，则会通过调用  setInitialValue 方法返回i，而在 setInitialValue 方法中，有一个语句是 `T value = initialValue()` ， 而默认情况下，initialValue 方法返回的是 null 。

```java
/**
 * Returns the value in the current thread's copy of this
 * thread-local variable.  If the variable has no value for the
 * current thread, it is first initialized to the value returned
 * by an invocation of the {@link #initialValue} method.
 *
 * @return the current thread's value of this thread-local
 */
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
/**
 * Variant of set() to establish initialValue. Used instead
 * of set() in case user has overridden the set() method.
 *
 * @return the initial value
 */
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```

 看下面这个例子：

```java
package com.milo.jdk.lang;
/**
 * You need to setting before getting it, otherwise there will be a NullPointerException
 * @author MILO
 *
 */
public class MiloTheadLocal {
	  	ThreadLocal<Long> longLocal = new ThreadLocal<Long>();  
	    ThreadLocal<String> stringLocal = new ThreadLocal<String>();  
	  
	    public void set() {  
	    	longLocal.set(Thread.currentThread().getId());
	    	stringLocal.set(Thread.currentThread().getName());
	    }  
	  
	    public long getLong() {  
	        return longLocal.get();  
	    }  
	  
	    public String getString() {  
	        return stringLocal.get();  
	    }  
	  
	    public static void main(String[] args) throws InterruptedException {  
	        final MiloTheadLocal test = new MiloTheadLocal();  
	  
	        //test.set();  
	        System.out.println(test.getLong());  
	        System.out.println(test.getString());  
	  
	        
	        Thread thread=new Thread() {
	        	public void run() {
	        		test.set();
	        		 System.out.println(test.getLong());  
	        		 System.out.println(test.getString());  
	        	}
	        };
	        thread.start();
	        //thread.join():用来指定当前主线程等待其他线程执行完毕后,再来继续执行Thread.join()后面的代码
	        thread.join();
	        System.out.println(test.getLong());  
	        System.out.println(test.getString());  
	    }  
}
```

运行结果：

```shell
Exception in thread "main" java.lang.NullPointerException
at com.milo.jdk.lang.MiloTheadLocal.getLong(MiloTheadLocal.java:17)
at com.milo.jdk.lang.MiloTheadLocal.main(MiloTheadLocal.java:28)
```

在 main 线程中，没有先 set ，直接 get 的话，运行时会报空指针异常。

但是如果改成下面这段代码，即重写了 initialValue 方法：

```java

package com.milo.jdk.lang;
/**
 * You not need to setting before getting it.
 * @author MILO
 *
 */
public class MiloTheadLocal2 {
	   ThreadLocal<Long> longLocal = new ThreadLocal<Long>() {  
	        protected Long initialValue() {  
	            return Thread.currentThread().getId();  
	        };  
	    };  
	      
	    ThreadLocal<String> stringLocal = new ThreadLocal<String>() {  
	        protected String initialValue() {  
	            return Thread.currentThread().getName();  
	        };  
	    };  
	  
	    public void set() {  
	        longLocal.set(Thread.currentThread().getId());  
	        stringLocal.set(Thread.currentThread().getName());  
	    }  
	  
	    public long getLong() {  
	        return longLocal.get();  
	    }  
	  
	    public String getString() {  
	        return stringLocal.get();  
	    }  
	  
	    public static void main(String[] args) throws InterruptedException {  
	        final MiloTheadLocal2 test = new MiloTheadLocal2();  
	  
	        //test.set();  
	        System.out.println(test.getLong());  
	        System.out.println(test.getString());  
	  
	        Thread thread1 = new Thread() {  
	            public void run() {  
	                //test.set();  
	                System.out.println(test.getLong());  
	                System.out.println(test.getString());  
	            };  
	        };  
	        thread1.start();  
	        thread1.join();  
	  
	        System.out.println(test.getLong());  
	        System.out.println(test.getString());  
	    }  
}
```

运行结果：

```shell
1  
main  
8  
Thread-0  
1  
main  
```


## ThreadLocal 的应用场景

最常见的ThreadLocal使用场景为 用来解决数据库连接、Session管理等。如：

### 数据库连接

```java
private static ThreadLocal<Connection> connectionHolder = new ThreadLocal<Connection>() {  
    public Connection initialValue() {  
        return DriverManager.getConnection(DB_URL);  
    }  
};  

public static Connection getConnection() {  
    return connectionHolder.get();  
}  
```

### Session管理

```java
private static final ThreadLocal threadSession = new ThreadLocal();  

public static Session getSession() throws InfrastructureException {  
    Session s = (Session) threadSession.get();  
    try {  
        if (s == null) {  
            s = getSessionFactory().openSession();  
            threadSession.set(s);  
        }  
    } catch (HibernateException ex) {  
        throw new InfrastructureException(ex);  
    }  
    return s;  
} 
```
