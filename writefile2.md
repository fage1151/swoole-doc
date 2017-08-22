## swoole_async_writefile
异步写文件，调用此函数后会立即返回。当写入完成时会自动回调指定的callback函数。

~~~
Swoole\Async::writeFile(string $filename, string $fileContent, callable $callback = null, int $flags = 0)
swoole_async_writefile('test.log', $file_content, function($filename) {
    echo "wirte ok.\n";
}, $flags = 0);
~~~
* 参数1为文件的名称，必须有可写权限，文件不存在会自动创建。打开文件失败会立即返回false
* 参数2为要写入到文件的内容，最大可写入4M
* 参数3为写入成功后的回调函数，可选
* 参数4为写入的选项，可以使用FILE_APPEND表示追加到文件末尾

> FILE_APPEND在1.9.1或更高版本可用
> Linux原生异步IO不支持FILE_APPEND，并且写入的内容长度必须为4096的整数倍，否则底层会自动在末尾填充0