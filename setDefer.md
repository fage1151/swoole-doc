# setDefer
setDefer()

~~~
bool setDefer([bool $is_defer = true]);
~~~
* $is_defer：bool值，为true时，表明该Client要延迟收包，为false时，表明该Client非延迟收包，默认值为true
* 返回值：设置成功返回true，否则返回false。只有一种情况会返回false，当设置defer(true)并发包后，尚未recv()收包，就设置defer(false)，此时返回false。
* 如果需要进行延迟收包，需要在发包之前调用

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
    
    $httpclient = new Swoole\Coroutine\Http\Client('0.0.0.0', 9599);
    $httpclient->setHeaders(['Host' => "api.mp.qq.com"]);
    $httpclient->set([ 'timeout' => 1]);
    $httpclient->setDefer();
    $httpclient->get('/');

    $tcp_res  = $tcpclient->recv();
    $redis_res = $redis->recv();
    $http_res  = $httpclient->recv();

    $response->end('Test End');
});
$server->start();
~~~