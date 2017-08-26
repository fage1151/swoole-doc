# Task Worker

---

<!-- toc -->

## 简介

Task Worker是Swoole中一种特殊的工作进程，该进程的作用是处理一些耗时较长的任务，以达到释放Worker进程的目的。Worker进程可以通过`swoole_server`对象的task方法投递一个任务到Task Worker进程.

Worker进程通过Unix Sock管道将数据发送给Task Worker，这样Worker进程就可以继续处理新的逻辑，无需等待耗时任务的执行。需要注意的是，由于Task Worker是独立进程，因此无法直接在两个进程之间共享全局变量，需要使用Redis、MySQL或者swoole_table来实现进程间共享数据。

## 实例

要使用Task Worker，需要进行一些必要的操作。

首先，需要设置swoole_server的配置参数：

```php
$serv->set(array(
    'task_worker_num' => 2, // 设置启动2个task进程
));
```

接着，绑定必要的回调函数：

```php
$serv->on('Task', 'onTask');
$serv->on('Finish','onFinish');
```
其中两个回调函数的原型如下所示：

```php
/**
 * @param $serv swoole_server swoole_server对象
 * @param $task_id int 任务id
 * @param $from_id int 投递任务的worker_id
 * @param $data string 投递的数据
 */
function onTask(swoole_server $serv, $task_id, $from_id, $data);

/**
 * @param $serv swoole_server swoole_server对象
 * @param $task_id int 任务id
 * @param $data string 任务返回的数据
 */
function onFinish(swoole_server $serv, $task_id, $data)；
```

在实际逻辑中，当需要发起一个任务请求时，可以使用如下方法调用：

```php
$data = "task data";
$serv->task($data , -1 ); // -1代表不指定task进程

// 在1.8.6+的版本中，可以动态指定onFinish函数
$serv->task($data, -1, function (swoole_server $serv, $task_id, $data) {
    echo "Task Finish Callback\n";
});
```

####**1.Task简介**
Swoole的业务逻辑部分是同步阻塞运行的，如果遇到一些耗时较大的操作，例如访问数据库、广播消息等，就会影响服务器的响应速度。因此Swoole提供了Task功能，将这些耗时操作放到另外的进程去处理，当前进程继续执行后面的逻辑。

####**2.开启Task功能**
开启Task功能只需要在swoole_server的配置项中添加[task_worker_num](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/01.%E9%85%8D%E7%BD%AE%E9%80%89%E9%A1%B9.md#6task_worker_num)一项即可，如下：
```php
$serv->set(array(
    'task_worker_num' => 8
));
```
即可开启task功能。此外，必须给swoole_server绑定两个回调函数：[onTask](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/02.%E4%BA%8B%E4%BB%B6%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0.md#6ontask)和[onFinish](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/02.%E4%BA%8B%E4%BB%B6%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0.md#7onfinish)。这两个回调函数分别用于执行Task任务和处理Task任务的返回结果。

####**3.使用Task**
首先是发起一个Task，代码如下：
```php
public function onReceive( swoole_server $serv, $fd, $from_id, $data ) {
    echo "Get Message From Client {$fd}:{$data}\n";
    // send a task to task worker.
    $param = array(
        'fd' => $fd
    );
    // start a task
    $serv->task( json_encode( $param ) );
    
    echo "Continue Handle Worker\n";
}
```
可以看到，发起一个任务时，只需通过swoole_server对象调用task函数即可发起一个任务。swoole内部会将这个请求投递给task_worker，而当前Worker进程会继续执行。

当一个任务发起后，task_worker进程会响应[onTask](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/02.%E4%BA%8B%E4%BB%B6%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0.md#6ontask)回调函数，如下：
```php
public function onTask($serv,$task_id,$from_id, $data) {
    echo "This Task {$task_id} from Worker {$from_id}\n";
    echo "Data: {$data}\n";
    for($i = 0 ; $i < 10 ; $i ++ ) {
        sleep(1);
        echo "Task {$task_id} Handle {$i} times...\n";
    }
    $fd = json_decode( $data , true )['fd'];
    $serv->send( $fd , "Data in Task {$task_id}");
    return "Task {$task_id}'s result";
}
```
这里我用sleep函数和循环模拟了一个长耗时任务。在onTask回调中，我们通过task_id和from_id(也就是worker_id)来区分不同进程投递的不同task。当一个task执行结束后，通过return一个字符串将执行结果返回给Worker进程。Worker进程将通过[onFinish](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/02.%E4%BA%8B%E4%BB%B6%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0.md#7onfinish)回调函数接收这个处理结果。

下面来看onFinish回调：
```php
public function onFinish($serv,$task_id, $data) {
    echo "Task {$task_id} finish\n";
    echo "Result: {$data}\n";
}
```
在[onFinish](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/02.%E4%BA%8B%E4%BB%B6%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0.md#7onfinish)回调中，会接收到Task任务的处理结果$data。在这里处理这个返回结果即可。
（**Tip:** 可以通过在传递的data中存放fd、buff等数据，来延续投递Task之前的工作）

[点此查看完整示例](https://github.com/LinkedDestiny/swoole-doc/blob/master/src/02/swoole_task_server.php)

