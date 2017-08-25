# EOF结束符协议
EOF协议处理的原理是每个数据包结尾加一串特殊字符表示包已结束。如memcache、ftp、stmp都使用\r\n作为结束符。发送数据时只需要在包末尾增加\r\n即可。使用EOF协议处理，一定要确保数据包中间不会出现EOF，否则会造成分包错误。

在swoole_server和swoole_client的代码中只需要设置2个参数就可以使用EOF协议处理。

~~~
$server->set(array(
    'open_eof_split' => true,
    'package_eof' => "\r\n",
));
$client->set(array(
    'open_eof_split' => true,
    'package_eof' => "\r\n",
));
~~~