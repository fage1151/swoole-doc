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


## Coroutine\Http2\Client
Http2协程客户端。

**实例**
~~~
use Swoole\Coroutine as co;

co::create(function ()
{
    $cli = new co\Http2\Client('127.0.0.1', 9518);
    $cli->set([ 'timeout' => 1]);
    $cli->connect();

    $req = new co\Http2\Request;
    $req->path = "/index.html";
    $req->headers = [
        'host' => "localhost",
        "user-agent" => 'Chrome/49.0.2587.3',
        'accept' => 'text/html,application/xhtml+xml,application/xml',
        'accept-encoding' => 'gzip',
    ];
    $req->cookies = ['name' => 'rango', 'email' => '1234@qq.com'];
    var_dump($cli->send($req));
    $resp = $cli->recv();
    var_dump($resp);

});
~~~