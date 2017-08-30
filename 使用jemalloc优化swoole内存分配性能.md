# 使用jemalloc优化swoole内存分配性能
**关于jemalloc**
jemalloc是一个比glibc malloc更高效的内存池技术，在Facebook公司被大量使用，在FreeBSD和FireFox项目中使用了jemalloc作为默认的内存管理器。使用jemalloc可以使程序的内存管理性能提升，减少内存碎片。

**安装jemalloc**
GITHUB主页：https://github.com/jemalloc/jemalloc
下载地址：https://github.com/jemalloc/jemalloc/releases/tag/4.0.4
编译安装：

~~~
cd jemalloc
./configure --with-jemalloc-prefix=je_
make -j 4
~~~
**使用jemalloc**
编译Swoole时增加`--with-jemalloc-dir=/path/to/jemalloc`

~~~
phpize
./configure --with-jemalloc-dir=/path/to/jemalloc
make 
make install
~~~