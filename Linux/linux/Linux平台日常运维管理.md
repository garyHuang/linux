# Linux平台日常运维管理

标签（空格分隔）： linux

---
## 基本查看
### 常用命令 w
```
12:32:36 up 196 days, 19:24,  2 users,  load average: 0.14, 0.08, 0.06
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
hks      pts/0    119.129.208.233  12:26    4.00s  0.08s  0.00s w
```
- [ ] w命令输出 说明：
 - [x] 用两个逗号隔开 三个数字，表示 1分钟内cpu的平均使用率，5分钟内，和15分钟内
 - [x] 如果是4核cpu，0.14表示 当前只有一个cpu在工作，使用率是 14% 
 - [x] 12:32:36 up 196 days, 19:24 系统运行的时长
 - [x] 第二行和第三行表示 当前登录的用户列表，从那个ip登录 等信息

### uptime 命令
**该命令只出现w命令的第一行，如果仅仅只查询负载可以使用uptime命令**
```
12:32:36 up 196 days, 19:24,  2 users,  load average: 0.14, 0.08, 0.06
```
 
## 查看系统cpu的核数，出现的个数就是CPU的核数
```
 cat /proc/cpuinfo | awk -F ':' '$1~/processor/{print $1 , $2}' 
```

## vmstat
**vmstat命令是最常见的Linux/Unix监控工具，可以展现给定时间间隔的服务器的状态值,包括服务器的CPU使用率，内存使用，虚拟内存交换情况,IO读写情况。**

### vmstat 1 5 
**1表示没秒钟采集一次系统运行情况，5表示采集5次**
```
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0      0 33515812 524712 19362504    0    0     0     7    0    0  3  0 97  0  0
 1  0      0 33515672 524712 19362504    0    0     0    16  903 1691  4  0 96  0  0
 1  0      0 33515672 524712 19362504    0    0     0     0 1002 1741  6  0 94  0  0
 1  0      0 33515556 524712 19362504    0    0     0     0 1072 1819  7  0 93  0  0
 1  0      0 33515640 524712 19362504    0    0     0    28 1006 1742  6  0 94  0  0
```
说明： r 表示运行的队列
b 表示阻塞的进程
swpd 表示虚拟内存使用的大小，如果大于0表示物体内存不足

## sar -n DEV 1 5
 说明： 表示秒钟打印一次，连续打印5次
如果提示 Cannot open /var/log/sysstat/sa05: No such file or directory
第一步：首先修改：/etc/default/sysstat 修改 ENABLED="true"
第二步：sar -o 23
第三步： sar -n DEV 1 5

## ps -ef  或者 ps aux  
过滤某个进程是否启动用， grep

##  netstat
- [ ] netstat -lnp 
 - [x] 查看监听的端口
- [ ] netstat -an 
 - [x] 查看链接状态
- [ ] netstat -an | grep 10.45.236.4:8002 | grep -i ESTABLISHED
 - [x] 统计并发的列表
- [ ] netstat -an | grep 10.45.236.4:8002 | grep -ic ESTABLISHED
 - [x] 统计并发数 

## tcpdump -nn -c 100
- [ ] -c 100 抓100个包
- [ ] tcpdump -nn port 22 #抓22端口的包
- [ ] -w 2.cap #写入到文件

## tshark 
- [ ] yum install -y wireshark #安装工具
- [ ] tshark -n -t a -R http.request -T fields -e "frame.time" -e "ip.src" -e "http.host" -e "http.request.method" -e "http.request.uri"

## selinux
- [ ] 临时关闭selinux的方法是setenforce 0
- [ ]  永久关闭selinux的方法是编辑配置文件/etc/selinux/config, 把SELINUX=Enforceing 改成 SELINUX=Disabled

## iptables 防火墙
### Centos 7 安装 iptables-services
> yum install iptables-services

- [ ]  iptables -t filter -I INPUT -p tcp --dport 80 -s 192.168.31.138 -j REJECT
 - [x] -t 指定表，有三张表  filter,nat,mangle 默认是 filter表
 - [x] -I 为插入，新增加的规则会在规则列表的最上面出现
 - [x] -A 为增加，新增加规则会在规则列表的最下面出现
 - [x] -p 指定协议
 - [x] --dport 80 指定端口,没有必需有-p 指定协议
 - [x] -s 指定来源ip地址
 - [x] -j 指定规则，ACCEPT 允许，REJECT 拒绝 , DROP 允许，不记录任何操作
 - [x] iptable -F 清空防火墙规则
 - [x] iptables-restore < 1.ipt #恢复防火墙规则
 - [x] iptable-save > 1.ipt #保存防火墙规则到1.ipt文件中
 - [x]  filter表 主要用来限制进入本机的包和出去的包
 - [x] nat表 主要用于网络地址转换，比如家用的小路由器就是用nat表实现的
 - [x] mangle表 主要用来给包打标记
 
## cron 定时任务
- [ ] crontab -l
 - [x] 查看当前用户的定时任务列表,-u指定用户账号 如： crontab -u root -l
- [ ]  crontab -e 
 - [x] 编辑当前用户的定时任务列表  
 - [x] */1 * * * * /bin/bash /opt/shell/test.sh
```
*/1 * * * * /bin/bash /opt/shell/test.sh
分 时 月 年 周 改命令表示，每分钟执行一次test.sh文件
```

## chkconfig
- [ ] chkconfig --list
 - [x] chkconfig 查看所有的服务列表 
- [ ] chkconfig atd off 
 - [x] 将atd的服务器关闭，将会影响 345级别的开机启动
- [ ] chkconfig atd on
 - [x] 将atd的服务器开启，将会影响345基本的开机启动 
- [ ]  chkconfig --level 345 atd on
 - [x] 设置 atd 345 开机启动
- [ ] 将可执行文件拷贝到 /etc/init.d/ 目录下
 - [x] chkconfig --add 123 #添加123到系统服务器
 - [x] chkconfig --del 123 # 删除123的系统服务器
- [ ] chkconfig --list atd #查看atd的启动状态

## linux 日志
- [ ] /var/log/messages
- [ ] /var/log/secure
- [ ] /var/log/dmesg
- [ ] last  #记录登录信息
- [ ] lastb #记录错误登录信息

## screen
- [ ] screen sleep 100 # 按Ctrl+A+D 放在后台允许
- [ ] screen -r pid 进入这个进程允许状态
- [ ] screen -wipe 清楚被kill掉的无效进程
- [ ] screen -ls 查看所有 screen 允许的进程

## rsync
- [ ] rsync -avLuPz --exclude="*.xml" --delete tomcat8/ tomcat001/
    - [x] -a 拷贝
    - [x] v 显示拷贝的文件
    - [x] P 显示拷贝文件进度
    - [x] z 压缩拷贝
    - [x] L 拷贝 软件链接的目标文件
    - [x] u 如果目标文件比源文件还新，那么则忽略该文件
    - [x] --exclude 匹配这个规则的文件例外不拷贝
    - [x] --delete 让两个目录下的文件保持一直，tomcat8下删除的文件 在 tomcat001 下也删除 
- [ ] rsync -avLuPz /opt/tomcat8/  192.168.31.213:/opt/tomcat

## rsync 服务
### rsync 方式1
- [ ] 新增配置文件 /etc/rsync.conf， rsync服务默认读取这个配置文件
```
port=8730
log file=/var/log/rsync.log
pid file=/var/run/rsync.pid
[hf]
path=/opt/
user chroot=true
max connection=4
read only=no
list=yes
uid=root
gid=root
auth users=root
secrets file=/etc/rs.pwd
hosts allow=192.168.1.30
```
 - [x] port 端口，默认端口是 8730
 - [x] log file 日志文件
 - [x] pid file pid保存的目录
 - [x] [hf] 模块名称,拉去文件的时候需要指明
 - [x] path rsync同步的路径
 - [x] user chroot 使用目录权限
 - [x] max connection 最大连接数
 - [x] read only 是否只读
 - [x] list 是否允许把模块的名字列出来
 - [x] uid 用哪个用户的身份推送用户
 - [x] gid 推送的组
 - [x] auth users 验证rsync的用户名
 - [x] secrets file 验证密码文件
 - [x] hosts allow 允许的hosts
 - [x] secrets file 密码文件格式：用户名是 auth users 指定的账户名，密码文件必须是600，或者是400
```
 root:123aaa
```
- [ ] rsync --dameon #启动rsync

- [ ] rsync -avzPL --port 8730 root@192.168.1.25::hf/ /opt/
 - [x]  --port 指定端口号
 - [x] root rsync配置的账号 auth users 的账户名
 - [x] hf 模块名称
 - [x] hf/ 从hf模块的根目录拷贝
 - [x] /opt/ 拷贝到 这个目录

- [ ] rsync -avzPL --port 8730 --password-file=/etc/rs.pwd root@192.168.1.25::hf/ /opt/
 - [x] /etc/rs.pwd #该文件里面只写密码，并且权限要设置为400

 