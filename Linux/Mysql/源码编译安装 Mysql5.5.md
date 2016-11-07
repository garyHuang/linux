#  源码编译安装 Mysql5.5

标签（空格分隔）： Mysql

[TOC]

---
## 1.1下载源码
去 http://mirrors.sohu.com/mysql/ 现在需要的Mysql版本 例如： mysql-5.5.51.tar.gz  
## 1.2安装编译相关软件
```
yum install -y cmake ncurses-devel gcc gcc-c++ ncurses libaio bison 
```
## 1.2创建数据目录和Mysql运行账号
```
mkdir /data/mysql
groupadd mysql           
useradd -g mysql mysql
```
## 1.3解压下载目录
```
tar -xvf mysql-5.5.51.tar.gz
```
## 1.4 make 编译
```
time cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql5 -DMYSQL_DATADIR=/data/mysql5 -DWITH_INNOBASE_STORAGE_ENGINE=1 -DMYSQL_UNIX_ADDR==/usr/local/mysql5/mysql.sock -DMYSQL_USER=mysql -DDEFAULT_CHARSET=utf8 -DDEFAULT_C
# 编译安装
make && make install
```
## 1.5修改配置文件my.cnf
```
[client]
port		= 3311
socket		= /data/mysql5/mysql.sock
# The MySQL server
[mysqld]
datadir = /data/mysql5
port		= 3311
socket		= /data/mysql5/mysql.socket
# 设置分配给数据库内存的50-70%
innodb_buffer_pool_size=1024M
# mysql使用独立表空间
innodb_file_per_table=1
innodb_log_files_in_group=2
# mysql数据文件大小，设置为1G
innodb_data_file_path=ibdata1:1G:autoextend
# 日志文件大小设置为512M
innodb_log_file_size=512M
#记录数据库全部操作，不建议开启。影响数据库5%的性能
general_log=0
# 日志操作记录文件
general_log_file=/data/mysql5/local01.log
# 开启慢日志查询
slow_query_log = 1
# 执行时长超过1S的SQL语句昨晚慢日志
long_query_time = 1
# 记录没有使用索引的查询,不建议开启，开启后分析慢日志不方便
log_queries_not_using_indexes=0
# 设置mysql最大连接数
max_connection=3000
#max_connection_error=100000
# 文件打开数量 为max_connection的10倍
open_files_limit=30000
innodb_open_files=30000
table_open_cache=30000
table_definition_cache=3000
# 这个要尽量的小
key_buffer_size=32M
#禁用查询缓存
query_cache_type=0
#查询缓存大小为0M
query_cache_size=0
# 设置数据库默认引擎
default-storage-engine=INNODB

# bin log 配置
max_binlog_size=67108864 # 配置
binlog_cache_size=1048576

[mysqldump]
quick
max_allowed_packet = 16M
[mysql]
no-auto-rehash
# Remove the next comment character if you are not familiar with SQL
#safe-updates
[myisamchk]
key_buffer_size = 128M
sort_buffer_size = 128M
read_buffer = 2M
write_buffer = 2M
[mysqlhotcopy]
interactive-timeout
```
## 1.6初始化Mysql
```
./scripts/mysql_install_db --user=mysql --datadir=/data/mysql5
```
## 1.7启动Mysql
```
cp support-files/mysql.server /etc/init.d/mysqld5
# 打开文件/etc/init.d/mysqld5 设置datadir 和basedir
# centos6 启动Mysql
chkconfig add mysqld5
service mysqld5 start
# centos7 启动Mysql
systemctl enable mysqld5
systemctl start mysqld5
```
## 1.8 Mysql字符集
```
#查看当前数据库的字符集
SHOW VARIABLES LIKE '%character%'
# 命令行查看当前数据库的字符集
status
# 查看当前数据库支持的字符集
SHOW CHARACTER SET 
```

