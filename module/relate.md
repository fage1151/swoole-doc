# swoole相关开源项目
## 框架
* [TSF](https://github.com/tencent-php/tsf) Tencent-TSF 腾讯公司推出的PHP协程框架，基于Swoole+PHP Generator实现Coroutine，可以像Golang一样用协程实现高并发服务器。  
* [swoole_framework](https://github.com/matyhtf/framework) SwooleFramework是一个全功能的后端服务器框架   
* [zphp](https://github.com/shenzhe/zphp) 一个极轻的的，专用于游戏(社交，网页，移动)的服务器端开发框架.提供高性能实时通信方案。zphp使用swoole作为底层网络通信的框架。  
* [zapi](https://github.com/keaixiaou/zapi) 基于swoole+generator的http api异步非阻塞轻量级框架,内置mysql、redis、memcached、mongodb全套异步客户端的连接池，内置http异步客户端，近乎同步的写法，却是异步的调用，性能强悍  
* [zhttp](https://github.com/keaixiaou/zhttp) 基于swoole+generator的异步非阻塞轻量级web框架,内置mysql、redis、memcached、mongodb全套异步客户端的连接池，内置http异步客户端，近乎同步的写法，却是异步的调用，性能强悍  
* [swoole-yaf](https://github.com/LinkedDestiny/swoole-yaf) 结合PHP的Yaf框架和Swoole扩展的高性能PHP Web框架  
* [Swoole-Yaf](https://github.com/wenjun1055/swoole-yaf) 将Yaf框架和Swoole扩展提供的HttpServer结合在一起，server和框架高度结合形成超高性能的组合  
* [ciswoole](https://github.com/smalleyes/ciswoole) ci2.2结合Swoole_Http_Server  
* [Blink](https://github.com/bixuehujin/blink) 是一个为构建 “long running” 服务而生的 Web 微型高性能框架，它为构建 Web 应用程序提供简洁优雅的API，尽量的减轻我们的常规开发工作  
* [Group-co](https://github.com/fucongcong/Group-Co) 优雅的异步协程框架,支持服务化搭建高并发httpserver，支持分布式使用，详情请戳链接。  
* [SwooleMan](https://github.com/osgochina/SwooleMan) SwooleMan 是基于Swoole开发的兼容Workerman API 的一个服务框架. 致力于帮助现有的workerman项目,方便的运行在Swoole服务上,享受swoole带来的高性能.  
* [swoole-worker](https://github.com/osgochina/swoole-worker) 基于swoole_process的workerman,移除了对pcntl,libevent,event扩展的依赖,转而使用swoole提供的swoole_process和swoole_event，定时器采用swoole的swoole_timer
* [FastD](https://github.com/JanHuang/fastD) FastD 适用于对性能有要求的 API 场景，并且灵活的扩展性可以让开发者们更容易地建造自己的服务。支持HTTP、TCP、UDP、WebSocket，简单，易用。
* [easySwoole](https://github.com/kiss291323003/easyswoole) easySwoole 专为API而生，支持多层级(组模式)控制器访问与多种事件回调,高度封装了Swoole Server 而依旧维持Swoole Server原有特性，支持在 Server 中监听自定义的TCP、UDP协议。
* [swPromise](https://github.com/coooold/swPromise) swPromise 基于swoole的PHP promise框架
* [zys](https://github.com/qieangel2013/zys)基于Yaf和Swoole的i高性能Service框架
* [Laravoole](https://github.com/garveen/laravoole) Laravel和Swoole结合的超高性能框架
* [LaravelFly](https://github.com/scil/LaravelFly)  Laravel结合swoole
* [stone](https://github.com/StoneGroup/stone) 基于swoole,能大幅提升基于Laravel的程序性能
* [MyQEE-Server](https://github.com/myqee/server) MyQEE-Server 将swoole服务和功能对象抽象化，为每个 Worker、Task、多端口分配一个对象，带来全新的编程体验让代码清晰有条理，适合多端口以及Http、WebSocket、Tcp混合的应用服务器开发，支持创建大文件、断点、分片上传的Http服务器
* [PtWebserver](https://git.oschina.net/pantian/PtWebserver) PtWebserver 基于php swoole 扩展的高性能web 服务器。应用对象常驻内存，不用重复创建对象，提高响应时间与性能
* [EPServer](https://github.com/ewenlaz/epserver) EPServer 高性能TCP服务器框架，底层基于swoole扩展
* [php-webserver](https://github.com/matyhtf/php-webserver) 基于swoole+http_parser2个扩展开发的高性能PHP web服务器。压测性能超过php-fpm的2倍
* [think-swoole](https://github.com/top-think/think-swoole)ThinkPHP 5.0 Swoole 扩展
                  
## 工具
* [swoole-crontab](https://github.com/osgochina/swoole-crontab) 基于swoole的定时器程序，支持秒级处理. 异步多进程处理。完全兼容crontab语法，且支持秒的配置  
* [redis-async](https://github.com/swoole/redis-async) 基于swoole开发的异步Redis+连接池，性能非常强劲。使用redis-async开发的Web应用，QPS可以高达3.5万QPS，超过php-fpm+php-redis扩展性能的10倍。  
* [mysql-async](https://github.com/swoole/mysql-async) 基于swoole扩展开发的异步MySQL类库，内置连接池和SQL任务排队机制
* [multiprocess](https://github.com/kcloze/multiprocess) multiprocess 基于swoole的进程管理组件，可轻松让普通PHP脚本变守护进程和多进程执行
* [DHT](https://github.com/ylqjgm/DHT) DHT 使用swoole编写的DHT爬虫程序，可正常获取infohash
* [swoole-vmstat](https://github.com/smalleyes/swoole-vmstat) swoole-vmstat 运用swoole在浏览器更友好的实现vmstat  
* [swoole-server-manager](https://github.com/df007df/swoole-server-manager) swoole-server-manager 常驻服务管理框架  
* [statistics](https://github.com/smalleyes/statistics) 一个运用php与swoole实现的统计监控系统
* [Sworm](https://github.com/heikezy/Sworm) 基于Swoole的异步MySQL数据库ORM框架
       
## 分布式
* [SwooleDistributed](https://github.com/tmtbe/SwooleDistributed)  swoole 分布式全栈框架框架  
* [swoole-task](https://github.com/luxixing/swoole-task) 是基于PHP swoole扩展开发的一个异步多进程任务处理框架，服务端和客户端通过http协议进行交互。它适用于任务需要花费较长时间处理，而客户端不必关注任务执行结果的场景.比如数据清洗统计类的工作，报表生成类任务  
* [swoole-jobs](https://github.com/kcloze/swoole-jobs) 基于swoole的job调度组件
* [DFS](https://github.com/qieangel2013/dfs) DFS 分布式文件服务器
* [DoraRPC](https://github.com/xcl3721/Dora-RPC) Dora RPC 是一款基础于Swoole定长包头通讯协议的最精简的RPC, 用于复杂项目前后端分离，分离后项目都通过API工作可更好的跟踪、升级、维护及管理。
* [swoole-JsonRPC](https://github.com/smalleyes/swoole-JsonRPC) swoole简单实现jsonRPC


