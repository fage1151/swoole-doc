## swoole
Swoole是标准的PHP扩展，实际上与普通的扩展不同。普通的扩展只是提供一个库函数。而swoole扩展在运行后会接管PHP的控制权，进入事件循环。当IO事件发生后，swoole会自动回调指定的PHP函数。
## swoole提供的模块
### ①swoole_server
强大的TCP/UDP Server框架，多线程，EventLoop，事件驱动，异步，Worker进程组，Task异步任务，毫秒定时器，SSL/TLS隧道加密。

#### swoole_http_server是swoole_server的子类，内置了Http的支持
#### swoole_websocket_server是swoole_http_server的子类，内置了WebSocket的支持
#### swoole_redis_server是swoole_server的子类，内置了Redis服务器端协议的支持
### ②swoole_client
TCP/UDP/UnixSocket客户端，支持IPv4/IPv6，支持SSL/TLS隧道加密，支持SSL客户端整数，支持同步并发调用，也支持异步事件驱动编程。
#### swoole_mysql
#### swoole_redis
#### swoole_http_client 包含http客户端和websocket客户端
#### Swoole\Http2\Client 对Http2.0客户端的支持,继承自Swoole\Client
### ③swoole_event
EventLoop API，让用户可以直接操作底层的事件循环，将socket，stream，管道等Linux文件加入到事件循环中。
### ④swoole_async
#### swoole_timer 异步毫秒定时器，可以实现间隔时间或一次性的定时任务
#### swoole_async_read/swoole_async_write 文件系统操作的异步接口
### ⑤swoole_process
进程管理模块，可以方便的创建子进程，进程间通信，进程管理。
### ⑥swoole_buffer
强大的内存区管理工具，像C一样进行指针计算，又无需关心内存的申请和释放，而且不用担心内存越界，底层全部做好了。
### ⑦swoole_table 
基于共享内存和自旋锁实现的超高性能内存表。彻底解决线程，进程间数据共享，加锁同步等问题。




