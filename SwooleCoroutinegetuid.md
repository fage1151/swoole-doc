# Swoole\Coroutine::getuid
获取当前协程的ID。函数原型

~~~
int function Swoole\Coroutine::getuid();
~~~
协程task的结构是：

~~~c
struct _coro_task
{
    int cid; //协程ID
    /**
     * user coroutine
     */
    zval *function;
    time_t start_time;
    void (*post_callback)(void *param);
    void *post_callback_params;
};
~~~
* 返回当前协程ID，是一个由0增长的长整型。