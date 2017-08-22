## swoole_async_readfile
异步读取文件内容

~~~
//函数风格
swoole_async_readfile(string $filename, mixed $callback);
//命名空间风格
Swoole\Async::readFile(string $filename, mixed $callback);
~~~

* 文件不存在会返回false
* 成功打开文件立即返回true
* 数据读取完毕后会回调指定的callback函数。

使用示例：
~~~
swoole_async_readfile(__DIR__."/server.php", function($filename, $content) {
     echo "$filename: $content";
});
~~~

> swoole_async_readfile会将文件内容全部复制到内存，所以不能用于大文件的读取
> 如果要读取超大文件，请使用swoole_async_read函数
> swoole_async_readfile最大可读取4M的文件，受限于SW_AIO_MAX_FILESIZE宏