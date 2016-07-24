---
  layout: post
  title: CentosUpGCC
---
  
一、简易安装

操作环境 CentOS6.5 64bit，原版本4.4.7，不能支持C++11的特性~，希望升级到4.8.2

不能通过yum的方法升级，需要自己手动下载安装包并编译

1.1 获取安装包并解压

wget http://ftp.gnu.org/gnu/gcc/gcc-4.8.2/gcc-4.8.2.tar.bz2
tar -jxvf gcc-4.8.2.tar.bz2
当然，http://ftp.gnu.org/gnu/gcc  里面有所有的gcc版本供下载，最新版本已经有4.9.2啦.

1.2 下载供编译需求的依赖项

参考文献[1]中说：这个神奇的脚本文件会帮我们下载、配置、安装依赖库，可以节约我们大量的时间和精力。

cd gcc-4.8.0　
./contrib/download_prerequisites　
1.3 建立一个目录供编译出的文件存放

mkdir gcc-build-4.8.2
cd gcc-build-4.8.2
1.4 生成Makefile文件

../configure -enable-checking=release -enable-languages=c,c++ -disable-multilib
1.5 编译（注意：此步骤非常耗时）

make -j4
-j4选项是make对多核处理器的优化，如果不成功请使用 make，相关优化选项可以移步至参考文献[2]。

我在安装此步骤时候出错，错误描述：

 
compilation terminated.
make[5]: *** [_gcov_merge_add.o] 错误 1
make[5]: Leaving directory `/home/imdb/gcc-4.8.2/gcc-build-4.8.2/x86_64-unknown-linux-gnu/32/libgcc'
make[4]: *** [multi-do] 错误 1
make[4]: Leaving directory `/home/imdb/gcc-4.8.2/gcc-build-4.8.2/x86_64-unknown-linux-gnu/libgcc'
make[3]: *** [all-multi] 错误 2
make[3]: *** 正在等待未完成的任务....
make[3]: Leaving directory `/home/imdb/gcc-4.8.2/gcc-build-4.8.2/x86_64-unknown-linux-gnu/libgcc'
make[2]: *** [all-stage1-target-libgcc] 错误 2
make[2]: Leaving directory `/home/imdb/gcc-4.8.2/gcc-build-4.8.2'
make[1]: *** [stage1-bubble] 错误 2
make[1]: Leaving directory `/home/imdb/gcc-4.8.2/gcc-build-4.8.2'
make: *** [all] 错误 2
 
大概看看，错误集中在 x86_64unknown-linux-gnu/32/libgcc 和 x86_64-unknown-linux-gnu/libgcc

根据参考文献[3]，安装如下两个软件包（仅用于CentOS6.X）：

sudo yum -y install glibc-devel.i686 glibc-devel
过程中CPU基本满载：



1.6、安装

sudo make install
 

二、验证安装

重启，然后查看gcc版本：

gcc -v
尝试写一个C++11特性的程序段 tryCpp11.cc，使用了shared_ptr

 
 1 //tryCpp11.cc
 2 #include <iostream>
 3 #include <memory>
 4 
 5 int main()
 6 {
 7     std::shared_ptr<int> pInt(new int(5));
 8     std::cout << *pInt << std::endl;
 9     return 0;
10 }
 
验证文件：

g++ -std=c++11 -o tryCpp11 tryCpp11.cc
./tryCpp11
Linux升级GCC 4.8.1清晰简明教程(Ubuntu 12.04 64位版为例)  http://www.linuxidc.com/Linux/2014-04/99583.htm

在CentOS 6.4中编译安装GCC 4.8.1 + GDB 7.6.1 + Eclipse 在CentOS 6.4中编译安装GCC 4.8.1 + GDB 7.6.1 + Eclipse

Ubuntu下Vim+GCC+GDB安装及使用 http://www.linuxidc.com/Linux/2013-01/78159.htm

Ubuntu下两个GCC版本切换 http://www.linuxidc.com/Linux/2012-10/72284.htm
