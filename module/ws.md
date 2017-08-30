# ws client
发起WebSocket握手请求，并将连接升级为WebSocket。

~~~
function swoole_http_client->upgrade(string $path, callable $callback);
~~~
* $path URL路径
* $callback 握手成功或失败后回调此函数
* 使用Upgrade方法必须设置onMessage回调函数
使用实例
$cli = new swoole_http_client('127.0.0.1', 9501);

$cli->on('message', function ($_cli, $frame) {
    var_dump($frame);
});

$cli->upgrade('/', function ($cli) {
    echo $cli->body;
    $cli->push("hello world");
});
onMessage回调
function onMessage(swoole_http_client $client, swoole_websocket_frame $frame);
$client 客户端对象，可调用push方法向服务器发送数据
$frame WebSocket数据帧，可参考 swoole_websocket_server->onMessage
异步websocket客户端
```php
class Ws
{
    public $parse_host;
    public $parse_port;
    public $client;
    public $onMessage = null;
    public $onConnect = null;
    public $onClose = null;
    public $upgrade = null;

    public function __construct($host)
    {
        $this->onMessage = function(){};
        $this->upgrade = function(){};
        $this->onClose = function(){};
        $this->onConnect = function(){};
        $this->parse_url_to_array($host);
    }

    public function parse_url_to_array($url)
    {

        $parsed_arr = parse_url($url);
        $this->parse_host = isset($parsed_arr['host']) ? $parsed_arr['host'] : '127.0.0.1';
        $this->parse_port = isset($parsed_arr['port']) ? $parsed_arr['port'] : '80';

    }

    public function connect()
    {
        $this->client = $client = new \Swoole\Http\Client($this->parse_host, $this->parse_port);
        //设置事件回调函数
        $this->client->on("message", $this->onMessage);
        $this->client->on("connect", $this->onConnect);
        $this->client->on("close", $this->onClose);
        $this->client->upgrade('/', $this->upgrade);
    }
}
$url = 'laychat.workerman.net:9292';
$tcp = new Ws($url);
$tcp->onConnect = function (Client $client) {
    var_dump($client);
};
$tcp->onMessage = function (Client $client,$data) {
    $client->push('{"type":"ping"}');
    var_dump($data);
};
$tcp->connect();