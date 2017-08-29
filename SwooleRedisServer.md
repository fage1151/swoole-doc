# Swoole\Redis\Server
Swoole-1.8.14版本增加一个兼容Redis服务器端协议的Server框架，可基于此框架实现Redis协议的服务器程序。Swoole\Redis\Server继承自Swoole\Server，可调用父类提供的所有方法。

Redis\Server不需要设置onReceive回调。实例程序：https://github.com/swoole/swoole-src/blob/master/examples/redis/server.php

**可用的客户端**
* 任意编程语言的redis客户端，包括PHP的redis扩展和phpredis库
* Swoole扩展提供的异步Redis客户端
* Redis提供的命令行工具，包括redis-cli、redis-benchmark

**方法**
Swoole\Redis\Server继承自Swoole\Server，可以使用父类提供的所有方法。

Redis\Server不需要设置onReceive回调。只需使用setHandler方法设置对应命令的处理函数，收到未支持的命令后会自动向客户端发送ERROR响应，消息为ERR unknown command '$command'。

**setHandler**
设置Redis命令字的处理器。

~~~
function swoole_redis_server->setHandler(string $command, callable $callback);
~~~
* $command 命令的名称
* $callback 命令的处理函数，回调函数返回字符串类型时会自动发送给客户端
* $callback 返回的数据必须为Redis格式，可使用format静态方法进行打包
服务器实例
~~~
use Swoole\Redis\Server;

$server = new Server('127.0.0.1', 9501);

//同步模式
$server->setHandler('Set', function($fd, $data) {
    $server->array($data[0], $data[1]);
    return Server::format(Server::INT, 1);
});

//异步模式
$server->setHandler('Get', function ($fd, $data) use ($server) {
    $db->query($sql, function($db, $result) use ($fd) {
        $server->send($fd, Server::format(Server::LIST, $result));
    });
});

$server->start();
~~~

客户端实例

~~~
redis-cli -h 127.0.0.1 -p 9501 set name rango

~~~
**format**
格式化命令响应数据。

~~~
function swoole_redis_server::format(int $type, mixed $value = null);
~~~

* format为静态方法
* $type表示数据类型，NIL类型不需要传入$value，ERROR和STATUS类型$value可选，INT、STRING、SET、MAP必选
