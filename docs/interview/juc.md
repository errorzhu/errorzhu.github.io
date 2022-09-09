# java并发容器

## CopyOnWriteArrayList
CopyOnWrite 的意思是说，当容器需要被修改的时候，不直接修改当前容器，而是先将当前容器进行 Copy，复制出一个新的容器，然后修改新的容器，完成修改之后，再将原容器的引用指向新的容器。这样就完成了整个修改过程,适合读多写少的场景。
缺点:
	内存占用问题
	在元素较多或者复杂的情况下，复制的开销很大
	数据一致性问题


## ConcurrentHashMap 和 ConcurrentSkipListMap

ConcurrentHashMap 使用数组 + 链表 + 红黑树构成,采用 Node + CAS + synchronized 保证线程安全
ConcurrentSkipListMap 使用跳表实现的map,使map有序

## CountDownLatch、CyclicBarrier 和 Semaphore 三个类的使用和原理