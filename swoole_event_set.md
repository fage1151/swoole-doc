## swoole_event_set
修改事件监听的回调函数和掩码。函数原型：

~~~
bool swoole_event_set($fd, mixed $read_callback, mixed $write_callback, int $flags);
~~~

参数与swoole_event_add完全相同。如果传入$fd在EventLoop中不存在这里会报错。

* 当$read_callback不为null时，将修改可读事件回调函数为指定的函数
* 当$write_callback不为null时，将修改可写事件回调函数为指定的函数
* $flags可关闭/开启，可写（SW_EVENT_READ）和可读（SW_EVENT_WRITE）事件的监听