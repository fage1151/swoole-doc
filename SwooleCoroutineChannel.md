# Coroutine\Channel
通道，类似于go语言的chan，支持多生产者协程和多消费者协程。底层自动实现了协程的切换和调度。

* Channel->push ：当队列中有其他协程正在等待pop数据时，自动按顺序唤醒一个消费者协程。当队列已满时自动yield让出控制器，等待其他协程消费数据
* Channel->pop：当队列为空时自动yield，等待其他协程生产数据。消费数据后，队列可写入新的数据，自动按顺序唤醒一个生产者协程。

>Coroutine\Channel使用本地内存，不同的进程之间内存是隔离的。只能在同一进程的不同协程内进行push和pop操作
> Coroutine\Channel在2.0.13或更高版本可用

## 属性
* $capacity 通道缓冲区容量
## 示例
```
use Swoole\Coroutine as co;
$chan = new co\Channel(1);
co::create(function () use ($chan) {
    for($i = 0; $i < 100000; $i++) {
        co::sleep(1.0);
        $chan->push(['rand' => rand(1000, 9999), 'index' => $i]);
        echo "$i\n";
    }
});
co::create(function () use ($chan) {
    while(1) {
        $data = $chan->pop();
        var_dump($data);
    }
});
swoole_event::wait();
```