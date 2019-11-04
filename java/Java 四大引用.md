## 概述

Java中对引用的定义有如下四种：强引用，软引用，弱引用，虚引用四种。

![](C:\wg\project\git\notebook\image\java-reference-diagram.png)

## 强引用

Java语言通过引用使得我们可以直接操作堆中的对象，下例中的变量str指向String实例所在的堆空间，通过str我们可以操作该对象

```java
String str = new String("StrongReference");
```

强引用的特点：

1. 可以直接访问目标对象
2. 所指向的对象在任何时候都不会被系统回收，JVM宁愿抛出OOM异常，也不会回收强引用所指向的对象
3. 可能会导致内存泄漏

## 软引用

软引用是除了强引用外最强的引用类型，我们可以通过 java.lang.ref.SoftReference 来使用软引用。SoftReference 实例可以保存对一个 Java 对象的软引用，在不妨碍垃圾收集器对该 Java 对象进行回收的前提下，也就是在该对象被垃圾回收器回收之前，通过 SoftReference 类的 get() 方法可以获得该 Java 对象的强引用，一旦该对象被回收之后，get() 就会返回 null 。

```java
/**jvm参数 ：-XX:+PrintGCDetails -Xmx4m -Xms4m -Xmn4m */
public class ReferenceTest {
    public static void main(String[] args) {
        byte[] byte1 = new byte[1024 * 515];
        SoftReference softReference = new SoftReference(byte1);
        byte1 = null;
        System.gc();
        System.out.println("obj 是否被回收 ？ "+softReference.get());
    }
}

/**
obj 是否被回收 ？ [B@7cd84586
Heap
 PSYoungGen      total 3072K, used 1108K [0x00000000ffc80000, 0x0000000100000000, 0x0000000100000000)
  eden space 2560K, 43% used [0x00000000ffc80000,0x00000000ffd952a8,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
  to   space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
 ParOldGen       total 512K, used 454K [0x00000000ffc00000, 0x00000000ffc80000, 0x00000000ffc80000)
  object space 512K, 88% used [0x00000000ffc00000,0x00000000ffc71898,0x00000000ffc80000)
 Metaspace       used 3231K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 352K, capacity 388K, committed 512K, reserved 1048576K
Java HotSpot(TM) 64-Bit Server VM warning: MaxNewSize (4096k) is equal to or greater than the entire heap (4096k).  A new max generation size of 3584k will be used.
*/
```

这是虽然进行了一次GC但是通过软引用还是能取得对象，接下来我们再申请一块大一点的空间

```java
public class ReferenceTest {
    public static void main(String[] args) {
        byte[] byte1 = new byte[1024 * 515];
        SoftReference softReference = new SoftReference(byte1);
        byte1 = null;
        System.gc();
        byte[] byte2 = new byte[1024 * 2500];
        System.out.println("obj 是否被回收 ？ "+softReference.get());
    }
}

/**
obj 是否被回收 ？ null
[Full GC (Ergonomics) [PSYoungGen: 2560K->0K(3072K)] [ParOldGen: 467K->464K(512K)] 3027K->464K(3584K), [Metaspace: 3313K->3313K(1056768K)], 0.0056548 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
Heap
 PSYoungGen      total 3072K, used 25K [0x00000000ffc80000, 0x0000000100000000, 0x0000000100000000)
  eden space 2560K, 1% used [0x00000000ffc80000,0x00000000ffc867f0,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
  to   space 512K, 85% used [0x00000000fff00000,0x00000000fff6ce98,0x00000000fff80000)
 ParOldGen       total 512K, used 464K [0x00000000ffc00000, 0x00000000ffc80000, 0x00000000ffc80000)
  object space 512K, 90% used [0x00000000ffc00000,0x00000000ffc74198,0x00000000ffc80000)
 Metaspace       used 3319K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 359K, capacity 388K, committed 512K, reserved 1048576K
*/
```

此时，PSYoungGen（年轻代）中的 2560K 空间肯定是不够用的，就会触发一次垃圾回收，当然你也可以显示的再调用一次 gc() ，经过这次 GC 后软引用已经被回收了。

软引用的特点：

1. 一个持软引用的对象，不会立马被垃圾回收器所回收。
2. 当堆内存的使用率接近阈值时，才会去回收软引用的对象。

## 弱引用

弱引用是一种比软引用还要弱的引用类型。弱引用会被 jvm 忽略，也就说在 GC 进行垃圾收集的时候，如果一个对象只有弱引用指向它，那么和没有引用指向它是一样的效果，jvm 都会对它就行果断的销毁，释放内存。其实这个特性是很有用的，jdk 也提供了 java.util.WeakHashMap 这么一个 key 为弱引用的 Map。我们可以通过 java.lang.ref.WeakReference 来保存一个弱引用的实例。

```java
public class ReferenceTest {
    public static void main(String[] args) {
        byte[] byte1 = new byte[1024 * 515];
        WeakReference<byte[]> weakReference = new WeakReference<>(byte1);
        byte1 = null;
        System.out.println("弱引用是否被回收 ： "+weakReference.get());
        System.gc();
        System.out.println("弱引用是否被回收 ： "+weakReference.get());
    }
}

/**
弱引用是否被回收 ： [B@7cd84586
弱引用是否被回收 ： null
*/
```

弱引用的特点：

1. 在系统进行GC的时候，只要发现弱引用都会对其进行回收。

##  虚引用

虚引用是所有引用类型中最弱的一种，一个持有虚引用的对象和几乎没有引用是一样的。

虚引用的作用在于跟踪垃圾回收过程，在对象被收集器回收时收到一个系统通知。 当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在垃圾回收后，将这个虚引用加入引用队列，在其关联的虚引用出队前，不会彻底销毁该对象。 所以可以通过检查引用队列中是否有相应的虚引用来判断对象是否已经被回收了。

```java
public class ReferenceTest {
    public static void main(String[] args) {
        byte[] byte1 = new byte[1024 * 515];
        ReferenceQueue<Object> referenceQueue = new ReferenceQueue<>();
        PhantomReference sf = new PhantomReference<>(byte1,referenceQueue);
        byte1 = null;
        System.out.println("是否被回收"+sf.get());
        System.gc();
        System.out.println("是否被回收"+sf.get());
    }
}

/**
是否被回收null
是否被回收null
*/
```

虚引用的特点：

1. 虚引用必须和引用队列一起使用。
2. **虚引用**并**不会**决定对象的**生命周期**。
3. 虚引用的 get 方法总是会返回 null。
4. 虚引用可以用来做为对象是否存活的监控。

## 引用队列（ReferenceQueue）

效果：引用队列可以配合软引用、弱引用及虚引用使用，当引用的对象将要被JVM回收时，会将其加入到引用队列中。

应用：通过引用队列可以了解JVM垃圾回收情况。

## WeakHashMap