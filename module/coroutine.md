Swoole在2.0开始内置协程(Coroutine)的能力，提供了具备协程能力IO接口（统一在命名空间Swoole\Coroutine\*）

>2.0.2或更高版本已支持PHP7

协程可以理解为纯用户态的线程，其通过协作而不是抢占来进行切换。相对于进程或者线程，协程所有的操作都可以在用户态完成，创建和切换的消耗更低。Swoole可以为每一个请求创建对应的协程，根据IO的状态来合理的调度协程，这会带来了以下优势：

* 开发者可以无感知的用同步的代码编写方式达到异步IO的效果和性能，避免了传统异步回调所带来的离散的代码逻辑和陷入多层回调中导致代码无法维护。

* 同时由于swoole是在底层封装了协程，所以对比传统的php层协程框架，开发者不需要使用yield关键词来标识一个协程IO操作，所以不再需要对yield的语义进行深入理解以及对每一级的调用都修改为yield，这极大的提高了开发效率。

启用协程

* PHP版本要求：>= 5.5，包括5.5、5.6、7.0、7.1
* 基于swoole_server或者swoole_http_server进行开发，目前只支持在onRequet, onReceive, onConnect事件回调函数中使用协程。

swoole提供了四种协程Client：

* TCP/UDP Client->\Swoole\Coroutine\Client  
* HTTP Client->\Swoole\Coroutine\HTTP\Client  
* Redis Client->\Swoole\Coroutine\Redis
* Mysql Client->\Swoole\Coroutine\MySQL

在协程Server中需要使用协程版Client，可以实现全异步server

同时swoole提供了协程工具集：\Swoole\Coroutine\Util，提供了获取当前协程id，反射调用等能力。
```php
<?php
$client = new Swoole\Coroutine\Client(SWOOLE_SOCK_TCP);
$client->connect("127.0.0.1", 8888, 0.5);
//调用connect将触发协程切换
$client->send("hello world from swoole");
//调用recv将触发协程切换
$ret = $client->recv();
$client->close();
echo $ret;
```
支持协程的回调方法列表 
目前Swoole2 仅有部分事件回调函数底层自动创建了协程，可以调用协程客户端。本节列出了支持协程客户端的回调列表以及实现的版本号。

v2.0.5  
onConnect  
onReceive  
onPacket  
onRequest  
onHandShake  
v2.0.6  
onMessage  
v2.0.7  
onOpen  
Redis\Server->handler
**新增配置**
在Swoole\Server的set方法中增加了一个配置参数max_coro_num，用于配置一个worker进程最多同时处理的协程数目。因为随着worker进程处理的协程数目的增加，其占用的内存也会增加，为了避免超出php的memory_limit限制，请根据实际业务的压测结果设置该值，默认为3000。



**注意事项**

* 全局变量：协程使得原有的异步逻辑同步化，但是在协程的切换是隐式发生的，所以在协程切换的前后不能保证全局变量以及static变量的一致性。
* 请勿在以下场景中触发协程切换：
    * 析构函数
    * 魔术方法__call()
* gcc 4.4下如果在编译swoole的时候（即make阶段），出现gcc warning： dereferencing pointer ‘v.327’ does break strict-aliasing rules、 dereferencing type-punned pointer will break strict-aliasing rules 请手动编辑Makefile，将CFLAGS = -Wall -pthread -g -O2替换为CFLAGS = -Wall -pthread -g -O2 -fno-strict-aliasing，然后重新编译make clean;make;make install
* 与xdebug、xhprof等zend扩展不兼容，例如不能使用xhprof对协程server进行性能分析采样。
* 原生的call_user_func和call_user_func_array中无法使用协程client，请使用\Swoole\Coroutine::call_user_func和\Swoole\Coroutine::call_user_func_array代替,在PHP7中如果无法保证在编译时反射调用的类是编译器已知的，请统一使用协程版反射调用