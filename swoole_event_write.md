## swoole_event_write

用于PHP自带stream/sockets扩展创建的socket，使用fwrite/socket_send等函数向对端发送数据。当发送的数据量较大，socket写缓存区已满，就会发送阻塞等待或者返回EAGAIN错误。

swoole_event_write函数可以将stream/sockets资源的数据发送变成异步的，当缓冲区满了或者返回EAGAIN，swoole底层会将数据加入到发送队列，并监听可写。socket可写时swoole底层会自动写入。

~~~
$fp = stream_socket_client('tcp://127.0.0.1:9501');
$data = str_repeat('A', 1024 * 1024*2);

swoole_event_add($fp, function($fp) {
     echo fread($fp);
});

swoole_event_write($fp, $data);
~~~

* swoole_event_write不能用于SSL/TLS等有隧道加密的stream/sockets资源
* swoole_event_write调用之前，必须在将socket加入event_loop，否则会发生错误
* $data 发送数据的长度不得超过Socket缓存区尺寸
> 此函数在swoole-1.7.9以上版本可用

SOCKET缓存区已满后，Swoole的底层逻辑
持续写入SOCKET如果对端读取不够快，那SOCKET缓存区会塞满。swoole底层会将数据存到内存缓存区中，直到可写事件触发再写入SOCKET。

* 内存缓存区尺寸可以在通过修改php.ini 中的 swoole.socket_buffer_size 项进行配置，默认为8M
* 也可以使用swoole_async_set方法动态设置内存缓存区尺寸
如果内存缓存区也被写满了，此时swoole底层会抛出pipe buffer overflow, reactor will block. 错误，并进入阻塞等待。

如果调用端希望不要阻塞，直接返回错误，可以使用swoole_async_set设置socket_dontwait为true，write将不会阻塞而是直接返回false

> 缓存塞满返回false是原子操作，只会出现全部写入成功或者全部失败