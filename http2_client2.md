# 异步Http2.0客户端

Swoole-1.9.7增加了对Http2.0客户端的支持。新增的客户端类名为Swoole\Http2\Client，继承自Swoole\Client，实现了Http2.0客户端协议的完整支持。

Http2.0客户端与Http1.1的最大差别是2.0支持了Stream并发机制，可以同时发起多个GET或POST请求。最大并发数量受限与服务器端规定的max_concurrent_streams设置。

**需要依赖nghttp2库，编译Swoole扩展时需要设置--enable-http2、--enable-openssl或--with-openssl-dir。**

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