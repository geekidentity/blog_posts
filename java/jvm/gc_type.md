---
categories: JVM

tags: 
  - Java
  - JVM

title: GC回收器类型

date: 2016-12-23
---

### SerialGC 串行回收器
这是最古老同时也是最稳定的回收回收器，使用该回收器时，新生代老年代都使用串行回收器。新生代使用复制算法，老年代使用标记-压缩算法。
使用：

```
-XX+UseSerailGC
```


# ParNewGC
该回收器是SerialGC的并行版本，在新生代中将进行并发回收。
使用：

```
-XX+UseParNewGC
```

# ParallelGC
该回收器是新生代并行，老年代串行。
使用：

```
-XX+UseParallelGC
```
还有一个ParallelOldGC，该回收器在老年代也使用并行回收，同时更加关注吞吐量。

# CMS
Concurrent Mark Sweep 并发标记清除。这是一个老年代回收器，使用此回收器时新生代将默认使用ParNewGC，
使用：

```
-XX:+UseConcMarkSweepGC
```
该回收器GC的过程：

1：初始标记（非并发）

    这个过程将标记根可以直接关联的对象，速度比较快。
2：并发标记（并发）
    
    标记全部对象
    
3：重新标记（非并发）
    
4：并发清除    

JVM的Stop the World

进行GC时会出现全局停顿，所有Java代码停止，但native代码可以执行，但不能和JVM交互

Stop the World多半是由GC引起的，还有Dump线程，死锁检查，堆Dump等。