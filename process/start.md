执行fork系统调用，启动进程。

    int swoole_process->start();
创建成功返回子进程的PID，创建失败返回false。可使用swoole_errno和swoole_strerror得到错误码和错误信息。

$process->pid 属性为子进程的PID
$process->pipe 属性为管道的文件描述符
执行后子进程会保持父进程的内存和资源，如父进程内创建了一个redis连接，那么在子进程会保留此对象，所有操作都是对同一个连接进行的。
注意事项
因为子进程会继承父进程的内存和IO句柄，所以如果父进程要创建多个子进程，务必要等待创建完毕后再使用swoole_event_add/异步swoole_client/定时器/信号等异步IO函数。
    
    <?php
    $process = new swoole_process('callback_function', true);
    //子进程执行的逻辑
    function callback_function(swoole_process $worker)
    {
        echo '子进程创建成功';
    }
    $ret = $process->start();
    echo '子进程进程号为'.$ret->pid;
    echo '管道的文件描述符为'.$ret->pipe;