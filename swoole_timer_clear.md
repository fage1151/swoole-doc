## swoole_timer_clear
使用定时器ID来删除定时器。

~~~
bool swoole_timer_clear(int $timer_id)
~~~

* $timer_id，定时器ID，调用swoole_timer_tick、swoole_timer_after后会返回一个整数的ID
* swoole_timer_clear不能用于清除其他进程的定时器，只作用于当前进程
<?php
swoole

