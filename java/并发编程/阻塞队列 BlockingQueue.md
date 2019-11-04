## 介绍

### 阻塞

当阻塞队列为空时，从队列中获取元素的操作将会被阻塞。
当阻塞队列为满时，从队列里添加元素的操作将会被阻塞。

### 用途

在多线程领域：所谓阻塞，在某些情况下会挂起线程（阻塞），一旦条件满足，被挂起的线程又会自动被唤醒。

好处是我们不需要关心什么时候需要阻塞线程，什么时候唤醒线程，因为这一切BlockingQueue都给你一手包办了。

## 分类

 - **ArrayBlockingQueue**：由数组结构组成的有界阻塞队列；

 - **LinkedBlockingQueue**：由链表结构组成的有界（但是默认值大小为 `Integer.MAX_VALUE`）阻塞队列；

 - **PriorityBlockingQueue**：支持优先级排序的无界阻塞队列；

 - **DelayQueue**：使用优先级队列支持延迟的无界阻塞队列；

 - **SynchronousQueue**：不存储元素的队列，也即单个元素的队列。没有容量，每一个 put 操作必须等待一个 take 操作，否则不能添加元素，反之亦然；

 - **LinkedTransferQueue**：由链表结构组成的无界阻塞队列；

 - **LinkedBlockingDeque**：由链表结构组成的双向阻塞队列；

## 核心方法

| 方法类型 | 抛出异常  |  特殊值  |  阻塞  |         超时          |
| :------: | :-------: | :------: | :----: | :-------------------: |
|   插入   |  add(e)   | offer(e) | put(e) | offer(e, timer, unit) |
|   移除   | remove()  |  poll()  | take() |   poll(time, unit)    |
|   检查   | element() |  peek()  |   /    |           /           |

说明：

| 方法类型 | 具体说明                                                     |
| -------- | ------------------------------------------------------------ |
| 抛出异常 | 当阻塞队列满时，add 会抛出 IllegalStateException("Queue full")；<br />当阻塞队列空时，remove 和 element 会抛出 NoSuchElementException() ； |
| 特殊值   | 当阻塞队列满时，offer 返回 false，否则返回 true ；<br />当阻塞队列空时，poll 和 peek 返回 E，否则返回 null； |
| 阻塞     | 线程一直阻塞直到队列不满或空；                               |
| 超时     | 阻塞队列阻塞指定时间后返回                                   |

