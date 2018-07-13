# Coroutine::create
创建一个新的协程，并立即执行。

~~~
function Swoole\Coroutine::create(callable $function);
~~~
* $function 协程执行的代码
* 创建成功返回true，失败返回false
* 系统能创建的协程总数量受限于server->max_coroutine设置。

在2.1.0或更高版本中如果开启了`swoole.use_shortname`，可以直接使用go关键词创建新的协程。

~~~
go(function () {
    $db = new Co\MySQL();
    $server = array(
        'host' => '127.0.0.1',
        'user' => 'root',
        'password' => 'root',
        'database' => 'test',
    );

    $db->connect($server);

    $result = $db->query('SELECT * FROM userinfo WHERE id = 3');
    var_dump($result);
});
~~~

**使用示例**

~~~php
<?php
for($i = 0; $i < 3; $i++) {
    Swoole\Coroutine::create(function() use ($i) {
        $cli = new Swoole\Coroutine\Http\Client('www.baidu.com', 80);
        $cli->setHeaders([
            'Host' => "www.baidu.com",
            "User-Agent" => 'Chrome/49.0.2587.3',
            'Accept' => 'text/html,application/xhtml+xml,application/xml',
            'Accept-Encoding' => 'gzip',
        ]);

        $cli->get('/');
        echo $cli->body;
        $cli->close();
    });
}
~~~