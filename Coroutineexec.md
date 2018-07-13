# Coroutine::exec
执行一条shell指令。底层自动进行协程调度。

`function Coroutine::exec(string $cmd) : array;`

* $cmd 要执行的shell指令
## 返回值
执行失败返回false，执行成功返回数组，包含了进程退出的状态码、信号、输出内容。
```
array(
    'code' => 0,
    'signal' => 0,
    'output' => '',
);
```

## 使用实例
```
go(function() {
    $ret = Co::exec("md5sum ".__FILE__);
});
```