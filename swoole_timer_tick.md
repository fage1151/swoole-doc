## swoole_timer_tick
设置一个间隔时钟定时器，与after定时器不同的是tick定时器会持续触发，直到调用swoole_timer_clear清除。

~~~
int swoole_timer_tick(int $ms, callable $callback, mixed $user_param);
~~~
* $ms 指定时间，单位为毫秒
* $callback_function 时间到期后所执行的函数，必须是可以调用的。
* $user_param 用户参数, 该参数会被传递到$callback_function中. 如果有多个参数可以使用数组形式. 也可以使用匿名函数的use语法传递参数到回调函数中
* 定时器仅在当前进程空间内有效
* 定时器是纯异步实现的，不能与阻塞IO的函数一起使用，否则定时器的执行时间会发生错乱