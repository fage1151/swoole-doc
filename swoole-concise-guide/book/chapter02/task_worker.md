# Task Worker

---
[TOC=2,4]
<!-- toc -->

## 简介

Task Worker是Swoole中一种特殊的工作进程，该进程的作用是处理一些耗时较长的任务，以达到释放Worker进程的目的。Worker进程可以通过`swoole_server`对象的task方法投递一个任务到Task Worker进程.

Worker进程通过Unix Sock管道将数据发送给Task Worker，这样Worker进程就可以继续处理新的逻辑，无需等待耗时任务的执行。

需要注意的是，由于Task Worker是独立进程，因此无法直接在两个进程之间共享全局变量，需要使用Redis、MySQL或者swoole_table来实现进程间共享数据。

## Task/Finish特性的用途 

task模块用来做一些异步的慢速任务，比如webim中发广播，发送邮件等。

* task进程必须是同步阻塞的
* task进程支持定时器

node.js 假如有10万个连接，要发广播时，那会循环10万次，这时候程序不能做任何事情，不能接受新的连接，也不能收包发包。

而swoole不同，丢给task进程之后，worker进程可以继续处理新的数据请求。任务完成后会异步地通知worker进程告诉它此任务已经完成。

当然task模块的作用还不仅如此，实现PHP的数据库连接池，异步队列等，还需要进一步挖掘。


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

### 1.Task简介
Swoole的业务逻辑部分是同步阻塞运行的，如果遇到一些耗时较大的操作，例如访问数据库、广播消息等，就会影响服务器的响应速度。因此Swoole提供了Task功能，将这些耗时操作放到另外的进程去处理，当前进程继续执行后面的逻辑。

### 2.开启Task功能
开启Task功能只需要在swoole_server的配置项中添加[task_worker_num](server/set.md)一项即可，如下：
```php
$serv->set(array(
    'task_worker_num' => 8
));
```
即可开启task功能。此外，必须给swoole_server绑定两个回调函数：[onTask](server/02.swoole_server事件回调函数.md)和[onFinish](server/02.swoole_server事件回调函数.md)。这两个回调函数分别用于执行Task任务和处理Task任务的返回结果。

### 3.使用Task
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

当一个任务发起后，task_worker进程会响应[onTask](server/02.swoole_server事件回调函数.md)回调函数，如下：
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
这里我用sleep函数和循环模拟了一个长耗时任务。在onTask回调中，我们通过task_id和from_id(也就是worker_id)来区分不同进程投递的不同task。当一个task执行结束后，通过return一个字符串将执行结果返回给Worker进程。Worker进程将通过[onFinish](server/02.swoole_server事件回调函数.md)回调函数接收这个处理结果。

下面来看onFinish回调：
```php
public function onFinish($serv,$task_id, $data) {
    echo "Task {$task_id} finish\n";
    echo "Result: {$data}\n";
}
```
在[onFinish](server/02.swoole_server事件回调函数.md)回调中，会接收到Task任务的处理结果$data。在这里处理这个返回结果即可。
（**Tip:** 可以通过在传递的data中存放fd、buff等数据，来延续投递Task之前的工作）

[点此查看完整示例](https://github.com/LinkedDestiny/swoole-doc/blob/master/src/02/swoole_task_server.php)

## 3.Task进阶：MySQL连接池
上一章中我简单讲解了如何开启和使用Task功能。这一节，我将提供一个Task的高级用法。<br>

在PHP中，访问MySQL数据库往往是性能提升的瓶颈。而MySQL连接池我想大家都不陌生，这是一个很好的提升数据库访问性能的方式。传统的MySQL连接池，是预先申请一定数量的连接，每一个新的请求都会占用其中一个连接，请求结束后再将连接放回池中，如果所有连接都被占用，新来的连接则会进入等待状态。<br>
知道了MySQL连接池的实现原理，那我们来看如何使用Swoole实现一个连接池。<br>
首先，Swoole允许开启一定量的Task Worker进程，我们可以让每个进程都拥有一个MySQL连接，并保持这个连接，这样，我们就创建了一个连接池。<br>
其次，设置swoole的[dispatch_mode](server/set.md)为抢占模式(主进程会根据Worker的忙闲状态选择投递，只会投递给处于闲置状态的Worker)。这样，每个task都会被投递给闲置的Task Worker。这样，我们保证了每个新的task都会被闲置的Task Worker处理，如果全部Task Worker都被占用，则会进入等待队列。<br>

下面直接上关键代码：<br>
```php
public function onWorkerStart( $serv , $worker_id) {
    echo "onWorkerStart\n";
    // 判定是否为Task Worker进程
    if( $worker_id >= $serv->setting['worker_num'] ) {
    	$this->pdo = new PDO(
    		"mysql:host=localhost;port=3306;dbname=Test", 
    		"root", 
    		"123456", 
    		array(
                PDO::MYSQL_ATTR_INIT_COMMAND => "SET NAMES 'UTF8';",
                PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                PDO::ATTR_PERSISTENT => true
        	)
        );
    }
}
```
首先，在每个Task Worker进程中，创建一个MySQL连接。这里我选用了PDO扩展。<br>

```php
public function onReceive( swoole_server $serv, $fd, $from_id, $data ) {
    $sql = array(
    	'sql'=>'select * from Test where pid > ?',
    	'param' => array(
    		0
    	),
    	'fd' => $fd
    );
    $serv->task( json_encode($sql) );
}
```
其次，在需要的时候，通过[task]()函数投递一个任务（也就是发起一次SQL请求）<br>
```php
public function onTask($serv,$task_id,$from_id, $data) {
   	$sql = json_decode( $data , true );
	
	$statement = $this->pdo->prepare($sql['sql']);
    $statement->execute($sql['param']);    	

    $result = $statement->fetchAll(PDO::FETCH_ASSOC);
    $serv->send( $sql['fd'],json_encode($result));
	return true;
}
```
最后，在onTask回调中，根据请求过来的SQL语句以及相应的参数，发起一次MySQL请求，并将获取到的结果通过send发送给客户端（或者通过return返回给Worker进程）。而且，这样的一次MySQL请求还不会阻塞Worker进程，Worker进程可以继续处理其他的逻辑。<br>

可以看到，简单十几行代码，就实现了一个高效的异步MySQL连接池。<br>
通过测试，单个客户端一共发起1W次select请求，共耗时9s;<br> 1W次insert请求，共耗时21s。<br>
(客户端会在每次收到前一个请求的结果后才会发起下一次请求，而不是并发)。

[点此查看完整服务端代码](https://github.com/LinkedDestiny/swoole-doc/blob/master/src/03/swoole_mysql_pool_server.php)<br>
[点此查看完整客户端代码](https://github.com/LinkedDestiny/swoole-doc/blob/master/src/03/swoole_mysql_pool_client.php)<br>

>可参考[swoole_framework中的代码](http://git.oschina.net/swoole/swoole_framework/blob/master/libs/Swoole/Async/MySQL.php)
>redis连接池可参考[swoole_framework中的代码](http://git.oschina.net/swoole/swoole_framework/blob/master/libs/Swoole/Async/Redis.php)

## 4.Task实战：yii中应用task
在YII框架中结合了swoole 的task 做了异步处理。
本例中 主要用到
1、protected/commands/ServerCommand.php 用来做server。
2、protected/event/下的文件 这里是在异步中的具体实现。

客户端调用参照 TestController
```php
<?php
class TestController extends Controller{
    public function actionTT(){
        $message['uid'] = 2;
        $message['email'] = '83212019@qq.com';
        $message['title'] = '接口报警邮件';
        $message['contents'] = "'EmailEvent'接口请求过程出错！ 错误信息如下：err_no:'00000' err_msg:'测试队列' 请求参数为:'[]'";
        $message['type'] = 2;

        $data['param'] = $message;
        $data['class'] = 'Email';
        $client = new EventClient();
        $data = $client->send($data);
    }
}
```

有个task表是用来记录异步任务的。如果失败重试3次。sql在protected/data/sql.sql里。  
[点此查看完整客户端代码](https://github.com/LinkedDestiny/swoole-doc/blob/master/src/03/swoole_mysql_pool_client.php)