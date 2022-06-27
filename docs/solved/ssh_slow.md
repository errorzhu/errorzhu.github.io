# 解决ssh登录异常慢的问题

## 背景
xshell 突然连一台服务异常慢,切抛出异常
```
/usr/bin/xauth: timeout in locking authority file /home/xxx/.Xauthority
```

## 分析
```
strace xauth list
```
```
stat("/home/xxx/.Xauthority-c", 0x7ffff52caac0) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/home/xxx/.Xauthority-c", O_WRONLY|O_CREAT|O_EXCL, 0600) = -1 EACCES (Permission denied)
clock_nanosleep(CLOCK_REALTIME, 0, {tv_sec=2, tv_nsec=0}, 0x7ffff52caa80) = 0
openat(AT_FDCWD, "/home/xxx/.Xauthority-c", O_WRONLY|O_CREAT|O_EXCL, 0600) = -1 EACCES (Permission denied)
clock_nanosleep(CLOCK_REALTIME, 0, {tv_sec=2, tv_nsec=0}, 0x7ffff52caa80) = 0
openat(AT_FDCWD, "/home/xxx/.Xauthority-c", O_WRONLY|O_CREAT|O_EXCL, 0600) = -1 EACCES (Permission denied)

```
```
ls -l /home
发现用户的目录变成root的
chown xxx:xxx /home/xxx
解决

```


## 参考 

https://unix.stackexchange.com/questions/215558/why-am-i-getting-this-message-from-xauth-timeout-in-locking-authority-file-ho