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
## 在php-fpm/apache中使用swoole_client_select实现并行处理
swoole_client的并行处理中用了select来做IO事件循环。

函数原型：

~~~
int swoole_client_select(array &$read, array &$write, array &$error, float $timeout);
~~~
* swoole_client_select接受4个参数，$read, $write, $error 分别是可读/可写/错误的文件描述符。
* 这3个参数必须是数组变量的引用。数组的元素必须为swoole_client对象。 1.8.6或更高版本可以支持swoole_process对象
* $timeout参数是select的超时时间，单位为秒，接受浮点数。
* 调用成功后，会返回事件的数量，并修改$read/$write/$error数组。使用foreach遍历数组，然后执行$item->recv/$item->send来收发数据。或者调用$item->close()或unset($item)来关闭socket。
* swoole_client_select返回0表示在规定的时间内，没有任何IO可用，select调用已超时。
swoole_client用法
~~~php
$clients = array();

for($i=0; $i< 20; $i++)
{
    $client = new swoole_client(SWOOLE_SOCK_TCP, SWOOLE_SOCK_SYNC); //同步阻塞
    $ret = $client->connect('127.0.0.1', 9501, 0.5, 0);
    if(!$ret)
    {
        echo "Connect Server fail.errCode=".$client->errCode;
    }
    else
    {
        $client->send("HELLO WORLD\n");
        $clients[$client->sock] = $client;
    }
}

while (!empty($clients))
{
    $write = $error = array();
    $read = array_values($clients);
    $n = swoole_client_select($read, $write, $error, 0.6);
    if ($n > 0)
    {
        foreach ($read as $index => $c)
        {
            echo "Recv #{$c->sock}: " . $c->recv() . "\n";
            unset($clients[$c->sock]);
        }
    }
}
~~~