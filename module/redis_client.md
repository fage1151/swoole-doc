## redis client
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