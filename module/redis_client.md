##redis client
异步Redis客户端 

[TOC]
Swoole-1.8.0版本增加了对异步Redis客户端的支持，基于redis官方提供的hiredis库实现。Swoole提供了__call魔术方法，来映射绝大部分Redis指令。

## **编译安装hiredis**
使用Redis客户端，需要安装hiredis库。下载hiredis源码后，执行

~~~
make -j
sudo make install
sudo ldconfig
~~~
* hiredis下载地址：https://github.com/redis/hiredis/releases

## **启用异步Redis客户端**
编译swoole是，在configure指令中加入--enable-async-redis

~~~
./configure --enable-async-redis
make clean
make -j
sudo make install
~~~
## **swoole_redis->__construct**
Redis异步客户端构造方法，可以设置Redis连接的配置选项。

~~~
function swoole_redis->__construct(array $options = null);

~~~
* $options 配置选项数组，默认为null
* 在1.9.15或更高版本可用
**超时控制**
~~~
$options['timeout'] = 1.5;

~~~
浮点型，单位为秒，最小粒度为1毫秒。Connect后，在规定的时间内服务器没有完成握手，底层将自动关闭socket，设置连接为失败，触发onConnect事件
**设置密码**

~~~
$options['password'] = 'passwd';

~~~
必须为字符串类型，可以设置Redis服务器密码，等同于auth指令
**设置数据库
~~~
$options['database'] = 0;

~~~
*整型，设置使用的Redis服务器的数据库编号，等同于select指令

>设置了password或database选项后，连接就绪后底层会自动发送相关指令
>等待服务器响应成功后才会触发onConnect连接成功事件
>如果password或database错误，onConnect连接结果为失败

## **swoole_redis->on**
注册事件回调函数。

~~~
function swoole_redis->on(string $event_name, callable $callback);

~~~
目前swoole_redis支持2种事件回调函数。on方法必须在connect前被调用。

**onClose**
当Redis服务器主动关闭连接或者客户端主动调用close关闭连接时，会触发onClose事件。

~~~
function onClose(swoole_redis $redis);
~~~
**onMessage**
当客户端收到来自服务器的订阅消息时触发onMessage事件。

~~~
function onMessage(swoole_redis $redis, array $message);
~~~
## **swoole_redis->connect**
连接到Redis服务器

**函数原型：**
~~~
function swoole_redis->connect(string $host, int $port, callable $callback);

~~~
* $host: Redis服务器的主机IP
* $port: Redis服务器的端口
* $callback: 连接成功后回调的函数
**回调函数**
~~~
function onConnect(swoole_redis $redis, bool $result);

~~~
* $redis: redis连接对象
* $result: 连接成功为true，连接失败为false，可以读取$redis->errCode获得错误码，读取$redis->errMsg获得错误消息
* 连接成功后就可以执行Redis指令了。

**使用示例**
~~~
$client = new swoole_redis;
$client->connect('127.0.0.1', 6379, function (swoole_redis $client, $result) {
    if ($result === false) {
        echo "connect to redis server failed.\n"
        return;
    }
    $client->set('key', 'swoole', function (swoole_redis $client, $result) {
        var_dump($result);
    });
});
~~~
## **swoole_redis->__call**
魔术方法，方法名会映射为Redis指令，参数作为Redis指令的参数。

**函数原型**
~~~
function swoole_redis->__call(string $command, array $params);
~~~
* $command，必须为合法的Redis指令，详细参见Redis指令列表
* $params的最后一个参数必须为可执行的函数，其他参数必须为字符串
**订阅/发布消息**

Redis服务器除了作为内存存储之外，还可以作为一个消息通道服务器。SwooleRedis客户端也支持了Redis的订阅/发布消息指令。

与普通的存储指令不同，消息订阅/发布指令不是请求响应式的。

* 订阅/发布指令没有回调函数，不需要在最后一个参数传入callback
* 使用订阅/发布消息命名，必须设置onMessage事件回调函数
* 客户端发出了subscribe命令后，只能执行subscribe， psubscribe，unsubscribe，punsubscribe这4条命令
~~~

$client = new swoole_redis;
$client->on('message', function (swoole_redis $client, $result) {
    var_dump($result);
    static $more = false;
    if (!$more and $result[0] == 'message')
    {
        echo "subscribe new channel\n";
        $client->subscribe('msg_1', 'msg_2');
        $client->unsubscribe('msg_0');
        $more = true;
    }
});
$client->connect('127.0.0.1', 6379, function (swoole_redis $client, $result) {
    echo "connect\n";
    $client->subscribe('msg_0');
});
~~~
**回调函数**
~~~
function onReceive(swoole_redis $redis, bool $result);

~~~
* $redis: redis连接对象
* 执行失败，$result为false, 可以读取$redis->errCode获得错误码，读取$redis->errMsg获得错误消息
* 执行成功，返回数据结果，可能是字符串、数组或true

**使用示例**

~~~
$client->get('key', function (swoole_redis $client, $result) {
    var_dump($result);
});

~~~
## **swoole_redis->close**
关闭Redis连接，不接受任何参数。

~~~
function swoole_redis->close()

~~~
异步redis客户端
```php
$client = new Redis;
$client->connect('127.0.0.1', 6379, function (Redis $client, $result) {
    echo "connect\n";
    var_dump($result);
    $db = 0;
    $client->select($db);
    $password = '111111';
    $client->auth($password);
});
$client->set('key', 'swoole', function (Redis $client, $result) {
    var_dump($result);
    $client->get('key', function (Redis $client, $result) {
        var_dump($result);
        $client->close();
    });
});
```