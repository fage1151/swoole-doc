# swoole_server->sendfile
发送文件到浏览器。

~~~
function swoole_http_response->sendfile(string $filename, int $offset = 0, int $length = 0);
~~~
* $filename 要发送的文件名称，文件不存在或没有访问权限sendfile会失败
* 底层无法推断要发送文件的MIME格式因此需要应用代码指定Content-Type
* 调用sendfile前不得使用write方法发送Http-Chunk
* 调用sendfile后底层会自动执行end
* sendfile不支持gzip压缩
* $offset 上传文件的偏移量，可以指定从文件的中间部分开始传输数据。此特性可用于支持断点续传。
* $length 发送数据的尺寸，默认为整个文件的尺寸

>$length、$offset参数在1.9.11或更高版本可用

使用示例
~~~
$response->header('Content-Type', 'image/jpeg');
$response->sendfile(__DIR__.$request->server['request_uri']);
~~~