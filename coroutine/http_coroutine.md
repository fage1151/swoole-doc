# http协程客户端
```php
$server = new Swoole\Http\Server('127.0.0.1', 9501, SWOOLE_BASE);

$server->on('Request', function($request, $response) {
 $cli = new Swoole\Coroutine\Http\Client('127.0.0.1', 80);
$cli->setHeaders([
    'Host' => "localhost",
    "User-Agent" => 'Chrome/49.0.2587.3',
    'Accept' => 'text/html,application/xhtml+xml,application/xml',
    'Accept-Encoding' => 'gzip',
]);
$cli->set([ 'timeout' => 1]);
$cli->get('/index.php');
echo $cli->body;
$cli->close();
});

$server->start();
```
**启用http**
* 需要在编译swoole时增加--enable-coroutine来开启此功能。
* swoole_http_client不依赖任何第三方库
* 支持Http-Chunk、Keep-Alive特性，暂不支持form-data格式
* Http协议版本为HTTP/1.1
* gzip压缩格式支持需要依赖zlib库