## java.lang.StackOverFlowError

StackOverflowError 是一个java中常出现的错误：在jvm运行时的数据区域中有一个java虚拟机栈，当执行java方法时会进行压栈弹栈的操作。在栈中会保存局部变量，操作数栈，方法出口等等。jvm规定了栈的最大深度，当执行时栈的深度大于了规定的深度，就会抛出StackOverflowError错误。

```java
public static void main(String[] args) {
    test();
}

private static void test()
{
    test();
}

// Exception in thread "main" java.lang.StackOverflowError
```

## java.lang.OutOfMemoryError: Java heap space

堆内存不足导致内存溢出。

```java
public static void main(String[] args) {
    // 创建 6m 内存， 虚拟机参数 -Xms5m -Xmx5m
    byte[] buf = new byte[1024*1024*6];
    System.out.println("#####");
}

// Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
```

## java.lang.OutOfMemoryError:  GC overhead limit exceeded

这个是 JDK6 新添的错误类型。是发生在 GC 占用大量时间为释放很小空间的时候发生的，是一种保护机制。一般是因为堆太小，导致异常的原因：没有足够的内存。 

Sun 官方对此的定义：超过 98% 的时间用来做GC并且回收了不到 2% 的堆内存时会抛出此异常。

```java
public static void main(String[] args) {
    // 虚拟机参数 -Xms10m -Xmx10m -XX:+PrintGCDetails
    int i = 0;
    List<String> list = new ArrayList<>();
    while(true) {
    	list.add(String.valueOf(++i).intern());
    }
}

// ... 大量 Full GC
// Exception in thread "main" java.lang.OutOfMemoryError: GC overhead limit exceeded
```

## java.lang.OutOfMemoryError:  Direct buffer memory

创建Buffer对象时，可以选择从JVM堆中分配内存，也可以OS本地内存中分配，由于本地缓冲区避免了缓冲区复制，在性能上相对堆缓冲区有一定优势，但同时也存在一些弊端。

两种缓冲区对应的API如下：

- JVM堆缓冲区：ByteBuffer.allocate(size)
- 本地缓冲区：ByteBuffer.allocateDirect(size)

从堆中分配的缓冲区为普通的Java对象，生命周期与普通的Java对象一样，当不再被引用 时，Buffer对象会被回收。而直接缓冲区（DirectBuffer）为本地内存，并不在Java堆中，也不能被JVM垃圾回收。由于直接缓冲区在 JVM里被包装进Java对象DirectByteBuffer中，当它的包装类被垃圾回收时，会调用相应的JNI方法释放本地内存，所以本地内存的释放 也依赖于JVM中DirectByteBuffer对象的回收。

```java
public static void main(String[] args) {
    // 虚拟机参数 -Xms10m -Xmx10m -XX:+PrintGCDetails -XX:MaxDirectMemorySize=5m
    System.out.println("配置的 maxDirectMemory:" +
    (sun.misc.VM.maxDirectMemory()/1024/1024) + "MB");

    ByteBuffer byteBuffer = ByteBuffer.allocateDirect(1024 * 1024 * 6);
}

// 配置的 maxDirectMemory:5MB
// [GC (System.gc()) [PSYoungGen: 740K->504K(2560K)] 937K->740K(9728K), 0.0017495 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
// [Full GC (System.gc()) [PSYoungGen: 504K->0K(2560K)] [ParOldGen: 236K->623K(7168K)] 740K->623K(9728K), [Metaspace: 3284K->3284K(1056768K)], 0.0067875 secs] [Times: user=0.02 sys=0.02, real=0.01 secs] 
// Exception in thread "main" java.lang.OutOfMemoryError: Direct buffer memory
```

## java.lang.OutOfMemoryError:  Unable to create new native thread

 linux 系统中单个进程的最大线程数有其最大的限制 PTHREAD_THREADS_MAX，这个限制可以在 /usr/include/bits/local_lim.h 中查看，对linuxthreads 这个值一般是 1024。当应用创建的线程数超过这个限制就会报错。

```java
// 通常 linux 系统有限制，windows上受进程内存限制
public static void main(String[] args) {
    for (int i = 0; ; i++) {
        new Thread(() -> {
            try { Thread.sleep(Integer.MAX_VALUE); } catch (InterruptedException e) { e.printStackTrace(); }
        });
    }
}

// Exception in thread "main" java.lang.OutOfMemoryError: unable to create new native thread
```

## java.lang.OutOfMemoryError: Metaspace

在Java8中,将之前 PermGen 中的所有内容, 都移到了 Metaspace 空间。例如: class 名称, 字段, 方法, 字节码, 常量池, JIT优化代码, 等等。

Metaspace 的使用量与JVM加载到内存中的 class 数量/大小有关。可以说, **java.lang.OutOfMemoryError: Metaspace 错误的主要原因, 是加载到内存中的 class 数量太多或者体积太大**。

```java
public class OOMMetaspaceDemo {
    public static void main(String[] args) {
        int i = 0;
        try {
            while (true) {
                Enhancer enhancer = new Enhancer();
                enhancer.setSuperclass(T.class);
                enhancer.setUseCache(false);
                enhancer.setCallback(new MethodInterceptor() {
                    @Override
                    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                        return methodProxy.invoke(o, args);
                    }
                });
                enhancer.create();
            }
        } catch (Exception e) {
            System.out.println("最终i=" + i);
            e.printStackTrace();
        }
    }

    static class T {
    }
}

// Exception in thread "main" java.lang.OutOfMemoryError: Metaspace
```

