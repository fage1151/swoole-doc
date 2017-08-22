Swoole 2.0 增加了一个 sleep 协程 API，可以方便地实现请求等待。完全同步的代码，底层是基于 epoll 的纯异步 IO 
```php
<?php
$serv = new Swoole\Http\Server('127.0.0.1',9090);
$serv->on('request',function($request,$response){
    Swoole\Coroutine::sleep(0.5);
    $response->end('hello world');
});
$serv->start();
```