# 入门
Swoole虽然是标准的PHP扩展，实际上与普通的扩展不同。普通的扩展只是提供一个库函数。而swoole扩展在运行后会接管PHP的控制权，进入事件循环。当IO事件发生后，swoole会自动回调指定的PHP函数。

新手入门教程：https://github.com/LinkedDestiny/swoole-doc
Swoole要求使用者必须具备一定的Linux/Unix环境编程基础，《学习Swoole需要掌握哪些基础知识》 本文列出了基础知识清单。

## swoole_server
强大的TCP/UDP Server框架，多线程，EventLoop，事件驱动，异步，Worker进程组，Task异步任务，毫秒定时器，SSL/TLS隧道加密。

* swoole_http_server是swoole_server的子类，内置了Http的支持
* swoole_websocket_server是swoole_http_server的子类，内置了WebSocket的支持
* swoole_redis_server是swoole_server的子类，内置了Redis服务器端协议的支持子类可以调用父类的所有方法和属性
## swoole_client
TCP/UDP/UnixSocket客户端，支持IPv4/IPv6，支持SSL/TLS隧道加密，支持SSL客户端整数，支持同步并发调用，也支持异步事件驱动编程。

## swoole_event
EventLoop API，让用户可以直接操作底层的事件循环，将socket，stream，管道等Linux文件加入到事件循环中。

eventloop接口仅可用于socket类型的文件描述符，不能用于磁盘文件读写
## swoole_async
异步IO接口，提供了 异步文件系统IO，定时器，异步DNS查询，异步MySQL等API，异步Http客户端，异步Redis客户端。

## swoole_timer 
异步毫秒定时器，可以实现间隔时间或一次性的定时任务
swoole_async_read/swoole_async_write 文件系统操作的异步接口
## swoole_process
进程管理模块，可以方便的创建子进程，进程间通信，进程管理。

## swoole_buffer
强大的内存区管理工具，像C一样进行指针计算，又无需关心内存的申请和释放，而且不用担心内存越界，底层全部做好了。

## swoole_table
基于共享内存和自旋锁实现的超高性能内存表。彻底解决线程，进程间数据共享，加锁同步等问题。

## tcp server
```php
<?php
$serv = new swoole_server('127.0.0.1', 9501);



$serv->on('connect', function ($serv, $fd) {
    echo "Client:Connect.\n";
});


$serv->on('receive', function ($serv, $fd, $from_id, $data) {
    $serv->send($fd, $data);
});

$serv->on('close', function ($serv, $fd) {
    echo "Client: Close.\n";
});

// Start our server, listen on the port and be ready to accept connections.
$serv->start();