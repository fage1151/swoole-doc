# 是否可以共用1个redis或mysql连接 
绝对不可以。必须每个进程单独创建redis/mysql连接，其他的存储客户端同样也是如此。原因是如果共用1个连接，那么返回的结果无法保证被哪个进程处理。持有连接的进程理论上都可以对这个连接进行读写，这样数据就发生错乱了。

**所以在多个进程之间，一定不能共用连接**

* 在swoole_server中，应当在`onWorkerStart`中创建连接对象
* 在swoole_process中，应当在`swoole_process->start`后，子进程回调函数中创建连接对象
* 本页面所述信息对使用pcntl_fork的程序同样有效

**示例**

~~~php
$serv = new swoole_server("0.0.0.0", 9502);

//必须在onWorkerStart回调中创建redis/mysql连接
$serv->on('workerstart', function($serv, $id) {
    $redis = new redis;
    $redis->connect('127.0.0.1', 6379);
    $serv->redis = $redis;
});

$serv->on('receive', function (swoole_server $serv, $fd, $from_id, $data) { 
    $value = $serv->redis->get("key");
    $serv->send($fd, "Swoole: ".$value);
});

$serv->start();
~~~