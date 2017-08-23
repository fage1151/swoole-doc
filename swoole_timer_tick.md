## swoole_timer_tick
设置一个间隔时钟定时器，与after定时器不同的是tick定时器会持续触发，直到调用swoole_timer_clear清除。

[TOC]

~~~
int swoole_timer_tick(int $ms, callable $callback, mixed $user_param);
~~~

* $ms 指定时间，单位为毫秒
* $callback_function 时间到期后所执行的函数，必须是可以调用的。
* $user_param 用户参数, 该参数会被传递到$callback_function中. 如果有多个参数可以使用数组形式. 也可以使用匿名函数的use语法传递参数到回调函数中
* 定时器仅在当前进程空间内有效
* 定时器是纯异步实现的，不能与阻塞IO的函数一起使用，否则定时器的执行时间会发生错乱

> $ms 最大不得超过 86400000
> tick定时器在1.7.14以上版本可用
> 定时器在执行的过程中可能会产生微小的偏差，请勿基于定时器实现精确时间计算

#### 回调函数
定时器触发的回调函数接受2个参数。

~~~
function callbackFunction(int $timer_id, mixed $params = null);
~~~
* $timer_id 定时器的ID，可用于swoole_timer_clear清除此定时器
* $params 由swoole_timer_tick传入的第三个参数

#### 定时器校正
定时器回调函数的执行时间不影响下一次定时器执行的时间。实例：在0.002ms设置了10ms的tick定时器，第一次会在0.012ms执行回调函数，如果回调函数执行了5ms，下一次定时器仍然会在0.022ms时触发，而不是0.027ms。

但如果定时器回调函数的执行时间过长，甚至覆盖了下一次定时器执行的时间。底层会进行时间校正，丢弃已过期的行为，在下一时间回调。如上面例子中0.012ms时的回调函数执行了15ms，本该在0.022ms产生一次定时回调。实际上本次定时器在0.027ms才返回，这时定时早已过期。底层会在0.032ms时再次触发定时器回调。

#### 使用示例
~~~
swoole_timer_tick(1000, function(){
    echo "timeout\n";
});
~~~
