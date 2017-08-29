# swoole_async_dns_lookup_coro
协程DNS查询。函数原型：

~~~
function swoole_async_dns_lookup_coro(string $domain) : string | bool;
~~~
* 查询成功返回对应的IP地址
* 失败返回false，可使用swoole_errno和swoole_last_error得到错误信息

~~~
Swoole\Coroutine::create(function(){
    echo swoole_async_dns_lookup_coro('www.baidu.com');
});
~~~