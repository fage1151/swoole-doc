# getDefer
getDefer()

~~~
bool getDefer();
~~~

* 返回值：返回当前设置的defer

~~~
<?php
$server = new Swoole\Http\Server("127.0.0.1", 9502, SWOOLE_BASE);

$server->set([
    'worker_num' => 1,
]);

$server->on('Request', function ($request, $response) {

    $httpclient = new Swoole\Coroutine\Http\Client('0.0.0.0', 9599);
    $httpclient->setHeaders(['Host' => "api.mp.qq.com"]);
    $httpclient->set([ 'timeout' => 1]);
    $httpclient->setDefer();
    $httpclient->get('/');
    echo $httpclient->getDefer();
    $http_res  = $httpclient->recv();

    $response->end('Test End');
});
$server->start();
~~~