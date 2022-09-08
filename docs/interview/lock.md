# 理解java锁

## 锁的类型

偏向锁/轻量级锁/重量级锁；
	这三种锁特指 synchronized 锁的状态，通过在对象头中的 mark word 来表明锁的状态。
	锁升级的路径：无锁→偏向锁→轻量级锁→重量级锁。
	偏向锁性能最好，可以避免执行 CAS 操作。
	而轻量级锁利用自旋和 CAS 避免了重量级锁带来的线程阻塞和唤醒，性能中等。
	重量级锁则会把获取不到锁的线程阻塞，性能最差。
可重入锁/非可重入锁；
	可重入锁: 线程当前已经持有这把锁了，能在不释放这把锁的情况下，再次获取这把锁。
	不可重入锁: 虽然线程当前持有了这把锁，但是如果想再次获取这把锁，也必须要先释放锁后才能再次尝试获取
共享锁/独占锁；
	共享锁: 同一把锁可以被多个线程同时获得
	独占锁: 锁只能同时被一个线程获得
公平锁/非公平锁；
	公平锁：线程现在拿不到这把锁，那么线程就都会进入等待，开始排队，先来先得。
	非公平锁: 在一定情况下，忽略掉已经在排队的线程，发生插队现象
悲观锁/乐观锁；
	悲观锁: 在获取资源之前，必须先拿到锁，以便达到"独占"的状态，当前线程在操作资源的时候，其他线程由于不能拿到锁，所以其他线程不能来影响我。
	乐观锁: 并不要求在获取资源前拿到锁，也不会锁住资源；相反，乐观锁利用 CAS 理念，在不独占资源的情况下，完成了对资源的修改
自旋锁/非自旋锁；
	自旋锁: 如果线程现在拿不到锁，并不直接陷入阻塞或者释放 CPU 资源，而是开始利用循环，不停地尝试获取锁，这个循环过程被形象地比喻为"自旋"。
	非自旋锁: 没有自旋的过程，如果拿不到锁就直接放弃，或者进行其他的处理逻辑，例如去排队、陷入阻塞等
可中断锁/不可中断锁。
	可中断锁: 一旦线程申请了锁，只能等到拿到锁以后才能进行其他的逻辑处理
	不可中断锁：不想获取锁，那么也可以在中断之后去做其他的事情
	
## synchronized 的 monitor锁
如果是同步代码块
```
public class SynTest {
    public void synBlock() {
        synchronized (this) {
            System.out.println("hello");
        }
    }
}
```

通过反编译查看汇编代码, synchronized 变成汇编的monitorenter monitorexit指令。

monitorenter
	执行 monitorenter 的线程尝试获得 monitor 的所有权，会发生以下这三种情况之一：
	a. 如果该 monitor 的计数为 0，则线程获得该 monitor 并将其计数设置为 1。然后，该线程就是这个 monitor 的所有者。
	b. 如果线程已经拥有了这个 monitor ，则它将重新进入，并且累加计数。
	c. 如果其他线程已经拥有了这个 monitor，那个这个线程就会被阻塞，直到这个 monitor 的计数变成为 0，代表这个 monitor 已经被释放了，于是当前这个线程就会再次尝试获取这个 monitor。
monitorexit
	monitorexit 的作用是将 monitor 的计数器减 1，直到减为 0 为止。代表这个 monitor 已经被释放了，已经没有任何线程拥有它了，也就代表着解锁，所以，其他正在等待这个 monitor 的线程，此时便可以再次尝试获取这个 monitor 的所有权。

如果是同步方法
```
public class SynTest {
    public synchronized void synBlock() {
        System.out.println("hello");
    }
}
```
被 synchronized 修饰的方法会有一个 ACC_SYNCHRONIZED 标志。当某个线程要访问某个方法的时候，会首先检查方法是否有 ACC_SYNCHRONIZED 标志，如果有则需要先获得 monitor 锁，然后才能开始执行方法，方法执行之后再释放 monitor 锁


## synchronized 与 lock
lock是接口,有很多不同功能的锁实现
区别
	synchronized是不可中断锁，ReentrantLock 是可中断锁
	synchronized 锁只能同时被一个线程拥有，但是 Lock 锁没有这个限制
	synchronized不可以设置公平/非公平,ReentrantLock可以

## volatile
	当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值立即刷新回主内中
	当读一个volatile变量时，JMM会把该线程对应的本地内存设置为无效，直接从主内存中读取共享变量


## 问题

1. synchronized 和 ReentrantLock 是如何实现的？它们有什么区别？
synchronized 属于独占式悲观锁，是通过 JVM 隐式实现的，synchronized 只允许同一时刻只有一个线程操作资源。

在 Java 中每个对象都隐式包含一个 monitor（监视器）对象，加锁的过程其实就是竞争 monitor 的过程，当线程进入字节码 monitorenter 指令之后，线程将持有 monitor 对象，执行 monitorexit 时释放 monitor 对象，当其他线程没有拿到 monitor 对象时，则需要阻塞等待获取该对象。

ReentrantLock 是 Lock 的默认实现方式之一，它是基于 AQS（Abstract Queued Synchronizer，队列同步器）实现的，它默认是通过非公平锁实现的，在它的内部有一个 state 的状态字段用于表示锁是否被占用，如果是 0 则表示锁未被占用，此时线程就可以把 state 改为 1，并成功获得锁，而其他未获得锁的线程只能去排队等待获取锁资源。

区别:
synchronized 是 JVM 隐式实现的，而 ReentrantLock 是 Java 语言提供的 API；
ReentrantLock 可设置为公平锁，而 synchronized 却不行；
ReentrantLock 只能修饰代码块，而 synchronized 可以用于修饰方法、修饰代码块等；
ReentrantLock 需要手动加锁和释放锁，如果忘记释放锁，则会造成资源被永久占用，而 synchronized 无需手动释放锁；
ReentrantLock 可以知道是否成功获得了锁，而 synchronized  却不行。

2. 模拟死锁并分析

```
public class DeadLock {
    public static void main(String[] args) {
        Object lock1 = new Object();
        Object lock2 = new Object();
        new Thread(()->{
            synchronized (lock1){
                System.out.println("线程1-获取锁1");
                try {
                    TimeUnit.SECONDS.sleep(3);
                } catch (InterruptedException e) {
                }
                synchronized (lock2){
                    System.out.println("线程1-获取锁2");
                }
            }
        }).start();

        new Thread(()->{
            synchronized (lock2){
                System.out.println("线程2-获取锁2");
                try {
                    TimeUnit.SECONDS.sleep(3);
                } catch (InterruptedException e) {
                }
                synchronized (lock1){
                    System.out.println("线程2-获取锁2");
                }
            }
        }).start();

    }
}
```
两个线程都在持有锁未释放的情况下，想去获取对方的锁
