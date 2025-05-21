# oozie调度任务随机卡住的排查

## 背景

我们使用cdh集群的oozie服务来调度工作流,周期的跑定时任务，突然有一天现场同事反馈,工作流中的任务一直处在running状态。

## 现象及分析

让现场同事手动触发过几次工作流，有时会卡住有时不会卡住,如果卡住,在yarn上可以查询到一直running的任务是oozie laucher的任务;而且工作流中的每个节点被卡住也是随机的

## 排查

1. 首先排查了oozie 日志，对应任务的yarn日志，hue日志，均没有异常
2. 排查了oozie server的配置，一开始发现配置的内存很小，调整到了2g,没有解决
3. 重新导入了工作流,重启过oozie,hue,resource manager均无效
4. 进行了深入排查
   
   ```
   1 首先通过yarn定位到卡住任务的app id,以及driver启动到哪个节点
   2 在对应节点使用ps -ef |grep ${app id} ,查询到对应进程
   3 使用jstack 查询对应java 进程的进程栈如下
   ```

```
Attaching to process ID 245734, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.181-b13
Deadlock Detection:

No deadlocks found.

Thread 15708: (state = BLOCKED)
 - sun.misc.Unsafe.park(boolean, long) @bci=0 (Interpreted frame)
 - java.util.concurrent.locks.LockSupport.parkNanos(java.lang.Object, long) @bci=20, line=215 (Interpreted frame)
 - java.util.concurrent.SynchronousQueue$TransferStack.awaitFulfill(java.util.concurrent.SynchronousQueue$TransferStack$SNode, boolean, long) @bci=160, line=460 (Interpreted frame)
 - java.util.concurrent.SynchronousQueue$TransferStack.transfer(java.lang.Object, boolean, long) @bci=102, line=362 (Interpreted frame)
 - java.util.concurrent.SynchronousQueue.poll(long, java.util.concurrent.TimeUnit) @bci=11, line=941 (Interpreted frame)
 - java.util.concurrent.ThreadPoolExecutor.getTask() @bci=134, line=1073 (Interpreted frame)
 - java.util.concurrent.ThreadPoolExecutor.runWorker(java.util.concurrent.ThreadPoolExecutor$Worker) @bci=26, line=1134 (Interpreted frame)
 - java.util.concurrent.ThreadPoolExecutor$Worker.run() @bci=5, line=624 (Interpreted frame)
 - java.lang.Thread.run() @bci=11, line=748 (Interpreted frame)

Locked ownable synchronizers:
    - None

Thread 245781: (state = BLOCKED)
 - java.lang.Object.wait(long) @bci=0 (Interpreted frame)
 - java.lang.ref.ReferenceQueue.remove(long) @bci=59, line=144 (Interpreted frame)
 - java.lang.ref.ReferenceQueue.remove() @bci=2, line=165 (Interpreted frame)
 - org.apache.hadoop.fs.FileSystem$Statistics$StatisticsDataReferenceCleaner.run() @bci=9, line=3696 (Interpreted frame)
 - java.lang.Thread.run() @bci=11, line=748 (Interpreted frame)

Locked ownable synchronizers:
    - None

Thread 245780: (state = BLOCKED)
 - java.lang.Thread.sleep(long) @bci=0 (Interpreted frame)
 - org.apache.hadoop.yarn.client.api.async.impl.AMRMClientAsyncImpl$HeartbeatThread.run() @bci=144, line=289 (Interpreted frame)

Locked ownable synchronizers:
    - None

Thread 245777: (state = BLOCKED)
 - sun.misc.Unsafe.park(boolean, long) @bci=0 (Interpreted frame)
 - java.util.concurrent.locks.LockSupport.park(java.lang.Object) @bci=14, line=175 (Interpreted frame)
 - java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await() @bci=42, line=2039 (Interpreted frame)
 - java.util.concurrent.LinkedBlockingQueue.take() @bci=29, line=442 (Interpreted frame)
 - org.apache.hadoop.yarn.client.api.async.impl.AMRMClientAsyncImpl$CallbackHandlerThread.run() @bci=18, line=310 (Interpreted frame)

Locked ownable synchronizers:
    - None

Thread 245752: (state = BLOCKED)

Locked ownable synchronizers:
    - None

Thread 245751: (state = BLOCKED)
 - java.lang.Object.wait(long) @bci=0 (Interpreted frame)
 - java.lang.ref.ReferenceQueue.remove(long) @bci=59, line=144 (Compiled frame)
 - java.lang.ref.ReferenceQueue.remove() @bci=2, line=165 (Compiled frame)
 - java.lang.ref.Finalizer$FinalizerThread.run() @bci=36, line=216 (Interpreted frame)

Locked ownable synchronizers:
    - None

Thread 245750: (state = BLOCKED)
 - java.lang.Object.wait(long) @bci=0 (Interpreted frame)
 - java.lang.Object.wait() @bci=2, line=502 (Compiled frame)
 - java.lang.ref.Reference.tryHandlePending(boolean) @bci=54, line=191 (Compiled frame)
 - java.lang.ref.Reference$ReferenceHandler.run() @bci=1, line=153 (Interpreted frame)

Locked ownable synchronizers:
    - None

Thread 245735: (state = IN_NATIVE)
 - sun.nio.ch.EPollArrayWrapper.epollWait(long, int, long, int) @bci=0 (Compiled frame; information may be imprecise)
 - sun.nio.ch.EPollArrayWrapper.poll(long) @bci=18, line=269 (Compiled frame)
 - sun.nio.ch.EPollSelectorImpl.doSelect(long) @bci=28, line=93 (Compiled frame)
 - sun.nio.ch.SelectorImpl.lockAndDoSelect(long) @bci=37, line=86 (Compiled frame)
 - sun.nio.ch.SelectorImpl.select(long) @bci=30, line=97 (Compiled frame)
 - org.apache.hadoop.net.SocketIOWithTimeout$SelectorPool.select(java.nio.channels.SelectableChannel, int, long) @bci=46, line=335 (Compiled frame)
 - org.apache.hadoop.net.SocketIOWithTimeout.connect(java.nio.channels.SocketChannel, java.net.SocketAddress, int) @bci=69, line=203 (Interpreted frame)
 - org.apache.hadoop.net.NetUtils.connect(java.net.Socket, java.net.SocketAddress, java.net.SocketAddress, int) @bci=91, line=531 (Interpreted frame)
 - org.apache.hadoop.ipc.Client$Connection.setupConnection(org.apache.hadoop.security.UserGroupInformation) @bci=194, line=686 (Interpreted frame)
 - org.apache.hadoop.ipc.Client$Connection.setupIOstreams(java.util.concurrent.atomic.AtomicBoolean) @bci=126, line=789 (Interpreted frame)
 - org.apache.hadoop.ipc.Client$Connection.access$3600(org.apache.hadoop.ipc.Client$Connection, java.util.concurrent.atomic.AtomicBoolean) @bci=2, line=410 (Interpreted frame)
 - org.apache.hadoop.ipc.Client.getConnection(org.apache.hadoop.ipc.Client$ConnectionId, org.apache.hadoop.ipc.Client$Call, int, java.util.concurrent.atomic.AtomicBoolean) @bci=110, line=1560 (Interpreted frame)
 - org.apache.hadoop.ipc.Client.call(org.apache.hadoop.ipc.RPC$RpcKind, org.apache.hadoop.io.Writable, org.apache.hadoop.ipc.Client$ConnectionId, int, java.util.concurrent.atomic.AtomicBoolean) @bci=16, line=1391 (Interpreted frame)
 - org.apache.hadoop.ipc.Client.call(org.apache.hadoop.ipc.RPC$RpcKind, org.apache.hadoop.io.Writable, org.apache.hadoop.ipc.Client$ConnectionId, java.util.concurrent.atomic.AtomicBoolean) @bci=7, line=1355 (Interpreted frame)
 - org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(java.lang.Object, java.lang.reflect.Method, java.lang.Object[]) @bci=255, line=228 (Interpreted frame)
 - org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(java.lang.Object, java.lang.reflect.Method, java.lang.Object[]) @bci=4, line=116 (Interpreted frame)
 - com.sun.proxy.$Proxy14.getApplications(com.google.protobuf.RpcController, org.apache.hadoop.yarn.proto.YarnServiceProtos$GetApplicationsRequestProto) @bci=20 (Interpreted frame)
 - org.apache.hadoop.yarn.api.impl.pb.client.ApplicationClientProtocolPBClientImpl.getApplications(org.apache.hadoop.yarn.api.protocolrecords.GetApplicationsRequest) @bci=18, line=296 (Interpreted frame)
 - sun.reflect.NativeMethodAccessorImpl.invoke0(java.lang.reflect.Method, java.lang.Object, java.lang.Object[]) @bci=0 (Interpreted frame)
 - sun.reflect.NativeMethodAccessorImpl.invoke(java.lang.Object, java.lang.Object[]) @bci=100, line=62 (Interpreted frame)
 - sun.reflect.DelegatingMethodAccessorImpl.invoke(java.lang.Object, java.lang.Object[]) @bci=6, line=43 (Interpreted frame)
 - java.lang.reflect.Method.invoke(java.lang.Object, java.lang.Object[]) @bci=56, line=498 (Interpreted frame)
 - org.apache.hadoop.io.retry.RetryInvocationHandler.invokeMethod(java.lang.reflect.Method, java.lang.Object[]) @bci=21, line=422 (Interpreted frame)
 - org.apache.hadoop.io.retry.RetryInvocationHandler$Call.invokeMethod() @bci=40, line=165 (Interpreted frame)
 - org.apache.hadoop.io.retry.RetryInvocationHandler$Call.invoke() @bci=5, line=157 (Interpreted frame)
 - org.apache.hadoop.io.retry.RetryInvocationHandler$Call.invokeOnce() @bci=21, line=95 (Interpreted frame)
 - org.apache.hadoop.io.retry.RetryInvocationHandler.invoke(java.lang.Object, java.lang.reflect.Method, java.lang.Object[]) @bci=41, line=359 (Interpreted frame)
 - com.sun.proxy.$Proxy15.getApplications(org.apache.hadoop.yarn.api.protocolrecords.GetApplicationsRequest) @bci=16 (Interpreted frame)
 - org.apache.oozie.action.hadoop.LauncherMain.getChildYarnApplications(org.apache.hadoop.conf.Configuration, org.apache.hadoop.yarn.api.protocolrecords.ApplicationsRequestScope, long) @bci=137, line=255 (Interpreted frame)
 - org.apache.oozie.action.hadoop.LauncherMain.getChildYarnJobs(org.apache.hadoop.conf.Configuration, org.apache.hadoop.yarn.api.protocolrecords.ApplicationsRequestScope, long) @bci=12, line=203 (Interpreted frame)
 - org.apache.oozie.action.hadoop.LauncherMain.getChildYarnJobs(org.apache.hadoop.conf.Configuration, org.apache.hadoop.yarn.api.protocolrecords.ApplicationsRequestScope) @bci=51, line=226 (Interpreted frame)
 - org.apache.oozie.action.hadoop.LauncherMain.getChildYarnJobs(org.apache.hadoop.conf.Configuration) @bci=4, line=197 (Interpreted frame)
 - org.apache.oozie.action.hadoop.LauncherMain.killChildYarnJobs(org.apache.hadoop.conf.Configuration) @bci=1, line=264 (Interpreted frame)
 - org.apache.oozie.action.hadoop.SparkMain.run(java.lang.String[]) @bci=14, line=69 (Interpreted frame)
 - org.apache.oozie.action.hadoop.LauncherMain.run(java.lang.Class, java.lang.String[]) @bci=18, line=104 (Interpreted frame)
 - org.apache.oozie.action.hadoop.SparkMain.main(java.lang.String[]) @bci=3, line=60 (Interpreted frame)
 - sun.reflect.NativeMethodAccessorImpl.invoke0(java.lang.reflect.Method, java.lang.Object, java.lang.Object[]) @bci=0 (Interpreted frame)
 - sun.reflect.NativeMethodAccessorImpl.invoke(java.lang.Object, java.lang.Object[]) @bci=100, line=62 (Interpreted frame)
 - sun.reflect.DelegatingMethodAccessorImpl.invoke(java.lang.Object, java.lang.Object[]) @bci=6, line=43 (Interpreted frame)
 - java.lang.reflect.Method.invoke(java.lang.Object, java.lang.Object[]) @bci=56, line=498 (Interpreted frame)
 - org.apache.oozie.action.hadoop.LauncherAM.runActionMain(org.apache.oozie.action.hadoop.ErrorHolder) @bci=100, line=410 (Interpreted frame)
 - org.apache.oozie.action.hadoop.LauncherAM.access$300(org.apache.oozie.action.hadoop.LauncherAM, org.apache.oozie.action.hadoop.ErrorHolder) @bci=2, line=55 (Interpreted frame)
 - org.apache.oozie.action.hadoop.LauncherAM$2.run() @bci=33, line=223 (Interpreted frame)
 - java.security.AccessController.doPrivileged(java.security.PrivilegedExceptionAction, java.security.AccessControlContext) @bci=0 (Compiled frame)
 - javax.security.auth.Subject.doAs(javax.security.auth.Subject, java.security.PrivilegedExceptionAction) @bci=42, line=422 (Interpreted frame)
 - org.apache.hadoop.security.UserGroupInformation.doAs(java.security.PrivilegedExceptionAction) @bci=14, line=1875 (Interpreted frame)
 - org.apache.oozie.action.hadoop.LauncherAM.run() @bci=82, line=217 (Interpreted frame)
 - org.apache.oozie.action.hadoop.LauncherAM$1.run() @bci=86, line=153 (Interpreted frame)
 - java.security.AccessController.doPrivileged(java.security.PrivilegedExceptionAction, java.security.AccessControlContext) @bci=0 (Compiled frame)
 - javax.security.auth.Subject.doAs(javax.security.auth.Subject, java.security.PrivilegedExceptionAction) @bci=42, line=422 (Interpreted frame)
 - org.apache.hadoop.security.UserGroupInformation.doAs(java.security.PrivilegedExceptionAction) @bci=14, line=1875 (Interpreted frame)
 - org.apache.oozie.action.hadoop.LauncherAM.main(java.lang.String[]) @bci=44, line=141 (Interpreted frame)

Locked ownable synchronizers:
    - None
```

---

```
5 通过ai快速分析jstack，发现进程处在sun.nio.ch.EPollArrayWrapper.epollWait这边,推测可能是和rm通信异常，导致oozie laucher并未调度起实际的作业
6 使用iptable 检查防火墙策略，发现一些drop 策略,推测oozie laucher 使用随机端口和rm通信被拒
7 使用iptables -I INPUT 1 -s node1 -j ACCEPT 添加集群内流量默认接收的策略
```

目前调度任务正常,没有卡住
