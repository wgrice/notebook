## WeakHashMap 定义

从类定义上来看，它和普通的HashMap一样，继承了AbstractMap类和实现了Map接口，也就是说它有着与HashMap差不多的功能。

```java
public class WeakHashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>
```

那么既然jdk已经提供了 HashMap，为什么还要再提供一个 WeakHashMap 呢？ 

## WeakHashMap由来

先来想象一下你因为某种需求需要一个 Cache ，你肯定会面临一个问题，就是所有数据不可能都放到 Cache 里，或者放到 Cache 里性价比太低了。这个时候你可能很快就想到了各种 Cache 数据过期策略，目前也有一些优秀的包提供了功能丰富的 Cache ，比如 Google 的 Guava Cache ，它支持数据定期过期、LRU、LFU 等策略，但它任然有可能会导致有用的数据被淘汰，没用的数据迟迟不淘汰（如果策略使用得当的情况下这都是小概率事件）。

如果我现在说有种机制，可以让你 Cache 里不用的 key 数据自动清理掉，用的还留着，没有误杀也没有漏杀你信不信！没错 WeakHashMap 就是能实现这种功能的东西，这也是它和普通的 HashMap 不同的地方——它有自清理的机制。

## WeakHashMap 实现

如果让你实现一种自清理的 HashMap ，你怎么做？ 我的做法肯定是想办法先知道某个 Key 肯定没有在用了，然后清理到 HashMap 中对应的 K-V 。在 JVM 里一个对象没用了是指没有任何其他有用对象直接或者间接执行它，具体点就是在 GC 过程中它是 GCRoots 不可达的。 JVM 提供了一种机制能让我们感知到一个对象是否已经变成了垃圾对象，这就是 WeakReference。

某个 WeakReference 对象所指向的对象如果被判定为垃圾对象，JVM  会将该 WeakReference 对象放到一个 ReferenceQueue 里，我们只要看下这个 Queue 里的内容就知道某个对象还有没有用了。 WeakHashMap 就是这么做的，所以这里的 Weak 是指 WeakReference 。

## WeakHashMap 源码分析
```java
private static final int DEFAULT_INITIAL_CAPACITY = 16;
private static final int MAXIMUM_CAPACITY = 1 << 30;
private static final float DEFAULT_LOAD_FACTOR = 0.75f;
```


和普通 HashMap 一样，WeakHashMap 也有一些默认值，比如默认容量是 16 ，最大容量 2^30 ，使用超过 75% 它就会自动扩容。

### Entry 节点

```java
private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V> {
        V value;
        final int hash;
        Entry<K,V> next;
        
        Entry(Object key, V value,
              ReferenceQueue<Object> queue,
              int hash, Entry<K,V> next) {
            super(key, queue);
            this.value = value;
            this.hash  = hash;
            this.next  = next;
        }
        /*
        * 其他代码  
        */
}
```

它的 Entry 和普通 HashMap 的 Entry 最大的不同是它继承了 WeakReference ，然后把 **Key 做成了弱引用（注意只有 Key 没有Value）**，然后传入了一个 ReferenceQueue，这就让它能在某个 key 失去所有强引用的时候感知到。

### put 插入
```java
public V put(K key, V value) {
    Object k = maskNull(key);
    int h = hash(k);
    Entry<K,V>[] tab = getTable();
    int i = indexFor(h, tab.length);

    for (Entry<K,V> e = tab[i]; e != null; e = e.next) {
        if (h == e.hash && eq(k, e.get())) {
            V oldValue = e.value;
            if (value != oldValue)
                e.value = value;
            return oldValue;
        }
    }

    modCount++;
    Entry<K,V> e = tab[i];
    tab[i] = new Entry<>(k, value, queue, h, e);
    if (++size >= threshold)
        resize(tab.length * 2);
    return null;
}
```

put 方法也很简单，用 key 的 hashcode 在 tab 中定位，然后判断是否是已经存在的 key，已经存在就替换旧值，否则就新建Entry，遇到多个不同的 key 有同样的 hashCode 就采用开链的方式解决 hash 冲突。注意这里和 HashMap 不太一样的地方，HashMap 会在链表太长的时候对链表做树化，把单链表转换为红黑树，防止极端情况下 hashcode 冲突导致的性能问题，但在WeakHashMap 中没有树化。
　　同样，在 size 大于阈值的时候，WeakHashMap 也对做 resize 的操作，也就是把 tab 扩大一倍。WeakHashMap 中的 resize 比 HashMap 中的 resize 要简单好懂些，但没 HashMap 中的 resize 优雅。WeakHashMap 中 resize 有另外一个额外的操作，就是 `expungeStaleEntries()`，就是对 tab 中的死对象做清理，稍后会详细介绍。

### get 获取

```java
public V get(Object key) {
    Object k = maskNull(key);
    int h = hash(k);
    Entry<K,V>[] tab = getTable();
    int index = indexFor(h, tab.length);
    Entry<K,V> e = tab[index];
    while (e != null) {
        if (e.hash == h && eq(k, e.get()))
            return e.value;
        e = e.next;
    }
    return null;
}
```

get 方法就没什么特别的了，因为 Entry 里存了 hash 值和 key 的值，所以只要用 indexFor 定位到 tab 中的位置，然后遍历一下单链表就知道了。

### expungeStaleEntries 清理

```java
private void expungeStaleEntries() {
    for (Object x; (x = queue.poll()) != null; ) {
        synchronized (queue) {
            @SuppressWarnings("unchecked")
                Entry<K,V> e = (Entry<K,V>) x;
            int i = indexFor(e.hash, table.length);

            Entry<K,V> prev = table[i];
            Entry<K,V> p = prev;
            while (p != null) {
                Entry<K,V> next = p.next;
                if (p == e) {
                    if (prev == e)
                        table[i] = next;
                    else
                        prev.next = next;
                    // Must not null out e.next;
                    // stale entries may be in use by a HashIterator
                    e.value = null; // Help GC
                    size--;
                    break;
                }
                prev = p;
                p = next;
            }
        }
    }
}
```

**expungeStaleEntries 就是 WeakHashMap 的核心了**，它承担着 Map 中死对象的清理工作。**原理就是依赖 WeakReference 和 ReferenceQueue 的特性**。

在每个 WeakHashMap 都有个 ReferenceQueue 的 queue，在 Entry 初始化的时候也会将 queue 传给 WeakReference ，这样当某个可以 key 失去所有强应用之后，其 key 对应的 WeakReference 对象会被放到 queue 里，有了 queue 就知道需要清理哪些 Entry 里。这里也是整个 WeakHashMap 里唯一加了同步的地方。

除了上文说的到 resize 中调用了 expungeStaleEntries() ，size() 中也调用了这个清理方法。另外 getTable()也调了，这就意味着几乎所有其他方法都间接调用了清理。

## WeakHashMap 缺陷
除了上述几个和HashMap不太一样的地方外，WeakHashMap 也提供了其他 HashMap 所有的方法，比如像 remove、clean、putAll、entrySet…… 功能上几乎可以完全替代 HashMap，但 WeakHashMap 也有一些自己的缺陷。

1. **非线程安全**
   关键修改方法没有提供任何同步，多线程环境下肯定会导致数据不一致的情况，所以使用时需要多注意。

2. **单纯作为 Map 没有 HashMap 好**
   HashMap 在 jdk8 做了好多优化，比如单链表在过长时会转化为红黑树，降低极端情况下的操作复杂度。但 WeakHashMap 没有相应的优化，有点像 jdk8 之前的 HashMap 版本。

3. **不能自定义ReferenceQueue**
   WeakHashMap 构造方法中没法指定自定的 ReferenceQueue，如果用户想用 ReferenceQueue 做一些额外的清理工作的话就行不通了。如果即想用 WeakHashMap 的功能，也想用 ReferenceQueue，貌似得自己实现一套新的 WeakHashMap 了。

## WeakHashMap 用途
这里列举几个我所知道的WeakHashMap的使用场景。

1. **阿里Arthas**
   在阿里开源的 Java 诊断工具中使用了 WeakHashMap 做类-字节码的缓存。

```java
 // 类-字节码缓存
 private final static Map<Class<?>/*Class*/, byte[]/*bytes of Class*/> classBytesCache
 	= new WeakHashMap<Class<?>, byte[]>();
```

2. **Cache**
   WeakHashMap 这种自清理的机制，非常适合做缓存了。

3. **ThreadLocalMap**
   ThreadLocal 中用 ThreadLocalMap 存储 Thread 对象，虽然 ThreadLocalMap 和 WeakHashMap 不是一个东西，但ThreadLocalMap 也利用到了 WeakReference 的特性，功能用途很类似，所以我很好奇为什么 ThreadLocalMap 不直接用WeakHashMap呢！