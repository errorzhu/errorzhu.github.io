# 多线程
## 并行与并发的区别
当有多个线程在操作时,如果系统只有一个CPU,则它根本不可能真正同时进行一个以上的线程，它只能把CPU运行时间划分成若干个时间段,再将时间段分配给各个线程执行，在一个时间段的线程代码运行时，其它线程处于挂起状。.这种方式我们称之为并发(Concurrent)。
当系统有一个以上CPU时,则线程的操作有可能非并发。当一个CPU执行一个线程时，另一个CPU可以执行另一个线程，两个线程互不抢占CPU资源，可以同时进行，这种方式我们称之为并行(Parallel)。

## 线程和进程的区别
线程（Thread）是并发编程的基础，也是程序执行的最小单元，它依托进程而存在。一个进程中可以包含多个线程，多线程可以共享一块内存空间和一组系统资源，因此线程之间的切换更加节省资源、更加轻量化，也因此被称为轻量级的进程

## 线程的常用方法
1. join
在一个线程中调用 other.join() ，这时候当前线程会让出执行权给 other 线程，直到 other 线程执行完或者过了超时时间之后再继续执行当前线程
2. yield
yield方法表示给线程调度器一个当前线程愿意出让 CPU 使用权的暗示，但是线程调度器可能会忽略这个暗示

sleep()和yield()方法的区别sleep()和yield()都是Thread类中的静态方法，都会使得当前处于运行状态的线程放弃CPU，但是两者的区别还是有比较大的：1：sleep使当前线程（即调用sleep方法的线程暂停一段时间），给其它的线程运行的机会，而且是不考虑其它线程的优先级的，而且不释放资源锁，也就是说如果有synchronized同步块，其它线程仍然是不能访问共享数据的；yeild只会让位给优先级一样或者比它优先级高的线程，而且不能由用户指定暂停多长时间2：当线程执行了sleep方法之后，线程将转入到睡眠状态，直到时间结束，而执行yield方法，直接转入到就绪状态。这些对线程的生命周期会造成影响的。3：sleep方法需要抛出或者捕获异常，因为线程在睡眠中可能被打断，而yield方法则没异常。

## 线程池
线程池是为了避免线程频繁的创建和销毁带来的性能消耗，而建立的一种池化技术，它是把已创建的线程放入"池"中，当有任务来临时就可以重用已有的线程，无需等待创建的过程，这样就可以有效提高程序的响应速度

## ThreadPoolExecutor 
corePoolSize 表示线程池的常驻核心线程数。如果设置为 0，则表示在没有任何任务时，销毁线程池；如果大于 0，即使没有任务时也会保证线程池的线程数量等于此值。但需要注意，此值如果设置的比较小，则会频繁的创建和销毁线程（创建和销毁的原因会在本课时的下半部分讲到）；如果设置的比较大，则会浪费系统资源，所以开发者需要根据自己的实际业务来调整此值

maximumPoolSize 表示线程池在任务最多时，最大可以创建的线程数。官方规定此值必须大于 0，也必须大于等于 corePoolSize，此值只有在任务比较多，且不能存放在任务队列时，才会用到

keepAliveTime 表示线程的存活时间，当线程池空闲时并且超过了此时间，多余的线程就会销毁，直到线程池中的线程数量销毁的等于 corePoolSize 为止，如果 maximumPoolSize 等于 corePoolSize，那么线程池在空闲的时候也不会销毁任何线程

workQueue 表示线程池执行的任务队列，当线程池的所有线程都在处理任务时，如果来了新任务就会缓存到此任务队列中排队等待执行

RejectedExecutionHandler 表示指定线程池的拒绝策略，当线程池的任务已经在缓存队列 workQueue 中存储满了之后，并且不能创建新的线程来执行此任务时，就会用到此拒绝策略，它属于一种限流保护的机制


## 线程池关闭的方式

shutdown()
它可以安全地关闭一个线程池，调用 shutdown() 方法之后线程池并不是立刻就被关闭，因为这时线程池中可能还有很多任务正在被执行，或是任务队列中有大量正在等待被执行的任务，调用 shutdown() 方法后线程池会在执行完正在执行的任务和队列中等待的任务后才彻底关闭

shutdownNow()
在执行 shutdownNow 方法之后，首先会给所有线程池中的线程发送 interrupt 中断信号，尝试中断这些任务的执行，然后会将任务队列中正在等待的所有任务转移到一个 List 中并返回，我们可以根据返回的任务 List 来进行一些补救的操作。



## CPU 核心数和线程数设置
计算密集型: 线程数 = cpu*2
io密集型: 线程数 = cpu*(1+ 平均等待时间/平均执行时间)

