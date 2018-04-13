# Coroutine\PostgreSQL



**启用协程Postgresql客户端**

* 需要在编译swoole时增加`./configure --enable-coroutine --enable-coroutine-postgresql` 来开启此功能
* 需要确保系统中已安装libpq库 

mac安装完postgresq自带libpq库，环境之间有差异，ubuntu可能需要apt-get install libpq-dev 也可以单独指定libpq库目录如：`./configure --enable-coroutine --enable-coroutine-postgresql --with-libpq-dir=/etc/postgresql`

**使用示例**

~~~
go(function () {
    $pg = new Swoole\Coroutine\PostgreSQL();
    $conn  = $pg -> connect ("host=127.0.0.1 port=5432 dbname=test user=root password=");
    $result = $pg -> query($conn, 'SELECT * FROM test;');
    $arr = $pg -> fetchAll($result);
    var_dump($arr);
});
~~~