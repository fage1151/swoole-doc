# 函数列表

swoole除了网络通信相关的函数外，还提供了一些获取系统信息的函数供PHP程序使用。
## swoole_set_process_name 
用于设置进程的名称。修改进程名称后，通过ps命令看到的将不再是php your_file.php。而是设定的字符串。 此函数接受一个字符串参数。 此函数与PHP5.5提供的cli_set_process_title功能是相同的。但swoole_set_process_name可用于PHP5.2之上的任意版本。swoole_set_process_name兼容性比cli_set_process_title要差，如果存在cli_set_process_title函数则优先使用cli_set_process_title。
```
void swoole_set_process_name(string $name);
```
示例代码：
```
swoole_set_process_name("swoole server");
var_dump($argv);
sleep(1000);
```
>swoole_set_process_name在1.6.3版本提供
>在onStart回调中执行此函数，将修改主进程的名称。在onWorkerStart中调用将修改worker子进程的名称。

如何为Swoole Server重命名各个进程名称

* 在swoole_server_create之前修改为manager进程名称
* onStart调用时修改为主进程名称
* onWorkerStart修改为worker进程名称

>1.6.12后增加了onManagerStart事件回调，可以在这里设置管理进程的名称
>低版本Linux内核和Mac OSX不支持进程重命名

## swoole_version
获取swoole扩展的版本号，如1.6.10
```
string swoole_version();
```
## swoole_strerror
将标准的Unix Errno错误码转换成错误信息。函数原型：
```
string swoole_strerror(int $errno);
```
## swoole_errno 
获取最近一次系统调用的错误码，等同于C/C++的errno变量。
```
int swoole_errno();
```

错误码的值与操作系统有关。可是使用swoole_strerror将错误转换为错误信息。
## swoole_get_local_ip 

此函数用于获取本机所有网络接口的IP地
```
function swoole_get_local_ip();
$result = array("eth0" => "192.168.1.100");
```

* 目前只返回IPv4地址，返回结果会过滤掉本地loop地址127.0.0.1。
* 结果数组是以interface名称为key的关联数组。比如 array("eth0" => "192.168.1.100")
* 此函数会实时调用ioctl系统调用获取接口信息，底层无缓存

## swoole_clear_dns_cache
清除swoole内置的DNS缓存，对swoole_client和swoole_async_dns_lookup 有效。
```
function swoole_clear_dns_cache();
```
## swoole_get_local_mac 
获取本机网卡Mac地址。
```
function swoole_get_local_mac() : array;
```
调用成功返回所有网卡的Mac地址
```
array(4) {
  ["lo"]=>
  string(17) "00:00:00:00:00:00"
  ["eno1"]=>
  string(17) "64:00:6A:65:51:32"
  ["docker0"]=>
  string(17) "02:42:21:9B:12:05"
  ["vboxnet0"]=>
  string(17) "0A:00:27:00:00:00"
}
```
>在1.9.18或更高版本可用