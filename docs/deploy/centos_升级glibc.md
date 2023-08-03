# centos7 升级 glibc7

# 准备
```
yum install -y bzip2 python3 bison
```

升级make
```
wget http://ftp.gnu.org/gnu/make/make-4.2.tar.gz
tar -zxvf make-4.2.tar.gz
cd make-4.2
./configure
make
make install
sudo ln -s -f /usr/local/bin/make /usr/bin/make
make --version
```

升级gcc
```
yum install centos-release-scl scl-utils-build
yum install -y devtoolset-8-toolchain
scl enable devtoolset-8 bash
gcc --versio


```

编译glibc
```

wget http://ftp.gnu.org/gnu/glibc/glibc-2.29.tar.gz
tar xvf glibc-2.29.tar.gz
cd glibc-2.29
mkdir build
cd build
../configure --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin
make  
make install

```

检查
```
strings /lib64/libc.so.6 |grep GLIBC
```