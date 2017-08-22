## swoole_process::kill 
向进程发送信号

~~~
int swoole_process::kill($pid, $signo = SIGTERM);
~~~
* 默认的信号为SIGTERM，表示终止进程
* $signo=0，可以检测进程是否存在，不会发送信号
### 僵尸进程
子进程退出后，父进程务必要执行swoole_process::wait进行回收，否则这个子进程就会变为僵尸进程。会浪费操作系统的进程资源。

父进程可以设置监听SIGCHLD信号，收到信号后执行swoole_process::wait回收退出的子进程。
