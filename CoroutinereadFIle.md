# Coroutine::readFile
协程方式读取文件。

`function Coroutine::readFile(string $filename);`

>需要2.1.2或更高版本

## 参数
* $filename文件名
## 返回值
* 读取成功返回字符串内容，读取失败返回false
* readFile方法没有尺寸限制，读取的内容会存放在内存中，因此读取超大文件时可能会占用过多内存
## 示例

```
use Swoole\Coroutine as co;
$filename = __DIR__ . "/defer_client.php";
co::create(function () use ($filename)
{
    $r =  co::readFile($filename);
    var_dump($r);
});
```