# Coroutine::stats
获取协程状态

`function \Swoole\Coroutine::stats() : array`

>需要4.0.1或更高版本

## 返回值 array
* coroutine_num： 当前运行的协程数量
```
var_dump(Swoole\Coroutine::stats());

array(1) {
  ["coroutine_num"]=>
  int(132)
}
```