# open_mqtt_protocol
启用mqtt协议处理，启用后会解析mqtt包头，worker进程onReceive每次会返回一个完整的mqtt数据包。

~~~
$serv->set(array('open_mqtt_protocol' => true));
~~~