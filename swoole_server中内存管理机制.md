# swoole_server中内存管理机制 

swoole_server启动后内存管理的底层原理与普通php-cli程序一致。具体请参考Zend VM内存管理方面的文章。

## 局部变量

在事件回调函数返回后，所有局部对象和变量会全部回收，不需要unset。如果变量是一个资源类型，那么对应的资源也会被PHP底层释放。

~~~
function test()
{
    $a = new Object;
    $b = fopen('/data/t.log', 'r+');
    $c = new swoole_client(SWOOLE_SYNC);
    $d = new swoole_client(SWOOLE_SYNC);
    global $e;
    $e['client'] = $d;
}
~~~
* $a, $b, $c 都是局部变量，当此函数return时，这3个变量会立即释放，对应的内存会理解释放，打开的IO资源文件句柄会立即关闭。
* $d 也是局部变量，但是return前将它保存到了全局变量$e，所以不会释放。当执行unset($e['client'])时，并且没有任何其他PHP变量仍然在引用$d变量，那么$d 就会被释放。

### 全局变量

在PHP中，有3类全局变量。

* 使用global关键词声明的变量
* 使用static关键词声明的类静态变量、函数静态变量
* PHP的超全局变量，包括$_GET、$_POST、$GLOBALS等

>全局变量和对象，类静态变量，保存在swoole_server对象上的变量不会被释放。需要程序员自行处理这些变量和对象的销毁工作。
>在swoole中如果想在内存中永久保存某些数据资源，可以将资源放到全局变量中或者类的静态成员中。


~~~
class Test
{
    static $array = array();
    static $string = '';
}

function onReceive($serv, $fd, $reactorId, $data)
{
    Test::$array[] = $fd;
    Test::$string .= $data;
}
~~~
* 在事件回调函数中需要特别注意非局部变量的array类型值，某些操作如 TestClass::$array[] = "string" 可能会造成内存泄漏，严重时可能发生爆内存，必要时应当注意清理大数组。

* 在事件回调函数中，非局部变量的字符串进行拼接操作是必须小心内存泄漏，如 TestClass::$string .= $data，可能会有内存泄漏，严重时可能发生爆内存。

## 解决方法
* 同步阻塞并且请求响应式无状态的Server程序可以设置max_request，当Worker进程/Task进程结束运行时或达到任务上限后进程自动退出。该进程的所有变量/对象/资源均会被释放回收。
* 程序内在onClose或设置定时器及时使用unset清理变量，回收资源
## 异步客户端
Swoole提供的异步客户端与普通的PHP变量不同，异步客户端在发起connect时底层会增加一次引用计数，在连接close时会减少引用计数。

>包括swoole_client、swoole_mysql、swoole_redis、swoole_http_client

~~~
function test()
{
    $client = new swoole_client(SWOOLE_TCP | SWOOLE_ASYNC);
    $client->on("connect", function($cli) {
        $cli->send("hello world\n");
    });
    $client->on("receive", function($cli, $data){
        echo "Received: ".$data."\n";
        $cli->close();
    });
    $client->on("error", function($cli){
        echo "Connect failed\n";
    });
    $client->on("close", function($cli){
        echo "Connection close\n";
    });
    $client->connect('127.0.0.1', 9501);
    return;
}
~~~
* $client是局部变量，常规情况下return时会销毁。
* 但这个$client是异步客户端在执行connect时swoole引擎底层会增加一次引用计数，因此return时并不会销毁。
* 该客户端执行onReceive回调函数时进行了close或者服务器端主动关闭连接触发onClose，这时底层会减少引用计数，$client才会被销毁。

## 异步回调程序内存管理
异步回调程序与同步阻塞程序的内存管理方式不同，异步程序是基于回调链引用计数实现内存的管理。本文会用一个最简单的实例讲解异步程序的内存管理。

实例程序
~~~
$serv = new Swoole\Http\Server("127.0.0.1", 9502);

$serv->on('Request', function($request, $response) {
    $cli = new Swoole\Http\Client('127.0.0.1', 80);
    $cli->post('/dump.php', array("key" => 'value'), function ($cli) use ($request, $response) {
        $response->end("<h1>{$cli->body}</h1>");
        $cli->close();
    });
});

$serv->start();
~~~
**onRequest**
* 请求到来这时会触发onRequest回调函数，可以得到$request和$response对象
在onRequest回调函数中，创建了一个Http\Client，并发起一次POST请求
* 然后onRequest函数结束并返回
* 这时按照正常的PHP函数调用流程，$request和$response对象会被销毁。但在上述程序中，$request和$response对象被使用了use语法，绑定到了匿名函数上，因此这2个对象的引用计数会被加1。onRequest函数返回时就不会真正销毁这2个对象了。

$cli对象，是在onRequest函数创建的局部变量，按照正常逻辑$cli对象在onRequest函数退出时也应该被销毁。但Swoole底层有一个特殊的逻辑，所有异步客户端对象在发起连接时底层会自动增加一次引用计数，在连接关闭时减少一次引用计数，因此$cli对象也不会销毁，POST请求中的匿名函数对象也不会销毁。

**Http响应**
* 创建的$cli对象，接收到来自服务器端的响应，或者连接超时、响应超时，这时会回调指定的匿名函数，调用end向客户端发送响应
* 回调函数中调用了$cli->close这时切断连接，$cli的引用计数减一。这时匿名函数退出底层会自动销毁$cli、$request、$response 3个对象

**多层嵌套**
如果Http\Client的回调函数中调用了其他的异步客户端，如Swoole\Redis，对象会继续传读引用，形成一个异步调用链。当调用链的最后一个对象销毁时会向着调用链头部逐个递减引用计数，最终销毁对象。

~~~
$serv = new Swoole\Http\Server("127.0.0.1", 9502);

$serv->on('Request', function($request, $response) {
    $cli = new Swoole\Http\Client('127.0.0.1', 80);
    //发起连接，$cli 引用计数增加
    $cli->post('/dump.php', array("key" => 'value'), function ($cli) use ($request, $response) {
        $redis = new Swoole\Redis;
        //发起连接，$redis 引用计数增加
        $redis->connect('127.0.0.1', 6379, function ($redis, $result) use ($request, $response, $cli) {
            $redis->get('test_key', function ($redis, $result) use ($request, $response, $cli) {
                $response->end("<h1>{$result}</h1>");
                //关闭连接，$cli 引用计数减少
                $cli->close();
                //关闭连接，$redis 引用计数减少
                $redis->close();
            });
        });
    });
});

$serv->start();
~~~
* 这里$response和$request对象被POST匿名函数、Redis->connect匿名函数、Redis->get匿名函数引用，因此需要等到这3个函数执行后，引用计数减少为0，才会真正的销毁
* $cli和$redis对象在发起TCP连接时，会被Swoole底层增加引用计数。只有$cli->close()和$redis->close被调用，或者远端服务器关闭连接，触发$cli->onClose和$redis->onClose，$cli和$redis这2个对象的，引用计数才会减少，函数退出时会销毁
* POST匿名函数、Redis->connect匿名函数、Redis->get匿名函数，3个对象依附于$cli和$redis对象，当$cli和$redis对象销毁时，这3个对象也会被销毁
* POST匿名函数、Redis->connect匿名函数、Redis->get匿名函数，匿名函数销毁时通过use语法引用的$response和$request对象也会销毁