# sleep协程api

进入等待状态。相当于PHP的sleep函数，不同的是Coroutine::sleep是协程调度器实现的，底层会yield当前协程，让出时间片，并添加一个异步定时器，当超时时间到达时重新resume当前协程，恢复运行。使用sleep接口可以方便地实现超时等待功能。

~~~
function Coroutine::sleep(float $seconds);
~~~
* $seconds为睡眠的时间，单位为秒，支持浮点型，最小精度为毫秒（0.001秒）
* $seconds必须大于0，最大不得超过一天时间（86400秒）

>在2.0.9或更高版本可用

```php
<?php
$serv = new Swoole\Http\Server('127.0.0.1',9090);
$serv->on('request',function($request,$response){
    Swoole\Coroutine::sleep(0.5);
    $response->end('hello world');
});
$serv->start();
```