# open_websocket_protocol
启用websocket协议处理，Swoole\WebSocket\Server会自动启用此选项。设置为false表示关闭websocket协议处理。

设置open_websocket_protocol选项为true后，会自动设置open_http_protocol协议也为true。

## WebSocket 
swoole-1.7.9 增加了内置的websocket服务器支持，通过几行PHP代码就可以写出一个异步非阻塞多进程的WebSocket服务器。

~~~
$server = new swoole_websocket_server("0.0.0.0", 9501);

$server->on('open', function (swoole_websocket_server $server, $request) {
    echo "server: handshake success with fd{$request->fd}\n";
});

$server->on('message', function (swoole_websocket_server $server, $frame) {
    echo "receive from {$frame->fd}:{$frame->data},opcode:{$frame->opcode},fin:{$frame->finish}\n";
    $server->push($frame->fd, "this is server");
});

$server->on('close', function ($ser, $fd) {
    echo "client {$fd} closed\n";
});

$server->start();
~~~
### onRequest回调
swoole_websocket_server 继承自 swoole_http_server

* 设置了onRequest回调，websocket服务器也可以同时作为http服务器
* 未设置onRequest回调，websocket服务器收到http请求后会返回http 400错误页面

### 客户端
* Chrome/Firefox/高版本IE/Safari等浏览器内置了JS语言的WebSocket客户端
* 微信小程序开发框架内置的WebSocket客户端
* 异步的PHP程序中可以使用Swoole\Http\Client作为WebSocket客户端
* apache/php-fpm或其他同步阻塞的PHP程序中可以使用swoole/framework提供的同步WebSocket客户端
* 非WebSocket客户端不能与WebSocket服务器通信