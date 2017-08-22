## swoole_event_add

swoole_event_add函数用于将一个socket加入到swoole的reactor事件监听中。此函数可以用在Server或Client模式下。

~~~
bool swoole_event_add(int $sock, mixed $read_callback, mixed $write_callback = null, int $flags = null);
~~~

参数1可以为以下三种类型：

* int，就是文件描述符,包括swoole_client的socket,以及第三方扩展的socket（比如mysql）
* stream资源，就是stream_socket_client/fsockopen 创建的资源
* sockets资源，就是sockets扩展中 socket_create创建的资源，需要在编译时加入 ./configure --enable-sockets

参数2为可读回调函数，参数3为可写事件回调，可以是字符串函数名、对象+方法、类静态方法或匿名函数，当此socket可读时回调指定的函数。

参数4为事件类型的掩码，可选择关闭/开启可读可写事件，如SWOOLE_EVENT_READ，SWOOLE_EVENT_WRITE，或者SWOOLE_EVENT_READ | SWOOLE_EVENT_WRITE

> swoole_event_add在swoole1.6.2+之后可用
> 第3，4个参数在1.7.1版本后可用，用于监听可写事件回调，以及设置读写事件的监听

在Server程序中使用，可以理解为在worker/taskworker进程中将此socket注册到epoll事件中。
在Client程序中使用，可以理解为在客户端进程中将此socket注册到epoll事件中。

~~~
$fp = stream_socket_client("tcp://www.qq.com:80", $errno, $errstr, 30);
fwrite($fp,"GET / HTTP/1.1\r\nHost: www.qq.com\r\n\r\n");

swoole_event_add($fp, function($fp) {
    $resp = fread($fp, 8192);
    //socket处理完成后，从epoll事件中移除socket
    swoole_event_del($fp);
    fclose($fp);
});
echo "Finish\n";  //swoole_event_add不会阻塞进程，这行代码会顺序执行
~~~
### 回调函数
* 在可读事件回调函数中必须使用fread、recv等函数读取Socket缓存区中的数据，否则事件会持续触发，如果不希望继续读取必须使用Swoole\Event::del移除事件监听
* 在可写事件回调函数中，写入socket之后必须调用Swoole\Event::del移除事件监听，否则可写事件会持续触发
* 执行fread、socekt_recv、socket_read、Swoole\Client::recv返回false，并且错误码为EAGAIN时表示当前Socket接收缓存区内没有任何数据，这时需要加入可读监听等待EventLoop通知
* 执行fwrite、socket_write、socket_send、Swoole\Client::send操作返回false，并且错误码为EAGAIN时表示当前Socket发送缓存区已满，暂时不能发送数据。需要监听可写事件等待EventLoop通知



