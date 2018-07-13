# Coroutine

**并发调用**

Client并发请求

在协程版本的Client中，实现了多个客户端并发的发包功能。

通常，如果一个业务请求中需要做一次redis请求和一次mysql请求，那么网络IO会是这样子：

~~~
redis发包->redis收包->mysql发包->mysql收包
~~~
以上流程网络IO的时间就等于 redis网络IO时间 + mysql网络IO时间。

而对于协程版本的Client，网络IO可以是这样子：

~~~
redis发包->mysql发包->redis收包->mysql收包
~~~
以上流程网络IO的时间就接近于 MAX(redis网络IO时间, mysql网络IO时间)。

现在支持并发请求的Client有：

* Swoole\Coroutine\Client
* Swoole\Coroutine\Redis
* Swoole\Coroutine\MySQL
* Swoole\Coroutine\Http\Client

除了Swoole\Coroutine\Client，其他Client都实现了defer特性，用于声明延迟收包。

因为Swoole\Coroutine\Client的发包和收包方法是分开的，所以就不需要实现defer特性了，而其他Client的发包和收包都是在一个方法中，所以需要一个setDefer()方法声明延迟收包，然后通过recv()方法收包。

* * * * *

**实例**
协程版本Client并发请求示例代码：

~~~php
<?php
$server = new Swoole\Http\Server("127.0.0.1", 9502, SWOOLE_BASE);

$server->set([
    'worker_num' => 1,
]);

$server->on('Request', function ($request, $response) {

    $tcpclient = new Swoole\Coroutine\Client(SWOOLE_SOCK_TCP);
    $tcpclient->connect('127.0.0.1', 9501，0.5)
    $tcpclient->send("hello world\n");

    $redis = new Swoole\Coroutine\Redis();
    $redis->connect('127.0.0.1', 6379);
    $redis->setDefer();
    $redis->get('key');

    $mysql = new Swoole\Coroutine\MySQL();
    $mysql->connect([
        'host' => '127.0.0.1',
        'user' => 'user',
        'password' => 'pass',
        'database' => 'test',
    ]);
    $mysql->setDefer();
    $mysql->query('select sleep(1)');

    $httpclient = new Swoole\Coroutine\Http\Client('0.0.0.0', 9599);
    $httpclient->setHeaders(['Host' => "api.mp.qq.com"]);
    $httpclient->set([ 'timeout' => 1]);
    $httpclient->setDefer();
    $httpclient->get('/');

    $tcp_res  = $tcpclient->recv();
    $redis_res = $redis->recv();
    $mysql_res = $mysql->recv();
    $http_res  = $httpclient->recv();

    $response->end('Test End');
});
$server->start();
~~~