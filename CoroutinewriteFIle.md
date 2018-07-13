# Coroutine::writeFile
协程方式写入文件。

`function Coroutine::writeFile(string $filename, string $fileContent, int $flags);`

>需要2.1.2或更高版本

## 参数
* $filename为文件的名称，必须有可写权限，文件不存在会自动创建。打开文件失败会立即返回false
* $fileContent为要写入到文件的内容，最大可写入4M
* $flags为写入的选项，可以使用FILE_APPEND表示追加到文件末尾，默认会清空当前文件内容
## 返回值
* 写入成功返回true，写入失败返回false

## 示例
```
use Swoole\Coroutine as co;
$filename = __DIR__ . "/defer_client.php";
co::create(function () use ($filename)
{
    $r =  co::writeFile($filename,"hello swoole!");
    var_dump($r);
});
```