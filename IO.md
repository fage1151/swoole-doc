## 异步文件系统IO
Swoole支持2种类型的异步文件读写IO，可以使用swoole_async_set来设置AIO模式:.


[TOC]


### Linux原生异步IO (AIO模式：SWOOLE_AIO_LINUX)
基于Linux Native AIO系统调用，是真正的异步IO，并非阻塞模拟。

> 优点：
> 所有操作均在一个线程内完成，不需要开线程池
> 不依赖线程执行IO，所以并发可以非常大

> 缺点：
> 只支持DriectIO，无法利用PageCache，所有对文件读写都会直接操作磁盘
> 写入数据的size必须为512整数倍数
> 写入数据的offset必须为512整数倍数

### 线程池模式异步IO (AIO模式： SWOOLE_AIO_BASE)
基于线程池模拟实现，文件读写请求投递到任务队列，然后由AIO线程读写文件，完成后通知主线程。AIO线程本身是同步阻塞的。所以并非真正的异步IO。

> 优点：
> 可以利用操作系统PageCache，读写热数据性能非常高，等于读内存
> 可修改thread_num项设置启用的AIO线程数量

> 缺点：
> 并发较差，不支持同时读写大量文件，最大并发受限与AIO的线程数量


## swoole_async_set
此函数可以设置异步IO相关的选项。

* swoole_async_set(array $setting);
* thread_num 设置异步文件IO线程的数量
* aio_mode 设置异步文件IO的操作模式，目前支持SWOOLE_AIO_BASE（使用类似于Node.js的线程池同步阻塞模拟异步）、SWOOLE_AIO_LINUX（Linux Native AIO） 2种模式
* enable_signalfd 开启和关闭signalfd特性的使用
* socket_buffer_size 设置SOCKET内存缓存区尺寸
* socket_dontwait 在内存缓存区已满的情况下禁止底层阻塞等待
* log_file 设置日志文件路径
* log_level 设置错误日志等级