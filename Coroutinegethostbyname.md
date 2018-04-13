# Coroutine::gethostbyname


将域名解析为IP，基于同步的线程池模拟实现。底层自动进行协程调度。

~~~
function Coroutine::gethostbyname(string $domain, int $family = AF_INET): string | bool

~~~
* $domain域名，如www.baidu.com
* $family默认为AF_INET表示返回IPv4地址，使用AF_INET6时返回IPv6地址
成功返回域名对应的IP地址，失败返回false
示例
~~~
use Swoole\Coroutine as co;
$ip = co::gethostbyname("www.baidu.com");
~~~
>协程DNS查询。与co::gethostbyname不同，swoole_async_dns_lookup_coro是基于UDP客户端实现。不支持/etc/hosts配置。