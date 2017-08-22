## swoole_process::alarm
高精度定时器
swoole_process::alarm是操作系统setitimer系统调用的封装，可以设置微秒级别的定时器。定时器会触发信号，需要与swoole_process::signal或pcntl_signal配合使用。

~~~
function swoole_process::alarm(int $interval_usec, int $type = ITIMER_REAL) : bool
~~~

* $interval_usec 定时器间隔时间，单位为微妙。如果为负数表示清除定时器
* $type 定时器类型，0 表示为真实时间,触发SIGALAM信号，1 表示用户态CPU时间，触发SIGVTALAM信号，2 表示用户态+内核态时间，触发SIGPROF信号
* 设置成功返回true，失败返回false，可以使用swoole_errno得到错误码

> alarm不能和Swoole\Timer同时使用
> alarm在1.8.13或更高版本可用

~~~
swoole_process::signal(SIGALRM, function () {
    static $i = 0;
    echo "#{$i}\talarm\n";
    $i++;
    if ($i > 20) {
        swoole_process::alarm(-1);
    }
});

//100ms
swoole_process::alarm(100 * 1000);
~~~