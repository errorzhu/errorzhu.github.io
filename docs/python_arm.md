# python 移植arm开发板
1. 安装ubuntu 虚拟机20.04.4
2. 配置apt源
3. 下载编译工具
```
apt install build-essential
apt install zlib1g
apt-get install zlib1g-dev
```

4. 下载python 源码
5. 宿主机编译python
```
cd Python-3.8.8/
./configure --prefix=/opt/python/python3 --enable-shared
make -j4 && make install
make distclean
```

6. 安装交叉编译工具
```
arm-linux-gnueabihf-gcc
tar -zxvf gcc-arm-8.2-2018.08-x86_64-arm-linux-gnueabihf.tar.gz
```
7. 编译python
```
export PATH=/opt/python/gcc-arm-8.2-2018.08-x86_64-arm-linux-gnueabihf/bin:$PATH
export PATH=/opt/python/python3/bin:$PATH
export LD_LIBRARY_PATH=/opt/python/python3/lib:$LD_LIBRARY_PATH
./configure --host=arm-linux-gnueabihf --build=armv7 --prefix=/opt/python/python-arm --enable-ipv6 --enable-shared ac_cv_file__dev_ptmx="yes" ac_cv_file__dev_ptc="no"
make -j4 && make install
```

8. 打包
```
tar -zcvf python-arm.tar.gz python-arm
```
9. 移植
```
export LD_LIBRARY_PATH=/sdcard/python-arm/lib:$LD_LIBRARY_PATH
/sdcard/python-arm/bin/python3.8
```