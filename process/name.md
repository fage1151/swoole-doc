修改进程名称。此函数是swoole_set_process_name的别名。
    
    bool swoole_process::name(string $new_process_name);

实例
    
    <?php
    $process = new swoole_process('callback_function', true);
    //子进程执行的逻辑
    function callback_function(swoole_process $worker)
    {
        $worker->name('child process')；
    }
    $pid = $process->start();
    swoole_set_process_name('parent process');