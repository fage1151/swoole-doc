# 心跳

注意：长链接应用必须加心跳，否则链接可能由于长时间未通讯被路由节点强行断开。

心跳作用主要有两个：

1、客户端定时给服务端发送点数据，防止连接由于长时间没有通讯而被某些节点的防火墙关闭导致连接断开的情况。

2、服务端可以通过心跳来判断客户端是否在线，如果客户端在规定时间内没有发来任何数据，就认为客户端下线。这样可以检测到客户端由于极端情况(断电、断网等)下线的事件。

使用Timer定时器功能可以实现发送心跳包的功能。事实上，Swoole已经内置了心跳检测功能，能自动close掉长时间没有数据来往的连接。而开启心跳检测功能，只需要设置heartbeat_check_interval和heartbeat_idle_time即可。如下：
~~~
$this->serv->set(
    array(
        'heartbeat_check_interval' => 60,
        'heartbeat_idle_time' => 600,
    )
);
~~~
* heartbeat_check_interval：启用心跳检测，此选项表示每隔多久轮循一次，单位为秒。如 heartbeat_check_interval => 60，表示每60秒，遍历所有连接，如果该连接在60秒内，没有向服务器发送任何数据，此连接将被强制关闭。
* heartbeat_idle_time：与heartbeat_check_interval配合使用。表示连接最大允许空闲的时间。

如
~~~
array(
    'heartbeat_idle_time' => 600,
    'heartbeat_check_interval' => 60,
);
~~~
表示每60秒遍历一次，一个连接如果600秒内未向服务器发送任何数据，此连接将被强制关闭
* 启用heartbeat_idle_time后，服务器并不会主动向客户端发送数据包
* 如果只设置了heartbeat_idle_time未设置heartbeat_check_interval底层将不会创建心跳检测线程，PHP代码中可以调用heartbeat方法手工处理超时的连接
* 在设置这两个选项后，swoole会在内部启动一个线程，每隔heartbeat_check_interval秒后遍历一次全部连接，检查最近一次发送数据的时间和当前时间的差，如果这个差值大于heartbeat_idle_time，则会强制关闭这个连接，并通过回调onClose通知Server进程。 

