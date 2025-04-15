---
title: 编译gcc
date: 2024-03-06 16:21:39
tags:
---

# 下载代码
[code](https://ftp.gnu.org/gnu/gcc/)。关于版本，我这里参考的是[cppreference](https://en.cppreference.com/w/cpp/17), 例如：通过阅读文档，可以知道：完全支持cpp17, 至少需要 [`gcc-7.0`](https://ftp.gnu.org/gnu/gcc/gcc-7.5.0/)
```shell
wget https://ftp.gnu.org/gnu/gcc/gcc-13.2.0/gcc-13.2.0.tar.gz
tar -xzf gcc-13.2.0.tar.gz
cd gcc-13.2.0
 ```
# 编译和安装
```shell
mkdir build
cd build
../configure --prefix=/usr/local/gcc-13.2.0/build --enable-languages=c,c++ --disable-multilib
make
make install
```
- `--prefix=[PATH]`: 这个参数非常重要，确定了gcc的安装路径。就是执行`make install`后，gcc的安装路径。
- 当执行完`make install`后，才会生成完整的gcc。

# 使用
- 配置环境变量
```shell
export CC=/usr/local/gcc-13.2.0/build/bin/gcc
export CXX=/usr/local/gcc-13.2.0/build/bin/g++
```
- 写入CMake
```cmake
set(CMAKE_C_COMPILER /usr/local/gcc-13.2.0/build/bin/gcc)
set(CMAKE_CXX_COMPILER /usr/local/gcc-13.2.0/build/bin/g++)
```
参考[how-to-specify-new-gcc-path-for-cmake](https://stackoverflow.com/questions/17275348/how-to-specify-new-gcc-path-for-cmake)

