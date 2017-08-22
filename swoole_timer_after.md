## swoole_timer_after
在指定的时间后执行函数，需要swoole-1.7.7以上版本。

~~~
swoole_timer_after(int $after_time_ms, mixed $callback_function, mixed $user_param);
~~~
swoole_timer_after函数是一个一次性定时器，执行完成后就会销毁。此函数与PHP标准库提供的sleep函数不同，after是非阻塞的。而sleep调用后会导致当前的进程进入阻塞，将无法处理新的请求。

* $after_time_ms 指定时间，单位为毫秒
* $callback_function 时间到期后所执行的函数，必须是可以调用的。
* $user_param 用户参数, 该参数会被传递到$callback_function中. 如果有多个参数可以使用数组形式. 也可以使用匿名函数的use语法传递参数到回调函数中

> $after_time_ms 最大不得超过 86400000

#### 使用示例
~~~
swoole_timer_after(1000, function(){
    echo "timeout\n";
});
~~~
