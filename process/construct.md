创建子进程
    
~~~
swoole_process::__construct(callable $function, $redirect_stdin_stdout = false, $create_pipe = true);
~~~
参数说明　　
$function，子进程被创建以后执行的函数，底层会自动将函数保存到对象的callback属性上。如果希望更改执行的函数，可赋值新的函数到对象的callback属性  
$redirect_stdin_stdout，重定向子进程的标准输入和输出。启用此选项后，在子进程内输出内容将不是打印屏幕，而是写入到主进程管道。读取键盘输入将变为从管道中读取数据。默认为阻塞读取。  
$create_pipe，是否创建管道，启用$redirect_stdin_stdout后，此选项将忽略用户参数，强制为true。如果子进程内没有进程间通信，可以设置为 false
    
~~~
<?php
$process = new swoole_process('callback_function', true);
//子进程执行的逻辑
function callback_function(swoole_process $worker)
{
    swoole_timer_tick(2000,function(){
        echo time();
    })
}
$pid = $process->start();
//父进程执行的逻辑
//监听子进程退出信号
swoole_process::signal(SIGCHLD, function($sig) {
  //必须为false，非阻塞模式
  while($ret =  swoole_process::wait(false)) {
      //执行回收后的处理逻辑，比如拉起一个新的进程
  }
});
~~~