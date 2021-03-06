# 开发者必须知道的几个问题

**1、swoole不依赖apache或者nginx**

swoole本身已经是一个类似apache/nginx的容器，只要PHP环境OK swoole就可以运行。

**2、是命令行启动的**

启动方式类似apache使用命令启动。

**3、长链接必须加心跳**

长链接必须加心跳，长链接必须加心跳，长链接必须加心跳，重要的话说三遍。 
长链接长时间不通讯肯定会被防火墙干掉而断开。不加心跳的长链接应用就等着老板KO你吧。[心跳说明](heart_beat.md)

**4、客户端和服务端协议一定要对应才能通讯**

这个是开发者非常常见的问题。例如客户端是用websocket协议，服务端必须也是websocket协议(服务端```new swoole_websocket_server("0.0.0.0", 9501)```)才能连得上，才能通讯。 
不要尝试在浏览器地址栏访问websocket协议端口，不要尝试用webscoket协议访问裸tcp协议端口，协议一定要对应。

这里的原理类似如果你要和英国人交流，那么要使用英语。如果要和日本人交流，那么要使用日语。这里的语言就类似与通许协议，双方(客户端和服务端)必须使用相同的语言才能交流，否则无法通讯。

**5、链接失败可能的原因**

刚开始使用swoole时很常见的一个问题是客户端链接服务端失败。 原因一般如下： 
1、服务器防火墙(包括云服务器安全组)阻止了链接 （50%几率是这个）
2、客户端和服务端使用的协议不一致 （30%几率）
3、ip或者端口写错了 (15%的几率)
4、服务端没启动

**6、编码注意事项**

* 不要在代码中执行sleep以及其他睡眠函数，这样会导致整个进程阻塞
* exit/die是危险的，会导致worker进程退出，如果需要返回，可以调用return。
* 可通过register_shutdown_function来捕获致命错误，在进程异常退出时做一些请求工作
* PHP代码中如果有异常抛出，必须在回调函数中进行try/catch捕获异常，否则会导致工作进程退出
* swoole不支持set_exception_handler，必须使用try/catch方式处理异常
* Worker进程不得共用同一个Redis或MySQL等网络服务客户端，Redis/MySQL创建连接的相关代码可以放到onWorkerStart回调函数中

**7、改代码要重启**

swoole是常驻内存的扩展，加载类/函数定义的文件后不会释放，改代码要重启swoole server才能看到新代码的效果。

**8、支持更高并发**

如果业务并发连接数超过1000同时在线，请务必[优化linux内核](内核参数调优.md).

**9、注意避免类和常量的重复定义**
由于swoole是常驻内存的，加载类/函数定义的文件后不会释放，所以要避免多次require/include相同的类或者常量的定义文件。建议使用require_once/include_once加载文件，否则会发生```cannot redeclare function/class``` 的致命错误。

**10、进程隔离**
进程隔离也是很多新手经常遇到的问题。修改了全局变量的值，为什么不生效，原因就是全局变量在不同的进程，内存空间是隔离的，所以无效。所以使用swoole开发Server程序需要了解进程隔离问题。

* 不同的进程中PHP变量不是共享，即使是全局变量，在A进程内修改了它的值，在B进程内是无效的
* 如果需要在不同的Worker进程内共享数据，可以用Redis、MySQL、文件、Swoole\Table、APCu、shmget等工具实现
* 不同进程的文件句柄是隔离的，所以在A进程创建的Socket连接或打开的文件，在B进程内是无效，即使是将它的fd发送到B进程也是不可用的

**11、不支持的函数**
swoole运行在PHP CLI模式下，PHP CLI模式下无法使用HTTP相关的函数。
* header、setcookie、session_start等函数,可以使用swoole_http_response->header()，swoole_http_response->cookie()，也无法使用move_uploaded_file()，is_uploaded_file()这些函数。
* 无法使用php://input，请用swoole_http_request->rawContent()代替。
* 无法使用$_SERVER、$_GET、$_POST、$_FILES、$_COOKIE、$_SESSION、$_REQUEST，请使用swoole_http_request->$server、swoole_http_request->$get，swoole_http_request->$post，swoole_http_request->$files，swoole_http_request->$cookie分别替代




