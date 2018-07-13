

# Coroutine::getaddrinfo
进行DNS解析，查询域名对应的IP地址，与gethostbyname不同，getaddrinfo支持更多参数设置，而且会返回多个IP结果。

`function Coroutine::getaddrinfo(string $domain, int $family = AF_INET, int $socktype = SOCK_STREAM,
    int $protocol = IPPROTO_TCP, string $service = null): array | bool`

* $domain域名，如www.baidu.com
* $family默认为AF_INET表示返回IPv4地址，使用AF_INET6时返回IPv6地址
* 其他参数设置请参考man getaddrinfo文档
* 成功返回多个IP地址组成的数组，失败返回false
## 示例
`$array = co::getaddrinfo("www.baidu.com");`

