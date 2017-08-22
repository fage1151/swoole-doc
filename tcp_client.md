# swoole_client
swoole_client提供了tcp/udp socket的客户端的封装代码，使用时仅需 new swoole_client 即可。 swoole的socket client对比PHP提供的stream族函数有哪些好处：

stream函数存在超时设置的陷阱和Bug，一旦没处理好会导致Server端长时间阻塞
fread有8192长度限制，无法支持UDP的大包
swoole_client支持waitall，在知道包长度的情况下可以一次取完，不必循环取。
swoole_client支持UDP connect，解决了UDP串包问题
swoole_client是纯C的代码，专门处理socket，stream函数非常复杂。swoole_client性能更好
除了普通的同步阻塞+select的使用方法外，swoole_client还支持异步非阻塞回调。
同步阻塞客户端
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
异步非阻塞客户端
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