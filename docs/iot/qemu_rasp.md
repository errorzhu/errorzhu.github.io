# 使用qemu在windows上创建arm的树莓派系统

## 准备
1. 安装qemu
2. 下载树莓派镜像 http://downloads.raspberrypi.org/raspbian/images/raspbian-2020-02-14/2020-02-13-raspbian-buster.zip
3. 下载内核文件(https://github.com/dhruvvyas90/qemu-rpi-kernel)
    versatile-pb-buster-5.4.51.dtb
    kernel-qemu-5.4.51-buster
## 启动

```
qemu-system-arm -M versatilepb -cpu arm1176 -m 256 -drive "file=2020-02-13-raspbian-buster.img,if=none,index=0,media=disk,format=raw,id=disk0" -device "virtio-blk-pci,drive=disk0,disable-modern=on,disable-legacy=off" -net "user,hostfwd=tcp::5022-:22" -dtb versatile-pb-buster-5.4.51.dtb -kernel kernel-qemu-5.4.51-buster -append "root=/dev/vda2 panic=1" -no-reboot -net nic -serial stdio

```