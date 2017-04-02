### Linux Redis Make and Install
redis是当前比较热门的NOSQL系统之一，它是一个key-value存储系统。和Memcached类似，但很大程度补偿了memcached的不足，它支持存储的value类型相对更多，包括string、list、set、zset和hash。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作。在此基础上，redis支持各种不同方式的排序。Redis数据都是缓存在计算机内存中，并且会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件。
 
#### 下载编译
从[http://download.redis.io/releases/](http://download.redis.io/releases/)下载相应的版本

	wget http://download.redis.io/releases/redis-3.0.5.tar.gz
	tar -zxvf redis-3.0.5.tar.gz -C ./
	cd redis-3.0.5
	make

![](https://github.com/silence940109/Java/blob/master/Linux_Redis_Make/image/redis_make.png)

#### 拷贝文件
编译完成后，在Src目录下，有四个可执行文件redis-server、redis-benchmark、redis-cli，在上一级上找到redis.conf,拷贝到新建的目录下

![](https://github.com/silence940109/Java/blob/master/Linux_Redis_Make/image/redis.png)

可以在在redis.conf中修改登录密码

	requirepass redis

默认是没有打开的

![](https://github.com/silence940109/Java/blob/master/Linux_Redis_Make/image/redis-password.png)

redis-server启动Redis服务器

可以在客户端连接，验证

![](https://github.com/silence940109/Java/blob/master/Linux_Redis_Make/image/redis-cli.png)
