## swoole_async_writefile
异步写文件，调用此函数后会立即返回。当写入完成时会自动回调指定的callback函数。

~~~
Swoole\Async::writeFile(string $filename, string $fileContent, callable $callback = null, int $flags = 0)
swoole_async_writefile('test.log', $file_content, function($filename) {
    echo "wirte ok.\n";
}, $flags = 0);
~~~