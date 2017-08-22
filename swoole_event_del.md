## swoole_event_del
swoole_event_del函数用于从reactor中移除监听的socket。swoole_event_del应当与swoole_event_add成对使用。 

~~~
bool swoole_event_del(int $sock);
~~~

* 参数为socket的文件描述符。