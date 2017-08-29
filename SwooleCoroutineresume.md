# Swoole\Coroutine::resume

恢复某个协程，使其继续运行。

~~~
function Swoole\Coroutine::resume(string $coroutineId);
~~~
* 参数$coroutineId为要恢复的协程ID，在协程内可以使用getuid获取到协程的ID
* 当前协程处于挂起状态时，另外的协程中可以使用resume再次唤醒当前协程