# Coroutine::fwrite
协程方式向文件写入数据。

`function Coroutine::fwrite(resource $handle, string $data, int $length = 0);`

>需要2.0.11或更高版本

## 参数
* $handle文件句柄，必须是fopen打开的文件类型stream资源
* $data要写入的数据内容，可以是文本或二进制数据
* $length写入的长度，默认为0，表示写入$data的全部内容，$length必须小于$data的长度
## 返回值
* 写入成功返回数据长度，失败返回false

## 示例
```
use Swoole\Coroutine as co;
$fp = fopen(__DIR__ . "/test.data", "a+");
co::create(function () use ($fp)
{
    $r =  co::fwrite($fp, "hello world\n", 5);
    var_dump($r);
});
```