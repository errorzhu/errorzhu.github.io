# linux下的java锁问题

## 一  目的

我们想使用一个文件实现一个简单的消息队列，多进程之间通信使用。多进程同时操作一个文件会冲突，自然的就想到使用文件锁。



## 二 初探

```
FileChannel channel = new RandomAccessFile(file, "rw").getChannel();
try(FileLock lock = channel.tryLock()) {
if (lock == null) {
// do something
} else {
}
}
```
一开始我们实现了如上的代码，但发现这样锁并没有效果



## 三 原因

Linux文件锁的类型有两种：

- 1.mandatory 强制锁，加上后第三方程序对加过锁的文件修改会报错：文件被占用、文件无法被修改等。这个是排它的，一般有这种需求的都想要这种锁。
- 2.advisory 建议锁/劝告锁，加上后第三方程序(gedite、vim/vi等、touch、rm等)可以直接修改删除。这个是 **建议** 的，也就是说它确实提供了加锁和检测是否有锁的手段，但假设你的第三方程序根本不检测文件有没有锁就直接修改/删除了，它也不 **排斥**，所以只对那些修改前try一下的**守规矩**程序有效。

```
cat /proc/locks
```
通过上述命令，发现当前linux 是 advisory 锁

## 四 改进

1. 强制使用 mandatory 锁
```
挂载磁盘时指定
mount -o mand /dev/sdb7 /mnt

```

2. 创建锁文件或锁目录
```
    public static void main(String[] args) throws InterruptedException {
        while (true){
            File lockFile = new File("/tmp/lock");
            boolean hasLock = lockFile.mkdir();
            while(!hasLock){
                hasLock = lockFile.mkdir();
            }
            //dosomething
            lockFile.delete();
        }

    }
```
缺点是程序异常退出，文件便无法及时清除

3. 创建带pid的锁文件

     对上一种方案进行优化，将锁文件名命名成pid，获取锁文件之前判断pid是否存在，如果不存在，证明锁文件持有的程序之前异常退出了，可以先删除。

4. 使用外部消息队列

       redis
       kafka

## 五 参考资料
[https://docs.oracle.com/javase/8/docs/api/java/nio/channels/FileLock.html](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/FileLock.html)