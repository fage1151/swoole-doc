# recv
recv()

~~~
mixed recv();
~~~
* 返回值：获取延迟收包的结果，当没有进行延迟收包或者收包超时，返回false。

~~~php
<?php
$server = new Swoole\Http\Server("127.0.0.1", 9502, SWOOLE_BASE);

$server->set([
    'worker_num' => 1,
]);

$server->on('Request', function ($request, $response) {
    $redis = new Swoole\Coroutine\Redis();
    $redis->connect('127.0.0.1', 6379);
    $redis->setDefer();
    $redis->get('key');
    
    $httpclient = new Swoole\Coroutine\Http\Client('0.0.0.0', 9599);
    $httpclient->setHeaders(['Host' => "api.mp.qq.com"]);
    $httpclient->set([ 'timeout' => 1]);
    $httpclient->setDefer();
    $httpclient->get('/');

    $redis_res = $redis->recv();
    $http_res  = $httpclient->recv();

    $response->end('Test End');
});
$server->start();
~~~