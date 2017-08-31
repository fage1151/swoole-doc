# 异步Http2.0客户端

Swoole-1.9.7增加了对Http2.0客户端的支持。新增的客户端类名为Swoole\Http2\Client，继承自Swoole\Client，实现了Http2.0客户端协议的完整支持。

Http2.0客户端与Http1.1的最大差别是2.0支持了Stream并发机制，可以同时发起多个GET或POST请求。最大并发数量受限与服务器端规定的max_concurrent_streams设置。

**需要依赖nghttp2库，编译Swoole扩展时需要设置--enable-http2、--enable-openssl或--with-openssl-dir。**

[TOC=2,3]

## **swoole_http2_client->__construct**
构造方法，与swoole_http_client的构造方法参数完全一致，共接受3个参数。

~~~
function swoole_http2_client->__construct($host, $port, $ssl = false)

~~~
* $host 服务器的地址，如果未设置host头，将自动使用$host参数作为默认的host头
* $port 端口号，SSL一般为443，非SSL一般为80
* $ssl 是否启用SSL加密，需要依赖openssl

## **swoole_http2_client->get**
发起GET请求，函数原型：

~~~
function swoole_http_client->get(string $path, callable $callback);

~~~
* $path 设置URL路径，如/index.html，注意这里不能传入http://domain
* $callback 调用成功或失败后回调此函数
* Http响应内容会在内存中进行数据拼接。因此如果响应体很大可能会占用大量内存

**回调函数**

与Http1.1客户端事件回调函数不同，Http2.0回调函数中的参数为Swoole\Http2\Response对象。而不是Client本身。可使用use语法将Client对象传递给匿名函数。

~~~
function callback(Swoole\Http2\Response $resp)
{
    var_dump($resp->cookie);
    var_dump($resp->header);
    var_dump($resp->server);
    var_dump($resp->body);
    var_dump($resp->statusCode);
}
~~~

* cookie 服务器设置的COOKIE信息
* header 服务器发送的Header信息
* server 底层连接与协议相关的信息
* body 服务器发送的响应包体
* statusCode 服务器发送的Http状态码，如200、502等

## **swoole_http2_client->post**
发起POST请求，函数原型：

~~~
function swoole_http2_client->post(string $path, mixed $data, callable $callback);

~~~
* $path 设置URL路径，如/index.html，注意这里不能传入http://domain
* $data 请求的包体数据，如果$data为数组底层自动会打包为x-www-form-urlencoded格式的POST内容，并设置Content-Type为application/x-www-form-urlencoded
* $callback 调用成功或失败后回调此函数

**使用实例**
```
$cli = new swoole_http2_client('127.0.0.1', 80); 
$cli->post('/post.php', array("a" => '1234', 'b' => '456'), function ($response) {
    echo "Length: " . strlen($cli->body) . "\n";
    echo $cli->body;
});
```
## **swoole_http2_client->setHeaders**
设置Http请求头

~~~
function swoole_http2_client->setHeaders(array $headers);

~~~
* $headers必须为键值对应的数组，底层会自动映射为$key: $value格式的Http标准头格式
* setHeaders设置的Http头在swoole_http2_client对象存活期间的每次请求永久有效
* 重新调用setHeaders会覆盖上一次的设置

## **swoole_http2_client->setCookies**
设置Cookie 

~~~
function swoole_http2_client->setCookies(array $cookies);

~~~
* $cookies 设置COOKIE，必须为键值对应数组
* 设置COOKIE后在客户端对象存活期间会持续保存
* 服务器端主动设置的COOKIE会合并到cookies数组中，可读取$client->cookies属性获得当前Http2客户端的COOKIE信息
* 重新调用setCookies方法会覆盖已有COOKIE

```php
<?php
$array = array(
    "host" => "www.jd.com",
    "accept-encoding" => "gzip, deflate",
    'accept' => 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
    'accept-language' => 'zh-CN,zh;q=0.8,en;q=0.6,zh-TW;q=0.4,ja;q=0.2',
    'user-agent' => 'Mozilla/5.0 (X11; Linux x86_64) Chrome/58.0.3026.3 Safari/537.36',
);

$client = new Swoole\Http2\Client("www.jd.com", 443, true);

$client->setHeaders($array);
$client->setCookies(array("a" => "1", "b" => "2"));

$client->get("/", function ($o) use($client) {
    echo "#{$client->sock} hello world 1\n";
    echo $o->body;
});

$client->post("/", $array, function ($o) use($client) {
    echo "{$client->sock} hello world 3\n";
    echo $o->body;
    $client->close();
});
Swoole\Event::wait();