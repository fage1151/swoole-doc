```php
$swoole_mysql = new Swoole\Coroutine\MySQL();
$swoole_mysql->connect([
    'host' => '127.0.0.1',
    'port' => 3306,
    'user' => 'user',
    'password' => 'pass',
    'database' => 'test',
]);
$res = $swoole_mysql->query('select sleep(1)');