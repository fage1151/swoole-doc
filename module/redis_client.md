## redis client
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
### 构造函数
~~~
function swoole_redis->__construct(array $options = null);
~~~
* 超时控制 $options['timeout'] = 1.5,浮点型，单位为秒，最小粒度为1毫秒
* 设置密码 $options['password'] = 'passwd',等同于auth指令
* 设置数据库 $options['database'] = 0,等同于select指令