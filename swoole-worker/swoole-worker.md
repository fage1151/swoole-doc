此项目是workerman(v3.4.5)的swoole移植版本，移除了对pcntl,libevent,event,ev扩展的依赖,转而使用swoole提供的swoole_process和swoole_event，定时器采用swoole的swoole_timer,server采用stream扩展
使用可以参考[workerman文档](http://doc.workerman.net/)
