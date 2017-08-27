# 不同的Server程序实例间如何通信
有2种方法可以实现不同的Server程序实例间通信。

## **额外监听一个UDP端口**
* 额外监听一个UDP端口并设置onPacket回调，接收来自其他Server发来的消息
* UDP通信不是可靠的，消息可能会丢失。需要应用层发送消息接收回执
* UDP通信不存在阻塞IO
## **使用swoole_client作为客户端访问Server**
在Server程序中创建一个TCP客户端，连接到另外一个Server程序。实现Server与Server之间通信。

* TCP通信时可靠的可以保证安全性
* 创建TCP客户端连接是推荐的做法，异步Server中创建一个异步的TCP客户端可以保证整个Server程序是纯异步的