#Coroutine::fgets
协程方式按行读取文件内容。

`function Coroutine::fgets(resource $handle);`
* Co::fgets底层使用了php_stream缓存区，默认大小为8192字节，可使用stream_set_chunk_size设置缓存区尺寸。

>需要2.1.1或更高版本

## 参数
* $handle文件句柄，必须是fopen打开的文件类型stream资源
## 返回值
* 读取到EOL（\r或\n）将返回一行数据，包括EOL
* 未读取到EOL，但内容长度超过php_stream缓存区8192字节，将返回8192字节的数据，不包含EOL
* 达到文件末尾EOF时，返回空字符串，可用feof判断文件是否已读完
* 读取失败返回false，使用swoole_last_error函数获取错误码
## 示例
```
$fp = fopen(__DIR__ . "/defer_client.php", "r");
go(function () use ($fp)
{
    $r =  co::fgets($fp);
    var_dump($r);
});
```