Swoole在2.0开始内置协程(Coroutine)的能力，提供了具备协程能力IO接口（统一在命名空间Swoole\Coroutine\*）

>2.0.2或更高版本已支持PHP7

协程可以理解为纯用户态的线程，其通过协作而不是抢占来进行切换。相对于进程或者线程，协程所有的操作都可以在用户态完成，创建和切换的消耗更低。Swoole可以为每一个请求创建对应的协程，根据IO的状态来合理的调度协程，这会带来了以下优势：

* 开发者可以无感知的用同步的代码编写方式达到异步IO的效果和性能，避免了传统异步回调所带来的离散的代码逻辑和陷入多层回调中导致代码无法维护。

* 同时由于swoole是在底层封装了协程，所以对比传统的php层协程框架，开发者不需要使用yield关键词来标识一个协程IO操作，所以不再需要对yield的语义进行深入理解以及对每一级的调用都修改为yield，这极大的提高了开发效率。

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