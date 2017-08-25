# 固定包头+包体协议
固定包头的协议非常通用，在BAT的服务器程序中经常能看到。这种协议的特点是一个数据包总是由包头+包体2部分组成。包头由一个字段指定了包体或整个包的长度，长度一般是使用2字节/4字节整数来表示。服务器收到包头后，可以根据长度值来精确控制需要再接收多少数据就是完整的数据包。Swoole的配置可以很好的支持这种协议，可以灵活地设置4项参数应对所有情况。

Swoole的Server和异步Client都是在onReceive回调函数中处理数据包，当设置了协议处理后，只有收到一个完整数据包时才会触发onReceive事件。同步客户端在设置了协议处理后，调用 $client->recv() 不再需要传入长度，recv函数在收到完整数据包或发生错误后返回。

$server->set(array(
    'open_length_check' => true,
    'package_max_length' => 81920,
    'package_length_type' => 'n', //see php pack()
    'package_length_offset' => 0,
    'package_body_offset' => 2,
));