# 使ssh远程执行后台进程立即退出ssh

## 问题

```
ssh target "nohup /root/deadloop.sh &"
```
使用ssh 远程后台执行一个死循环,并不会立即return


## 原因
SSH 将远程 shell 的 stdin、stdout 和 stderr 连接到本地终端，因此才可以与远程端运行的命令进行交互。
但副作用是它会一直运行直到这些连接被关闭，这只有在远程命令及其所有子命令（！）终止时才会发生。

## 解决

```
ssh target "nohup /root/loop.sh >/dev/null 2>&1 &"
```
使用重定向标准输入输出和错误,从而断开和本地的连接
