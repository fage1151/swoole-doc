# http client 
Swoole-1.8.0版本增加了对异步Http/WebSocket客户端的支持。底层是用纯C编写，拥有超高的性能。

[TOC]

**启用Http客户端**
* 1.8.6版本之前，需要在编译swoole时增加--enable-async-httpclient来开启此功能。
* swoole_http_client不依赖任何第三方库
* 支持Http-Chunk、Keep-Alive、form-data
* Http协议版本为HTTP/1.1
* gzip压缩格式支持需要依赖zlib库

## **构造方法**
~~~
function swoole_http_client->__construct(string $ip, int port, bool $ssl = false);
~~~
* $ip 目标服务器的IP地址，可使用swoole_async_dns_lookup查询域名对应的IP地址
* $port 目标服务器的端口，一般http为80，https为443
* $ssl 是否启用SSL/TLS隧道加密，如果目标服务器是https必须设置$ssl参数为true

**对象属性**
* $body 请求响应后服务器端返回的内容
* $statusCode 服务器端返回的Http状态码，如404、200、500等

## **swoole_http_client->set**
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
## **swoole_http_client->setHeaders**
设置Http请求头

~~~
function swoole_http_client->setHeaders(array $headers);
~~~
* $headers必须为键值对应的数组，底层会自动映射为$key: $value格式的Http标准头格式
* setHeaders设置的Http头在swoole_http_client对象存活期间的每次请求永久有效
* 重新调用setHeaders会覆盖上一次的设置

## **swoole_http_client->setCookies**
设置Cookie

~~~
function swoole_http_client->setCookies(array $cookies);

~~~
* $cookies 设置COOKIE，必须为键值对应数组
* 设置COOKIE后在客户端对象存活期间会持续保存
* 服务器端主动设置的COOKIE会合并到cookies数组中，可读取$client->cookies属性获得当前Http客户端的COOKIE信息

## **swoole_http_client->setData**
设置Http请求的包体

~~~
function swoole_http_client->setData(string $data);
~~~
* $data 为字符串格式
* 设置$data后并且未设置$method，底层会自动设置为POST
* 未设置Http请求包体并且未设置$method，底层会自动设置为GET
## **swoole_http_client->addFile**
添加POST文件。


~~~
function swoole_http_client->addFile(string $path, string $name, string $filename = null, string $mimeType = null, int $offset = 0, int $length)

~~~
* $path 文件的路径，必选参数，不能为空文件或者不存在的文件
* $name 表单的名称，必选参数，FILES参数中的key
* $filename 文件名称，可选参数，默认为basename($path)
* $mimeType 文件的MIME格式，可选参数，底层会根据文件的扩展名自动推断
* $offset 上传文件的偏移量，可以指定从文件的中间部分开始传输数据。此特性可用于支持断点续传。
* $length 发送数据的尺寸，默认为整个文件的尺寸

使用addFile会自动将POST的Content-Type将变更为form-data。addFile底层基于sendfile，可支持异步发送超大文件。

>addFile在1.8.9或更高版本可用
>$offset, $length 参数在1.9.11或更高版本可用

**使用示例**
```
<?php
$cli = new swoole_http_client('127.0.0.1', 80);
//post request
$cli->setHeaders(['User-Agent' => "swoole"]);
$cli->addFile(__DIR__.'/post.data', 'post');
$cli->addFile(dirname(__DIR__).'/test.jpg', 'debug');
$cli->post('/dump2.php', array("xxx" => 'abc', 'x2' => 'rango'), function ($cli) {
    echo $cli->body;
});
```
## **swoole_http_client->get**
发起GET请求，函数原型：

~~~
function swoole_http_client->get(string $path, callable $callback);
~~~
* $path 设置URL路径，如/index.html，注意这里不能传入http://domain
* $callback 调用成功或失败后回调此函数
* 默认使用GET方法，可使用setMethod设置新的请求方法
* Http响应内容会在内存中进行数据拼接。因此如果响应体很大可能会占用大量内存

**使用实例**
~~~
$cli = new swoole_http_client('127.0.0.1', 80);

$cli->setHeaders([
    'Host' => "localhost",
    "User-Agent" => 'Chrome/49.0.2587.3',
    'Accept' => 'text/html,application/xhtml+xml,application/xml',
    'Accept-Encoding' => 'gzip',
]);

$cli->get('/index.php', function ($cli) {
    echo "Length: " . strlen($cli->body) . "\n";
    echo $cli->body;
});

~~~
## **swoole_http_client->post**
发起POST请求，函数原型：

~~~
function swoole_http_client->post(string $path, mixed $data, callable $callback);

~~~
* $path 设置URL路径，如/index.html，注意这里不能传入http://domain
* $data 请求的包体数据，如果$data为数组底层自动会打包为x-www-form-urlencoded格式的POST内容，并设置Content-Type为application/x-www-form-urlencoded
* $callback 调用成功或失败后回调此函数
* 默认使用POST方法，可使用setMethod设置新的方法

**使用实例**
```
$cli = new swoole_http_client('127.0.0.1', 80); 
$cli->post('/post.php', array("a" => '1234', 'b' => '456'), function ($cli) {
    echo "Length: " . strlen($cli->body) . "\n";
    echo $cli->body;
});
```
## **swoole_http_client->execute**
更底层的Http请求方法，需要代码中调用setMethod和setData等接口设置请求的方法和数据。


~~~
function swoole_http_client->execute(string $path, callable $callback);
~~~
## **swoole_http_client->download**
通过Http下载文件。download与get方法的不同是download收到数据后会写入到磁盘，而不是在内存中对Http Body进行拼接。因此download仅使用小量内存，就可以完成超大文件的下载。 函数原型：

~~~
function swoole_http_client->download(string $path, string $filename, callable $callback, int $offset = 0);

~~~
* $path URL路径
* $filename 指定下载内容写入的文件路径，会自动写入到downloadFile属性
* $callback 下载成功后的回调函数
* $offset 指定写入文件的偏移量，此选项可用于支持断点续传，可配合Http头Range:bytes=$offset-实现
* $offset为0时若文件已存在，底层会自动清空此文件
* 执行成功返回true
* 打开文件失败或feek失败返回false
**使用示例**
```
$cli = new swoole_http_client('127.0.0.1', 80);

$cli->setHeaders([
    'Host' => "localhost",
    "User-Agent" => 'Chrome/49.0.2587.3',
    'Accept' => '*',
    'Accept-Encoding' => 'gzip',
]);

$cli->download('/video.avi', __DIR__.'/video.avi', function ($cli) {
    var_dump($cli->downloadFile);
});
```
**断点续传**
```
$cli = new swoole_http_client('127.0.0.1', 80);
$file = __DIR__.'/video.avi';
$offset = filesize($file);
$cli->setHeaders([
    'Host' => "localhost",
    "User-Agent" => 'Chrome/49.0.2587.3',
    'Accept' => '*',
    'Accept-Encoding' => 'gzip',
    'Range' => "bytes=$offset-",
]);

$cli->download('/video.avi', $file, function ($cli) {
    var_dump($cli->downloadFile);
}, $offset);
```
#3 **swoole_http_client->close**
关闭连接，函数原型为：

~~~
function swoole_http_client->close() : bool

~~~
* 操作成功返回 true

>swoole_http_client与普通的swoole_client不同，close后如果再次请求get、post等方法时，底层会重新连接服务器

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