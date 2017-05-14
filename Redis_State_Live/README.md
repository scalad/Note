### Redis监控工具-Redis-Stat、RedisLive ###

Redis-stat（Ruby）和Redis Live（python）是两款Redis监控工具，下面将介绍如何安装部署这两个工具，监控Redis运行情况

#### 测试环境： ####

	Ubuntu 14.04 LTS x64
	Redis redis-3.0.7.tar.gz
	Ruby ruby 1.9.3
	Python 2.7.6

#### Redis 安装 ####
Redis安装请参照：[Redis安装与配置](https://github.com/scalad/Note/tree/master/Redis_Config)

#### Redis-stat 安装部署 ####
redis-stat is a simple Redis monitoring tool written in Ruby.

It is based on INFO command of Redis, and thus generally won’t affect the performance of the Redis instance unlike the other monitoring tools based on MONITOR command.

redis-stat allows you to monitor Redis instances

* either with vmstat-like output from the terminal
* or with the dashboard page served by its embedded web server.
* 
通常来说，不会像基于MONITOR命令的监控工具一样，对Redis本身有性能上的影响

[Github地址](https://github.com/junegunn/redis-stat)

#### 卸载原有Ruby ####
	sudo apt-get autoremove --purge ruby*

#### 安装Ruby ####
	sudo apt-get install ruby-full

#### 安装Redis-stat ####
	gem install redis-stat

#### 基本使用 ####
1. redis-stat命令参数
	
	usage: redis-stat [HOST[:PORT] ...] [INTERVAL [COUNT]]
	
	    -a, --auth=PASSWORD             设置密码
	    -v, --verbose                   显示更多信息
	        --style=STYLE               输出编码类型: unicode|ascii
	        --no-color                  取消ANSI颜色编码
	        --csv=OUTPUT_CSV_FILE_PATH  以CSV格式存储结果
	        --es=ELASTICSEARCH_URL      把结果发送到 ElasticSearch: [http://]HOST[:PORT][/INDEX]
	
	        --server[=PORT]             运行redis-stat的web server (默认端口号: 63790)
	        --daemon                    使得redis-stat成为进程。必须使用 --server 选项
	        
	        --version                   显示版本号
	        --help                      显示帮助信息

2. redis-stat运行命令行监控
	redis-stat
	redis-stat 1
	redis-stat 1 10
	redis-stat --verbose
	redis-stat localhost:6380 1 10
	redis-stat localhost localhost:6380 localhost:6381 5
	redis-stat localhost localhost:6380 1 10 --csv=/tmp/outpu.csv --verbose

3. Server端运行界面

![](https://github.com/scalad/Note/blob/master/Redis_State_Live/image/redis-stat-0.3.0.png)

4. Web界面中的redis-stat

当设置–server选项之后，redis-stat会在后台启动一个嵌入式的web server(默认端口号：63790)，可以让你在浏览器中监控Redis

	redis-stat --server
	redis-stat --verbose --server=8080 5
	
	# redis-stat server can be daemonized
	redis-stat --server --daemon
	
	# Kill the daemon
	killall -9 redis-stat-daemon

5. Web端运行界面

然后在你的浏览器中输入：

	http://你的Redis IP:63790

![](https://github.com/scalad/Note/blob/master/Redis_State_Live/image/redis-stat-web.png)

### RedisLive 安装部署 ###

Redis Live is a dashboard application with a number of useful widgets. At it’s heart is a monitoring script that periodically issues INFO and MONITOR command to the redis instances and stores the data for analytics.

长时间运行对Redis性能有所影响

[Github地址](https://github.com/nkrode/RedisLive)

[Real time dashboard for redis](http://www.nkrode.com/article/real-time-dashboard-for-redis)

#### 安装运行依赖 ####
1. tornado
	pip install tornado

2. redis.py
	pip install redis

3. python-dateutil
	pip install python-dateutil

#### 下载RedisLive ####
	git clone https://github.com/kumarnitin/RedisLive.git

#### conf配置 ####
进入src目录

	cp redis-live.conf.example ./redis-live.conf

	vim redis-live.conf

	{  
        "RedisServers":  
        [   
                {  
                  "server" : "你的Redis IP地址",  
                  "port"  : 6379  
                }
                ........
                可以多个
        ],  
          
 
        "DataStoreType" : "redis",  
 
        "RedisStatsServer":  
        {  
                "server" : "你的Redis 监控IP地址",  
                "port" : 6379  
        },
        
        "SqliteStatsStore" :
        {
                "path":  "to your sql lite file"
        }
	}
	
其中RedisServers为你要监控的redis实例，可以添加多个，RedisStatsServer是存储RedisLive监控数据的实例，如果redis有密码，可以在实例配置中加入password选项；如果没有存储RedisLive数据的实例，需要将DataStoreType改成”DataStoreType” : “sqlite”这种设置

#### 启动RedisLive ####
1. 启动监控脚本，监控120秒，duration参数是以秒为单位

	sudo ./redis-monitor.py --duration=120

2. 启动webserver。

RedisLive使用tornado作为web服务器，所以不需要单独安装服务器

Tornado web server 是使用Python编写出來的一个极轻量级、高可伸缩性和非阻塞IO的Web服务器软件

	sudo ./redis-live.py

#### Web运行界面 ####
然后在你的浏览器中输入：

	http://你的Redis IP:8888/index.html

![](https://github.com/scalad/Note/blob/master/Redis_State_Live/image/redis-live.png)

本文来自[http://wxmimperio.tk/2016/02/25/Redis-Monitor-Tools/](http://wxmimperio.tk/2016/02/25/Redis-Monitor-Tools/)