# Coroutine::create [编辑本页]
创建一个新的协程，并立即执行。

~~~
function Swoole\Coroutine::create(callable $function);
~~~
* $function 协程执行的代码
* 创建成功返回true，失败返回false
* 系统能创建的协程总数量受限于server->max_coroutine设置。

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