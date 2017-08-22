设置异步信号监听。  
**swoole_process::signal在swoole-1.7.9以上版本可用**

* 此方法基于signalfd和eventloop是异步IO，不能用于同步程序中
* 同步阻塞的程序可以使用pcntl扩展提供的pcntl_signal
* $callback如果为null，表示移除信号监听
* 如果已设置了此信号的回调函数，重新设置时会覆盖历史设置

父进程得到子进程退出的信号，并回收子进程

~~~
swoole_process::signal(SIGCHLD, function($sig) {
  //必须为false，非阻塞模式
  while($ret =  swoole_process::wait(false)) {
      //执行回收后的处理逻辑，比如拉起一个新的进程
  }
});
~~~
    
 查看linux下的信号列表
```
kill -l
```