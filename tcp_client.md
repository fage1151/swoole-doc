# swoole_client
swoole_client提供了tcp/udp socket的客户端的封装代码，使用时仅需 new swoole_client 即可。 swoole的socket client对比PHP提供的stream族函数有哪些好处：

* stream函数存在超时设置的陷阱和Bug，一旦没处理好会导致Server端长时间阻塞
* fread有8192长度限制，无法支持UDP的大包
* swoole_client支持waitall，在知道包长度的情况下可以一次取完，不必循环取。
* swoole_client支持UDP connect，解决了UDP串包问题
* swoole_client是纯C的代码，专门处理socket，stream函数非常复杂。swoole_client性能更好

除了普通的同步阻塞+select的使用方法外，swoole_client还支持异步非阻塞回调。

## 同步阻塞客户端

```php
$client = new swoole_client(SWOOLE_SOCK_TCP);
if (!$client->connect('127.0.0.1', 9501, -1))
{
    exit("connect failed. Error: {$client->errCode}\n");
}
$client->send("hello world\n");
echo $client->recv();
$client->close();
```
swoole_client支持长连接形式的同步客户端，方便在php-fpm环境中使用

```php
$cli = new swoole_client(SWOOLE_TCP | SWOOLE_KEEP);
```
**SWOOLE_KEEP只允许用于同步客户端**

加入SWOOLE_KEEP标志后，创建的TCP连接在PHP请求结束或者调用$cli->close时并不会关闭。下一次执行connect调用时会复用上一次创建的连接。长连接保存的方式默认是以ServerHost:ServerPort为key的(连接相同的host:port将复用已经存在的连接)。可以在第3个参数内指定key。
```php
swoole_client->__construct(int $sock_type, int $is_sync = SWOOLE_SOCK_SYNC, string $key);
```
**php-fpm/apache环境下只能使用同步客户端**

## 异步非阻塞客户端
```php
$client = new swoole_client(SWOOLE_SOCK_TCP, SWOOLE_SOCK_ASYNC);
$client->on("connect", function(swoole_client $cli) {
    $cli->send("GET / HTTP/1.1\r\n\r\n");
});
$client->on("receive", function(swoole_client $cli, $data){
    echo "Receive: $data";
    $cli->send(str_repeat('A', 100)."\n");
    sleep(1);
});
$client->on("error", function(swoole_client $cli){
    echo "error\n";
});
$client->on("close", function(swoole_client $cli){
    echo "Connection close\n";
});
$client->connect('127.0.0.1', 9501);
```
异步客户端只能使用在cli命令行环境 异步的swoole client的使用场景对于新手同学来说可能比较陌生，因为异步客户端是不可以应用在apache或fpm中的，而且仅能用于cli环境

## 并行调用
~~~
int swoole_client_select(array &$read, array &$write, array &$error, float $timeout);
~~~
swoole_client_select接受4个参数，$read, $write, $error 分别是可读/可写/错误的文件描述符。
这3个参数必须是数组变量的引用。数组的元素必须为swoole_client对象。 1.8.6或更高版本可以支持swoole_process对象
$timeout参数是select的超时时间，单位为秒，接受浮点数。

**此函数可以用于Apache/PHP-fpm环境**

~~~
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
