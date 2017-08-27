# 如何实现异步任务

**问：**

如何异步处理繁重的业务，避免主业务被长时间阻塞。例如我要给1000用户发送邮件，这个过程很慢，可能要阻塞数秒，这个过程中因为主流程被阻塞，会影响后续的请求，如何将这样的繁重任务交给其它进程异步处理。

**答:**

可以在本机或者其它服务器甚至服务器集群预先建立一些任务进程处理繁重的业务，任务进程数可以开多一些，例如cpu的10倍，然后调用方利用AsyncTcpConnection将数据异步发送给这些任务进程异步处理，异步得到处理结果。

**swoole支持处理异步任务**

投递一个异步任务到task_worker池中。此函数是非阻塞的，执行完毕会立即返回。Worker进程可以继续处理新的请求。使用Task功能，必须先设置 task_worker_num，并且必须设置Server的onTask和onFinish事件回调函数。

~~~php
int swoole_server::task(mixed $data, int $dst_worker_id = -1) 
$task_id = $serv->task("some data");
//swoole-1.8.6或更高版本
$serv->task("taskcallback", -1, function (swoole_server $serv, $task_id, $data) {
    echo "Task Callback: ";
    var_dump($task_id, $data);
});
~~~

此功能用于将慢速的任务异步地去执行，比如一个聊天室服务器，可以用它来进行发送广播。当任务完成时，在task进程中调用$serv->finish("finish")告诉worker进程此任务已完成。当然swoole_server->finish是可选的。

task底层使用Unix Socket管道通信，是全内存的，没有IO消耗。单进程读写性能可达100万/s，不同的进程使用不同的管道通信，可以最大化利用多核。

task_worker的数量在swoole_server::set参数中调整，如task_worker_num => 64，表示启动64个进程来接收异步任务

这样，繁重的任务交给本机或者其它服务器的进程去做，任务完成后会异步收到结果，业务进程就不会阻塞了。