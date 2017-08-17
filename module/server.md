## tcp server
```php
<?php
$serv = new swoole_server('127.0.0.1', 9501);



$serv->on('connect', function ($serv, $fd) {
    echo "Client:Connect.\n";
});


$serv->on('receive', function ($serv, $fd, $from_id, $data) {
    $serv->send($fd, $data);
});

$serv->on('close', function ($serv, $fd) {
    echo "Client: Close.\n";
});

// Start our server, listen on the port and be ready to accept connections.
$serv->start();