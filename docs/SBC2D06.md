# SBC2D06使用自定义rootfs

## 编译uboot 和 kernel
准备环境
```
tar -xvf gcc-arm-8.2-2018.08-x86_64-arm-linux-gnueabihf.tar.gz -C ./
tar -jxvf boot.tar.bz2 -C ./
tar -jxvf  kernel.tar.bz2 -C ./
tar -jxvf project.tar.bz2  -C ./
tar -jxvf sdk.tar.bz2  -C ./

```

```
sudo dpkg-reconfigure dash
```

```
# sudo apt-get  install libncurses5-dev libncursesw5-dev
# sudo apt-get  install lib32z1
# sudo apt-get  install lsb-core
# sudo apt-get install libc6-dev-i386
# sudo apt-get install lib32z1 
# sudo apt-get install libuuid1:i386
# sudo apt-get install cmake
# sudo apt install bc
# sudo apt-get install xz-utils
# sudo apt-get install automake
# sudo apt-get install libtool
# sudo apt-get install libevdev-dev
# sudo apt-get install pkg-config

sudo apt-get install openssh-server
sudo apt-get install xz-utils
sudo apt-get install python
sudo apt-get install git
sudo apt-get install make
sudo apt-get install gcc
sudo apt-get install g++
```
配置内核
```
cd kernel
ARCH=arm make menuconfig
cp .config arch/arm/configs/infinity2m_spinand_ssc011a_s01a_minigui_doublenet_defconfig -f
```

开始编译
```
export PATH=/home/bbelephant/work/ssd20x/gcc-arm-8.2-2018.08-x86_64-arm-linux-gnueabihf/bin:$PATH
sudo chown root:root -R ./*
./Release_to_customer.sh -f nand -p ssd202 -o 2D06
cp .config ./arch/arm/configs/infinity2m_spinand_ssc011a_s01a_minigui_doublenet_defconfig -f
```

# sd卡烧录

```
# cd project
# ./make_sd_upgrade_sigmastar.sh
project/image/output/images/SigmastarUpgradeSD.bin /sdcard
```
uboot执行
```
# setenv UpgradePort 1
# saveenv
# sdstar
```

## 制作rootfs
在ubuntu虚拟机执行
挂载脚本
```
#!/bin/bash
mnt ()
{
    echo "MOUNTING"
    sudo mount -t proc /proc ${2}proc
    sudo mount -t sysfs /sys ${2}sys
    sudo mount -o bind /dev ${2}dev
    sudo mount -o bind /dev/pts ${2}dev/pts
    sudo chroot ${2}
}
umnt ()
{
    echo "UNMOUNTING"
    sudo umount ${2}proc
    sudo umount ${2}sys
    sudo umount ${2}dev/pts
    sudo umount ${2}dev
}

if [ "$1" = "-m" ] && [ -n "$2" ];
then
    mnt $1 $2
    echo "mnt -m pwd"
elif [ "$1" = "-u" ] && [ -n "$2" ];
then
    umnt $1 $2
    echo "mnt -u pwd"
else
    echo ""
    echo "Either 1'st, 2'nd or bothparameters were missing"
    echo ""
    echo "1'st parameter can be one ofthese: -m(mount) OR -u(umount)"
    echo "2'nd parameter is the full pathof rootfs directory(with trailing '/')"
    echo ""
    echo "For example: ch-mount -m/media/sdcard/"
    echo ""
    echo 1st parameter : ${1}
    echo 2nd parameter : ${2}
fi
```


```

sudo apt-get install qemu-user-static
sudo cp /usr/bin/qemu-arm-static /home/xxxx/ssd20x/ubuntu_base/usr/bin/
#chmod 777  ssd20x/ubuntu_base/tmp
sudo ./ms.sh -m home/xxxx/ssd20x/ubuntu_base/
```

```
apt update
apt install usbutils
apt install sudo 
apt install language-pack-en-base
apt install ssh
apt install net-tools
apt install ethtool
apt install ifupdown
apt install iputils-ping
apt install rsyslog
apt install htop
apt install vi
apt install dhcpcd5
apt install samba samba-common
apt install wpasupplicant
apt install jq
apt install alsa-base 
apt install minicom
```

```
passwd root
```

```
echo "industio" > /etc/hostname
echo "127.0.0.1 localhost" >> /etc/hosts
echo "127.0.1.1 industio" >> /etc/hosts
```

```
vi /lib/systemd/system/getty@.service
修改为ttyS0
cp /lib/systemd/system/serial-getty@.service /lib/systemd/system/serial-getty@ttyS2.service
ln -s /lib/systemd/system/serial-getty@ttyS2.service /etc/systemd/system/getty.target.wants/
再修改/lib/systemd/system/serial-getty@ttyS2.service把里面的“%i.device”改为“%i”
```

```
exit
sudo ./ms.sh -u home/xxxx/ssd20x/ubuntu_base/
```
烧写rootfs至sd卡
```
fdisk /dev/sdb
sudo mkfs.ext3 /dev/sdb1
mount /dev/sdb1 /opt/tmp_fs
cp -rf /xxxx/ssd20x/ubuntu_base/*  /opt/tmp_fs
umount /opt/tmp_fs

```
sd卡插入开发板修改uboot

```
setenv bootargs console=ttyS0,115200 root=/dev/mmcblk1p1 rw rootwait=1 LX_MEM=0x7f00000 mma_heap=mma_heap_name0,miu=0,sz=0xa00000 mma_memblock_remove=1 highres=off mmap_reserved=fb,miu=0,sz=0x300000,max_start_off=0x3300000,max_end_off=0x3600000 mtdparts=nand0:384k@1280k(IPL0),384k(IPL1),384k(IPL_CUST0),384k(IPL_CUST1),768k(UBOOT0),768k(UBOOT1),256k(ENV),256k(ENV1),0x20000(KEY_CUST),0x60000(LOGO),0x500000(KERNEL),0x500000(RECOVERY),-(UBI)

saveenv
reset
```

## 参考
http://doc.industio.com/docs/ido-sbc2d06/ido-sbc2d06-1ds5ej9vk8mj6