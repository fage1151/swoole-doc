# mysql client
Swoole在1.8.6版本提供了全新的异步MySQL客户端，底层自行实现了MySQL的通信协议，无需依赖其他第三方库，如libmysqlclient、mysqlnd、mysqli等。

## **swoole_mysql->construct**
创建异步mysql客户端。
## **swoole_mysql->on**
设置事件回调函数。目前仅支持onClose事件回调。

~~~
function swoole_mysql->on($event_name, callable $callback);

~~~
**onClose事件**
当连接关闭时回调此函数。

~~~
$db->on('Close', function($db){
    echo "MySQL connection is closed.\n";
});
~~~
## **swoole_mysql->connect**
异步连接到MySQL服务器。

~~~
function swoole_mysql->connect(array $serverConfig, callable $callback);

~~~
$serverConfig为MySQL服务器的配置，必须为关联索引数组
$callback连接完成后回调此函数
服务器配置
~~~
$server = array(
    'host' => '192.168.56.102',
    'user' => 'test',
    'password' => 'test',
    'database' => 'test',
    'charset' => 'utf8',
);
~~~
* host MySQL服务器的主机地址，支持IPv6（::1）和UnixSocket（unix:/tmp/mysql.sock）
* port MySQL服务器监听的端口，选填，默认为3306
* user 用户名，必填
* password 密码，必填
* database 连接的数据库，必填
* charset 设置客户端字符集，选填，默认使用Server返回的字符集。如果字符串不存在，底层会抛出Swoole\MySQL\Exception异常

**回调函数**
~~~
function onConnect(swoole_mysql $db, bool $result);

~~~
* $db 为swoole_mysql对象
* $result 连接是否成功，只有为true时才可以执行query查询
* $result 为false，可以通过connect_errno和connect_error得到失败的错误码和错误信息

## **swoole_mysql->escape**
转义SQL语句中的特殊字符，避免SQL注入攻击。底层基于mysqlnd提供的函数实现，`需要依赖PHP的mysqlnd扩展`。

* 编译时需要增加--enable-mysqlnd来启用，如果你的PHP中没有mysqlnd将会出现编译错误
* 必须在connect完成后才能使用
* 客户端未设置字符集时默认使用Server返回的字符集设置，可在connect方法中加入charset修改连接字符集
>此方法在1.9.6或更高版本可用

~~~
function swoole_mysql->escape(string $str) : string

~~~
**使用实例**
~~~
$db = new swoole_mysql;
$server = array(
    'host' => '127.0.0.1',
    'user' => 'root',
    'password' => 'root',
    'database' => 'test',
);
$db->connect($server, function ($db, $result) {
    $data = $db->escape("abc'efg\r\n");
});
~~~

## **swoole_mysql->query**
执行SQL查询。

~~~


function swoole_mysql->query($sql, callable $callback);
~~~

* $sql为要执行的SQL语句
* $callback执行成功后会回调此函数
* 每个MySQLi连接只能同时执行一条SQL，必须等待返回结果后才能执行下一条SQL

**回调函数**

~~~
function onSQLReady(swoole_mysql $link, mixed $result);

~~~
* 执行失败，$result为false，读取$link对象的error属性获得错误信息，errno属性获得错误码
* 执行成功，SQL为非查询语句，$result为true，读取$link对象的affected_rows属性获得影响的行数，insert_id属性获得Insert操作的自增ID
* 执行成功，SQL为查询语句，$result为结果数组

**事务处理**
在Swoole\MySQL中执行下列SQL语句可以实现事务处理。

* 启动事务：START TRANSACTION
* 提交事务：COMMIT
* 回滚事务：ROLLBACK

## **swoole_mysql->begin**
启动事务。函数原型：

~~~
function swoole_mysql->begin(callable $callback);

~~~
启动一个MySQL事务，事务启动成功会回调指定的函数
与commit和rollback结合实现MySQL事务处理
* 同一个MySQL连接对象，同一时间只能启动一个事务
* 必须等到上一个事务commit或rollback才能继续启动新事务
* 否则底层会抛出Swoole\MySQL\Exception异常，异常code为21

>事务处理在1.9.15或更高版本可用

**使用实例**
~~~
$db->begin(function( $db, $result) {
    $db->query("update userinfo set level = 22 where id = 1", function($db, $result) {
        $db->rollback(function($db, $result) {
            echo "commit ok\n";
        });
    });
});
~~~
## **swoole_mysql->commit**
提交事务。

~~~
function swoole_mysql->commit(callable $callback);

~~~
* 提交事务，当服务器返回响应时回调此函数
* 必须先调用begin启动事务才能调用commit否则底层会抛出Swoole\MySQL\Exception异常
* 异常code为22

>在1.9.15或更高版本可用

**使用实例**
```
$db->begin(function( $db, $result) {
    $db->query("update userinfo set level = 22 where id = 1", function($db, $result) {
        $db->commit(function($db, $result){
            echo "commit ok\n";
        });
    });
});
```
**异步mysql客户端**
```php
global $mysql;
$mysql = new Mysql;
$server = array(
    'host' => '192.168.56.102',
    'port' => 3306,
    'user' => 'test',
    'password' => 'test',
    'database' => 'test',
    'charset' => 'utf8', //指定字符集
    'timeout' => 2,  // 可选：连接超时时间（非查询超时时间），默认为SW_MYSQL_CONNECT_TIMEOUT（1.0）
);

$mysql->connect($server, function (Mysql $db, $r) {
    if ($r === false) {
        var_dump($db->connect_errno, $db->connect_error);
        die;
    }
});
$sql = 'show tables';
$mysql->query($sql, function (Mysql $db, $r) {
    if ($r === false) {
        var_dump($db->error, $db->errno);
    } elseif ($r === true) {
        var_dump($db->affected_rows, $db->insert_id);
    }
    var_dump($r);
});