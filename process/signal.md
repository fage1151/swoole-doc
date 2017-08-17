设置异步信号监听。  

父进程得到子进程退出的信号，并回收子进程

    swoole_process::signal(SIGCHLD, function($sig) {
      //必须为false，非阻塞模式
      while($ret =  swoole_process::wait(false)) {
          //执行回收后的处理逻辑，比如拉起一个新的进程
      }
    });