# 在php-fpm或apache中使用swoole
swoole中绝大部分的模块只能用于CLI命令行环境，只有同步阻塞的swoole_client可以用于php-fpm或apache环境。

## **同步swoole_client**
~~~php
$client = new swoole_client(SWOOLE_SOCK_TCP); //同步阻塞
$client->connect('127.0.0.1', 9501) or die("connect failed\n");

$client->send(str_repeat("A", 600));
$data = $client->recv(700, 0) or die("recv failed\n");
echo "recv: " . $data . "\n";
~~~

## **在php-fpm/apache中创建长连接**
~~~php
$cli = new swoole_client(SWOOLE_TCP | SWOOLE_KEEP);
~~~

加入SWOOLE_KEEP标志后，创建的TCP连接在PHP请求结束或者调用`$cli->close`时并不会关闭。下一次执行connect调用时会复用上一次创建的连接。长连接保存的方式默认是以ServerHost:ServerPort为key的。可以再第3个参数内指定key。

* SWOOLE_KEEP只允许用于同步客户端
* $swoole_client->close(true)用于关闭长连接

> swoole_client在unset时会自动调用close方法关闭socket
> SWOOLE_KEEP长连接模式在1.6.12后可用，长连接的$key参数在1.7.5后增加

## 在php-fpm/apache中使用task功能
AsyncTask是swoole提供一套生产者消费者模型，可以方便地将一个慢速任务投递到队列，由进程池异步地执行。task功能目前只能在swoole_server中使用。1.9.0版本提供了RedisServer框架，可以基于RedisServer和Task实现一个Server程序，在php-fpm或apache中直接调用Redis扩展就可以使用swoole的task功能了。

创建RedisServer
~~~
use Swoole\Redis\Server;

$server = new Server("127.0.0.1", 9501, SWOOLE_BASE);

$server->set(array(
    'task_worker_num' => 32,
    'worker_num' => 1,
));

$server->setHandler('LPUSH', function ($fd, $data) use ($server) {
    $taskId = $server->task($data);
    if ($taskId === false)
    {
        return Server::format(Server::ERROR);
    }
    else
    {
        return Server::format(Server::INT, $taskId);
    }
});

$server->on('Finish', function() {

});

$server->on('Task', function ($serv, $taskId, $workerId, $data) {
    //处理任务
});

$server->start();
~~~
* 如果是本机调用可以监听UnixSocket，局域网内调用需要使用IP:PORT
* Task中$data就是客户端投递的数据
* 其他语言也可以使用Redis客户端投递任务
* 可以根据Task任务执行的速度调节task_worker_num控制启动的进程数量，这些进程是由swoole底层负责管理的，在发生致命错误或进程退出后底层会重新创建新的任务进程

**投递任务**
php-fpm/apache中投递任务
~~~php
$redis = new Redis;
$redis->connect('127.0.0.1', 9501);
$taskId = $redis->lpush("myqueue", json_encode(array("hello", "swoole")));
~~~
注意这个RedisServer并不是一台真正的Redis服务器，它只支持LPUSH一个指令。
