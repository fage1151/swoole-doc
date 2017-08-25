Swoole在2.0开始内置协程(Coroutine)的能力，提供了具备协程能力IO接口（统一在命名空间Swoole\Coroutine\*）

>2.0.2或更高版本已支持PHP7

swoole提供了四种协程Client：

* TCP/UDP Client->\Swoole\Coroutine\Client  
* HTTP Client->\Swoole\Coroutine\HTTP\Client  
* Redis Client->\Swoole\Coroutine\Redis
* Mysql Client->\Swoole\Coroutine\MySQL

在协程Server中需要使用协程版Client，可以实现全异步server

同时swoole提供了协程工具集：\Swoole\Coroutine\Util，提供了获取当前协程id，反射调用等能力。
```php
<?php
for ($i = 0; $i < 100; $i++) {
    Swoole\Coroutine::create(function() use ($i) {
        $redis = new Swoole\Coroutine\Redis();
        $res = $redis->connect('127.0.0.1', 6379);
        $ret = $redis->incr('coroutine');
        $redis->close();
        if ($i == 50) {
            Swoole\Coroutine::create(function() use ($i) {
                $redis = new Swoole\Coroutine\Redis();
                $res = $redis->connect('127.0.0.1', 6379);
                $ret = $redis->set('coroutine_i', 50);
                $redis->close();
            });
        }
    });
}
```
支持协程的回调方法列表 
目前Swoole2 仅有部分事件回调函数底层自动创建了协程，可以调用协程客户端。本节列出了支持协程客户端的回调列表以及实现的版本号。

v2.0.5  
onConnect  
onReceive  
onPacket  
onRequest  
onHandShake  
v2.0.6  
onMessage  
v2.0.7  
onOpen  
Redis\Server->handler