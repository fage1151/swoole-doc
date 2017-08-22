# http协程客户端
```php
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


$server = new Swoole\Http\Server('127.0.0.1', 9501, SWOOLE_BASE);

#1
$server->on('Request', function($request, $response) {
    $mysql = new Swoole\Coroutine\MySQL();
    #2
    $res = $mysql->connect([
        'host' => '127.0.0.1',
        'user' => 'root',
        'password' => 'root',
        'database' => 'test',
    ]);
    #3
    if ($res == false) {
        $response->end("MySQL connect fail!");
        return;
    }
    $ret = $mysql->query('show tables', 2);
    $response->end("swoole response is ok, result=".var_export($ret, true));
});

$server->start();
```
**启用http**
需要在编译swoole时增加--enable-coroutine来开启此功能。
swoole_http_client不依赖任何第三方库
支持Http-Chunk、Keep-Alive特性，暂不支持form-data格式
Http协议版本为HTTP/1.1
gzip压缩格式支持需要依赖zlib库