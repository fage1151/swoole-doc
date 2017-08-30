# Reactor、Worker、Task的关系
三种角色分别的职责是：

**Reactor线程**
* 负责维护客户端机器的TCP连接、处理网络IO、收发数据
* 完全是异步非阻塞的模式
* 全部为C代码，除Start/Shudown事件回调外，不执行任何PHP代码
* 将TCP客户端发来的数据缓冲、拼接、拆分成完整的一个请求数据包
* Reactor以多线程的方式运行

**Worker进程**
* 接受由Reactor线程投递的请求数据包，并执行PHP回调函数处理数据
* 生成响应数据并发给Reactor线程，由Reactor线程发送给TCP客户端
* 可以是异步非阻塞模式，也可以是同步阻塞模式
* Worker以多进程的方式运行

**Task进程**
* 接受由Worker进程通过swoole_server->task/taskwait方法投递的任务
* 处理任务，并将结果数据返回给Worker进程
* 完全是同步阻塞模式
* Task以多进程的方式运行

**关系**
可以理解为reactor就是nginx，worker就是php-fpm。reactor线程异步并行地处理网络请求，然后再转发给worker进程中去处理。reactor和worker间通过IPC方式通信。

swoole的reactor，worker，task_worker之间可以紧密的结合起来，提供更高级的使用方式。

一个更通俗的比喻，假设Server就是一个工厂，那reactor就是销售，帮你接项目订单。而worker就是工人，当销售接到订单后，worker去工作生产出客户要的东西。而task_worker可以理解为行政人员，可以帮助worker干些杂事，让worker专心工作。

> 底层会为Worker进程、Task进程分配一个唯一的ID
> 不同的task/worker进程之间可以通过sendMessage接口进行通信