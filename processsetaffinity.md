## swoole_process::setaffinity 
设置CPU亲和性，可以将进程绑定到特定的CPU核上。

~~~
function swoole_process::setaffinity(array $cpu_set);
~~~

* 接受一个数组参数表示绑定哪些CPU核，如array(0,2,3)表示绑定CPU0/CPU2/CPU3
* 成功返回true，失败返回false
* $cpu_set内的元素不能超过CPU核数
* CPU-ID不得超过（CPU核数 - 1）

使用SWOOLE_CPU_NUM常量可以得到当前服务器的CPU核数

**setaffinity函数在1.7.18以上版本可用**

此函数的作用是让进程只在某几个CPU核上运行，让出某些CPU资源执行更重要的程序。

