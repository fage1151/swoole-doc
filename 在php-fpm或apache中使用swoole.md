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

> swoole_client在unset时会自动调用close方法关闭socket
> SWOOLE_KEEP长连接模式在1.6.12后可用，长连接的$key参数在1.7.5后增加