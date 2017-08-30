# http client 
Swoole-1.8.0版本增加了对异步Http/WebSocket客户端的支持。底层是用纯C编写，拥有超高的性能。

**启用Http客户端**
* 1.8.6版本之前，需要在编译swoole时增加--enable-async-httpclient来开启此功能。
* swoole_http_client不依赖任何第三方库
* 支持Http-Chunk、Keep-Alive、form-data
* Http协议版本为HTTP/1.1
* gzip压缩格式支持需要依赖zlib库

**构造方法**
~~~
function swoole_http_client->__construct(string $ip, int port, bool $ssl = false);
~~~
* $ip 目标服务器的IP地址，可使用swoole_async_dns_lookup查询域名对应的IP地址
* $port 目标服务器的端口，一般http为80，https为443
* $ssl 是否启用SSL/TLS隧道加密，如果目标服务器是https必须设置$ssl参数为true

**对象属性**
* $body 请求响应后服务器端返回的内容
* $statusCode 服务器端返回的Http状态码，如404、200、500等

**swoole_http_client->set**
设置客户端参数，此方法与Swoole\Client->set接收的参数完全一致，可参考 Swoole\Client->set 方法的文档。

除了设置TCPSocket的参数之外，Swoole\Http\Client 额外增加了一些选项，来控制Http和WebSocket客户端。

**超时控制**
设置timeout选项，启用Http请求超时检测。单位为秒，最小粒度支持毫秒。

~~~
$http->set(['timeout' => 3.0]);
~~~
* 连接超时或被服务器关闭连接，statusCode将设置为-1
* 在约定的时间内服务器未返回响应，请求超时，statusCode将设置为-2
* 请求超时后底层会自动切断连接
* 设置为-1表示永不超时，底层将不会添加超时检测的定时器

>仅在1.9.14或更高版本可用

**keep_alive**
设置keep_alive选项，启用或关闭Http长连接。


~~~
$http->set(['keep_alive' => false]);
~~~
**websocket_mask**
WebSocket客户端启用或关闭掩码。默认为关闭。启用后会对WebSocket客户端发送的数据使用掩码进行数据转换。

~~~
$http->set(['websocket_mask' => true]);
~~~
## **swoole_http_client->setMethod**
设置Http请求方法

~~~
function swoole_http_client->setMethod(string $method);
$client->setMethod("PUT");
~~~
* $method 必须为符合Http标准的方法名称，如果$method设置错误可能会被Http服务器拒绝请求
* setMethod仅在当前请求有效，发送请求后会理解清除method设置

**swoole_http_client->setData**
设置Http请求的包体

~~~
function swoole_http_client->setData(string $data);
~~~
* $data 为字符串格式
* 设置$data后并且未设置$method，底层会自动设置为POST
* 未设置Http请求包体并且未设置$method，底层会自动设置为GET

**异步http客户端**
```php
<?php
class Http
{
    public $parse_scheme;
    public $parse_host;
    public $parse_port;
    public $parse_path;
    public $parse_query;
    public $real_ip;
    public $client;
    public $request_headers = [];
    public $request_cookies = [];
    public $request_data = [];
    public $request_method = '';
    public $onResponse = null;
    public $onError = null;
    public $ssl = false;
    public function __construct($url,$method='get',$headers=[],$cookies=[])
    {
        $this->parse_url_to_array($url);
        $available_methods = ['post','get'];
        if(!in_array($method,$available_methods)){
            throw new \Exception('request method is inavailable');
        }
        $this->request_headers = $headers;
        $this->request_cookies = $cookies;
        $this->request_method = $method;
        $this->onError = function(){};
        $this->onResponse = function(){};

    }
    public function parse_url_to_array($url)
    {

        $parsed_arr = parse_url($url);
        $this->parse_scheme = isset($parsed_arr['scheme']) ? $parsed_arr['scheme'] : 'http';
        $this->parse_host = isset($parsed_arr['host']) ? $parsed_arr['host'] : '127.0.0.1';
        $this->parse_port = isset($parsed_arr['port']) ? $parsed_arr['port'] : $this->parse_scheme == 'https'?'443':'80';
        $this->parse_path = isset($parsed_arr['path']) ? $parsed_arr['path'] : '/';
        $this->parse_query = isset($parsed_arr['query']) ? $parsed_arr['query'] : '';

    }

    public function request($data=[])
    {
        $this->request_data = $data;
        swoole_async_dns_lookup($this->parse_host, function($host, $ip){
            if($ip == ''){
                call_user_func_array($this->onError,[$host]);
                return;
            }
            $this->real_ip = $ip;
            if($this->parse_scheme === 'https'){
                $this->ssl = true;
            };
            $this->client = new \Swoole\Http\Client($this->real_ip, $this->parse_port,$this->ssl);
            $this->client->setHeaders($this->request_headers);
            $this->client->setCookies($this->request_cookies);
            $request_method = $this->request_method;
            if($request_method == 'post'){
                $this->client->post($this->parse_path.'?'.$this->parse_query,$this->request_data,$this->onResponse);
            }else{
                $this->client->get($this->parse_path.'?'.$this->parse_query,$this->onResponse);
            }
        });
    }
}
$url = 'http://www.workerman.net';
$request_method = 'get';
$data = ['uid'=>1];
$http = new Http($url, $request_method);
$http->onResponse = function ($cli) {
    var_dump($cli->body);
};
$http->request($data);
```