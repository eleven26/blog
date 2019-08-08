---
title: centos 6.10 升级 swoole 到 4.3.3.md
date: 2019-08-08 09:00:00
tags: [Swoole, CentOS]
---

upgrade-swoole.sh

```bash
#!/usr/bin/env bash

# 参考资料:
# https://blog.csdn.net/hutianyou123/article/details/78133309 (CentOS6.5升级手动安装GCC4.8.2 与 CentOS 6.4 编译安装 gcc 4.8.1)
# https://www.cnblogs.com/coyu/p/5750627.html (CentOS yum升级GCC到4.8)

# swoole 4.3.3 依赖 4.8 以后的 gcc, 需要先升级 gcc
wget http://ftp.gnu.org/gnu/gcc/gcc-4.8.2/gcc-4.8.2.tar.bz2
tar -jxvf gcc-4.8.2.tar.bz2

cd gcc-4.8.2
bash ./contrib/download_prerequisites

mkdir gcc-build-4.8.2
cd gcc-build-4.8.2
../configure -enable-checking=release -enable-languages=c,c++ -disable-multilib
make -j4
sudo make install

wget http://people.centos.org/tru/devtools-2/devtools-2.repo
mv devtools-2.repo /etc/yum.repos.d
yum install devtoolset-2-gcc devtoolset-2-binutils devtoolset-2-gcc-c++

mv /usr/bin/gcc /usr/bin/gcc-4.4.7
mv /usr/bin/g++ /usr/bin/g++-4.4.7
mv /usr/bin/c++ /usr/bin/c++-4.4.7
ln -s /opt/rh/devtoolset-2/root/usr/bin/gcc /usr/bin/gcc
ln -s /opt/rh/devtoolset-2/root/usr/bin/c++ /usr/bin/c++
ln -s /opt/rh/devtoolset-2/root/usr/bin/g++ /usr/bin/g++
gcc --version


# 安装 swoole 4.3.3
wget https://github.com/swoole/swoole-src/archive/v4.3.3.tar.gz
tar -xvf v4.3.3.tar.gz
cd swoole-src-4.3.3
phpize
./configure
make
sudo make install

php --re swoole | grep version

```
