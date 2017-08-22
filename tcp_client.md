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

别名swoole_select，swoole_client_select接受4个参数。

* $read, $write, $error 分别是可读/可写/错误的文件描述符。这3个参数必须是数组变量的引用。数组的元素必须为swoole_client对象。 1.8.6或更高版本可以支持swoole_process对象
* $timeout参数是select的超时时间，单位为秒，接受浮点数。

返回值

* 调用成功后，会返回事件的数量，并修改$read/$write/$error数组。使用foreach遍历数组，然后执行$item->recv/$item->send来收发数据。或者调用$item->close()或unset($item)来关闭socket。

* swoole_client_select返回0表示在规定的时间内，没有任何IO可用，select调用已超时。

**此函数可以用于Apache/PHP-fpm环境**

### swoole_client用法
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
### swoole_process用法
~~~
<?php
$process = new swoole_process(function (swoole_process $worker)
{
    echo "Worker: start. PID=" . $worker->pid . "\n";
    sleep(2);
    $worker->write("hello master\n");
    $worker->exit(0);
}, false);

$pid = $process->start();
$r = array($process);
$write = $error = array();
$ret = swoole_select($r, $write, $error, 1.0);//swoole_select是swoole_client_select的别名
var_dump($ret);
var_dump($process->read());
~~~

### 配置选项 
Swoole\Client和Swoole\Http\Client可以使用set方法设置一些选项，启用某些特性。

结束符检测
~~~
$client->set(array(
    'open_eof_check' => true,
    'package_eof' => "\r\n\r\n",
    'package_max_length' => 1024 * 1024 * 2,
))
~~~
长度检测
~~~
$client->set(array(
    'open_length_check'     => 1,
    'package_length_type'   => 'N',
    'package_length_offset' => 0,       //第N个字节是包长度的值
    'package_body_offset'   => 4,       //第几个字节开始计算长度
    'package_max_length'    => 2000000,  //协议最大长度
));
~~~
Socket缓存区尺寸
~~~
$client->set(array(
    'socket_buffer_size'     => 1024*1024*2, //2M缓存区
));
~~~
包括socket底层操作系统缓存区、应用层接收数据内存缓存区、应用层发送数据内存缓冲区
关闭Nagle合并算法
~~~
$client->set(array(
    'open_tcp_nodelay'     =>  true,
));
~~~
SSL/TLS证书
~~~
$client->set(array(
    'ssl_cert_file'     =>  $your_ssl_cert_file_path,
    'ssl_key_file'     =>  $your_ssl_key_file_path,
));
~~~
swoole-1.7.21或更高版本可用
绑定IP和端口
机器有多个网卡的情况下，设置bind_address参数可以强制客户端Socket绑定某个网络地址。
设置bind_port可以使客户端Socket使用固定的端口连接到外网服务器
~~~
$client->set(array(
    'bind_address'     =>  '192.168.1.100',
    'bind_port'     =>  36002,
));
~~~
swoole-1.8.5或更高版本可用
Socks5代理设置
~~~
$client->set(array(
    'socks5_host'     =>  '192.168.1.100',
    'socks5_port'     =>  1080,
    'socks5_username' => 'username',
    'socks5_password' => 'password',
));
~~~

socks5_username、socks5_password为可选参数
Http代理设置

~~~
$client->set(array(
    'http_proxy_host'     =>  '192.168.1.100',
    'http_proxy_port'     =>  1080,
));
~~~
使用说明
目前支持open_length_check和open_eof_check2种自动协议处理功能，参考swoole_server中的配置选项
启用了自动协议后，同步阻塞客户端recv方法将不接受长度参数，每次必然返回一个完整的数据包
启用了自动协议后，异步非阻塞客户端onReceive每次必然返回一个完整的数据包