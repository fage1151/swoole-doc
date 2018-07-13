# Coroutine::fread
协程方式读取文件。

`function Coroutine::fread(resource $handle, int $length = 0);`

>需要2.0.11或更高版本

## 参数
* $handle文件句柄，必须是fopen打开的文件类型stream资源
* $length读取的长度，默认为0，表示读取文件的全部内容
## 返回值
* 读取成功返回字符串内容，读取失败返回false

## 示例
```
use Swoole\Coroutine as co;
$fp = fopen(__DIR__ . "/defer_client.php", "r");
co::create(function () use ($fp)
{
    fseek($fp, 256);
    $r =  co::fread($fp);
    var_dump($r);
});
```