## swoole_async_write
异步写文件，与swoole_async_writefile不同，swoole_async_write是分段写的。不需要一次性将要写的内容放到内存里，所以只占用少量内存。swoole_async_write通过传入的offset参数来确定写入的位置。

~~~
bool swoole_async_write(string $filename, string $content, int $offset = -1, mixed $callback = NULL);
~~~

* 当offset为-1时表示追加写入到文件的末尾
* Linux原生异步IO不支持追加模式，并且$content的长度和$offset必须为512的整数倍。如果传入错误的字符串长度或者$offset写入会失败，并且错误码为EINVAL