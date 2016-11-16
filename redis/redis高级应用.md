###1、redis简介
     redis是一个key-value存储系统。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。

###2、安装
####2.1下载软件包，解压目录，进入解压目录，使用make命令
```
make
#测试是否make成功
make test 
#安装到 /usr/local/redis 目录下
make PREFIX=/usr/local/redis install
```
####3、配置说明
> * daemonize 如果需要在后台运行将值设置为yes
> * pidfile 配置多个pid地址，你让/var/run/redus.pid
> * dir 配置数据文件保存的目录
> * bind 绑定ip，设置后只接收来之该ip的访问
> * port 监听端口，默认为6379
> * timeout 设置客户端链接超时时间，单位为秒
> * loglevel 分为4级别， debug 、 verbose 、 notice 、 waring
> * logfile 日志文件地址
> * databases 设置数据库个数默认使用数据库为16
> * save 设置redis进行数据库镜像的频率
> * dbfilename 数据库文件名称
> * requirepass 设置redis密码
> * 

####4、api说明
```
set name lijie //设置一个name为lijie的键值对
setnx name lijie //设置一个键值对，这个name必需是不存在的
setex name 10 red //设置一个键值对，有效期是10秒钟
keys map* //查看所有map开头的键，支持正则
EXISTS name //判断name键是否存在，存在返回1，不存在返回0
del name  //删除那么键，成功返回1，失败返回0
expire name 10 //设置key的过期时间,设置过期时间为10秒
PERSIST name //取消过期时间
ttl name  //获取过期时间，返回-2已经过期或者不存在，返回-1，没有设置过期时间
rename name newName //重命名
type name //返回键的数据类型
ping //测试redis客户端是否能正常链接redis服务器端
CONFIG GET timeout //返回redis配置的值
FLUSHDB //清空当前数据库的所有键
flushall //清空所有数据库中的所有键
select 1 //切换数据库，其中1是数据库索引
```
####5、检查集群延迟
./redis-cli -h 192.168.1.21 --latency

####6、redis设置账号密码
修改配置文件，找到 requirepass foobared，在下面增加一行(注明：密码最好设置复杂一点，redis一秒种可以接收150K次的请求，简单密码很容易破解)
```
requirepass 123456
```
设置完成后重启
设置完成后，客户端链接：
./redis-cli -h 192.168.1.21 -a 123456
或者: ./redis-cli -h 192.168.1.21 后，输入 auth 123456
####7、redis事务处理
MULTI //开启事务，接下来可以执行任意多条命令,如下。
set a 10026
incr a
exec //直接提交事务
DISCARD //回滚事务
####8、redis乐观锁
 WATCH a //监控某个key
 muti    // 启动事务
 set a  b //设置不一样的值
 exec    //提交任务，当执行完毕后，会自动释放watch。如果再次监控这个值，就需要重新监控,如果提交事务的时候发现a的值已经呗修改了，则不会再次修改a的值

####9、 appendonly
 当设置appendonly的值为yes的时候，系统会自动将每次写的命令放入到appendonly.aof，文件中。
 写入这个文件的规则是：
 appendfsync everysec //每秒钟写如一次
 appendfsync always //实时写入
 appendfsync no  //从不写入

####11、redis广播和订阅
SUBSCRIBE tv1 //监听某个key，可以是多个
PUBLISH tv1 hello //想tv1发布hello消息，这样所有订阅tv1的客户端都能接收到这个消息。