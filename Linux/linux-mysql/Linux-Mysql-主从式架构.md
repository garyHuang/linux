# Linux-Mysql-主从式架构

 - 列表项

标签（空格分隔）： linux-mysql

[toc]

---
## 1 说明
> 主从式架构 (Client–servermodel) 或客户端-服务器（Client/Server）结构简称C/S架构，是一种网络架构，它把客户端 (Client)（通常是一个采用图形用户界面的程序）与服务器(Server) 区分开来。每一个客户端软件的实例都可以向一个服务器或应用程序服务器发出请求。
 
## 2 安装mysql
### 2.1 下载
  我使用 mysql-5.1.40-linux-x86_64-icc-glibc23.tar.gz 可在mysql官网下载,解压到目录/usr/local/mysql，复制 /usr/local/mysql_slave

### 2.2 创建mysql账户
```
useradd -s /bin/nologin -M /mysql
```
### 2.3 拷贝my-small.cnf 到当前目录下
```
cp support-files/my-small.cnf my.cnf
```
### 2.4 拷贝 mysql.server 到/etc/init.d/mysqld 和 /etc/init.d/mysqldSlave
```
cp support-files/mysql.server /etc/init.d/mysqld
cp support-files/mysql.server /etc/init.d/mysqldSlave
```
### 2.5 修改 /etc/init.d/mysqld ,和 /etc/init.d/mysqldSlave
/etc/init.d/mysqld 修改配置和属性
```
basedir=/usr/local/mysql
datadir=/data/mysql
conf=${basedir}/my.cnf
```
/etc/init.d/mysqldSlave 修改配置和属性
```
basedir=/usr/local/mysql_slave
datadir=/data/mysql_slave
conf=${basedir}/my.cnf
```
### 2.6 修改各自的my.cnf文件
mysql/my.cnf
```
[mysqld]
port            = 3306
socket          = /tmp/mysql.sock
server-id       =1
log-bin=gary

```
mysql_slave/my.cnf
```
[mysqld]
port            = 3307
socket          = /tmp/mysql_slave.sock
server-id       =2
replicate-ignore-db=mysql
```
### 2.7 初始化mysql
mysql
```
./scripts/mysql_install_db --user=mysql --datadir=/data/mysql
```
mysql_slave
```
./scripts/mysql_install_db --user=mysql --datadir=/data/mysql_slave
```
### 2.8 启动 /usr/local/mysql ，链接该mysql
```
/etc/init.d/mysql start
/usr/local/mysql -S /tmp/mysql.sock
```
### 2.9 创建主数据库复制账号
```
mysql> grant replication slave on *.* to 'repl'@'127.0.0.1' identified by '123456' ;
mysq> flush privileges ;
```
### 2.10 暂时锁定主数据库
```
flush tables with read lock ;
```
### 2.11 查看主数据状态
```
mysql> show master status ; 
+-------------+----------+--------------+------------------+
| File        | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+-------------+----------+--------------+------------------+
| gary.000003 |      187 |              | mysql,mysql      |
+-------------+----------+--------------+------------------+
```
### 2.12 启动mysql_slave
```
mysql> slave stop ;
mysql> change master to master_host='127.0.0.1',master_port=3306,master_user='repl',master_password='123456',master_log_file='gary.000003',master_log_pos=187 ;
mysql> slave start ;
```
### 2.13 查看主从复制是否搭建成功
在slave数据库上运行
```
 show slave status \G 
```
如果看到 Slave_IO_Running和 Slave_SQL_Running 都是yes说明主从数据库搭建成功了
### 2.14 测试是否搭建成功
重新登录，mysql主站。创建数据库，创建测试.

### 2.14 特别注意，从数据库只能提供只读账号，如果修改错误删除了数据，会导致无法主从同步
> grant select on *.* to 'root'@'%' identified by '123456' ;


## Mysql事务
### 查看Mysql事务
```
SELECT @@global.tx_isolation ;
SELECT @@session.tx_isolation ;
```
### 设置事务级别
```
SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ ;
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ ; 
```
### 四大事务级别
```
READ UNCOMMITTED 读取不提交
Read committed  读取提交
Repeatable read 重复读取，Mysql默认事务
Serializable 事务最高级别，不可使用
```

### 查看mysql事务是否自动提交
0为手动提交，1为自动提交
```
select @@autocommit;
```
### 设置手动提交
```
set autocommit = 0;
COMMIT  ## 提交事务
rollback ## 回滚
```
### 设置自动提交
```
set autocommit = 1;
```
