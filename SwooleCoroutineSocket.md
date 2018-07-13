# Coroutine\Socket
Swoole-2.2版本增加了更底层的Coroutine\Socket模块，相比Server和Client相关模块Socket可以实现更细粒度的一些IO操作。

可使用Co\Socket短命名简化类名。

>需要2.2.0或更高版本

## 协程调度
Coroutine\Socket模块提供的IO操作接口均为同步编程风格，底层自动使用协程调度器实现异步非阻塞IO。

## 错误码
在执行socket相关系统调用时，可能返回-1错误，底层会设置Coroutine\Socket->$errCode属性为系统错误编号errno，请参考响应的man文档。如$socket->accept()返回错误时，errCode含义可以参考man accept中列出的错误码文档。