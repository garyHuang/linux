# linux 服务部署与优化（ftp）

标签（空格分隔）： linux

[toc]

---
## 1、pure-ftp 
### 1.1 pure-ftp 下载pureftp
```
cd /usr/local/src/
wget http://download.pureftpd.org/pub/pure-ftpd/releases/pure-ftpd-1.0.42.tar.bz2
```
### 1.2 安装pure-ftp
```
tar jxf pure-ftpd-1.0.42.tar.bz2
cd pure-ftpd-1.0.42
./configure --prefix=/usr/local/pureftpd --without-inetd --with-altlog --with-puredb --with-throttling --with-peruserlimits --with-tls
make && make install
```
### 1.3 修改配置文件 
```
cd /usr/local/src/pure-ftpd-1.0.42/configuration-file
mkdir -p /usr/local/pureftpd/etc/ #创建etc配置文件目录
cp pure-ftpd.conf    /usr/local/pureftpd/etc/pure-ftpd.conf #拷贝配置文件
cp pure-config.pl    /usr/local/pureftpd/sbin/pure-config.pl #拷贝启动文件到sbin目录
chmod a+x /usr/local/pureftpd/sbin/pure-config.pl  #给启动文件授可执行权限
```
### 1.4 pure-ftpd.conf 文件内容
```
ChrootEveryone              yes
BrokenClientsCompatibility  no
MaxClientsNumber            50
Daemonize                   yes
MaxClientsPerIP             8
VerboseLog                  no
DisplayDotFiles             yes
AnonymousOnly               no
NoAnonymous                 no
SyslogFacility              ftp
DontResolve                 yes
MaxIdleTime                 15
PureDB                        /usr/local/pureftpd/etc/pureftpd.pdb
LimitRecursion              3136 8
AnonymousCanCreateDirs      no
MaxLoad                     4
AntiWarez                   yes
Umask                       133:022
MinUID                      100
AllowUserFXP                no
AllowAnonymousFXP           no
ProhibitDotFilesWrite       no
ProhibitDotFilesRead        no
AutoRename                  no
AnonymousCantUpload         no
PIDFile                     /usr/local/pureftpd/var/run/pure-ftpd.pid
MaxDiskUsage               99
CustomerProof              yes
```
### 1.5、启动
```
/usr/local/pureftpd/sbin/pure-config.pl /usr/local/pureftpd/etc/pure-ftpd.conf
```
### 1.6 创建账号
 创建成功后，可用ftp通过21端口登录
```
mkdir /data/www/
useradd www
chown -R www:www /data/www/
/usr/local/pureftpd/bin/pure-pw useradd ftp_user1  -uwww -d /data/www/ #提示输入密码
/usr/local/pureftpd/bin/pure-pw mkdb #创建密码文件，保存密码
/usr/local/pureftpd/bin/pure-pw list #查看当前的用户信息列表
/usr/local/pureftpd/bin/pure-pw  userdel ftp_user2 #删除当前的ftp账号
```

## 2、vsftp 安装
### 2.1 安装 vsftpd 服务器
```
yum install -y vsftpd
chkconfig --add vsftpd
chkconfig vsftpd on
```
### 2.2 修改配置文件 /etc/vsftpd/vsftpd.conf
```
chroot_local_user=YES
```
### 2.3 创建ftp虚拟账号
```
useradd -r /bin/nologin virftp
```
### 2.4 创建ftp登录虚拟账号密码文件
```
 touch /etc/vsftpd/vsftpd_login
  文件内容为：(#号后面部分不要写入文件)
testuser1 #第一个ftp账号
111222aaa #第一个ftp密码
testuser2 #第二个ftp账号
123456    #第二个ftp账号密码
```
### 2.5、创建密码文件
```
chown 600 /etc/vsftpd/vsftpd_login
db_load -T -t hash -f /etc/vsftpd/vsftpd_login /etc/vsftpd/vsftpd_login.db
```
### 2.6、创建vsftpd用户配置文件
 创建目录 /etc/vsftpd/vsftpd_user_conf, 在该目录下，创建和用户名相同的文件，首先创建testuser1文件，内容如下：
```
# 账号访问的固定目录
local_root=/home/virftp/test1  
# 禁止匿名登录
anonymous_enable=NO
# 有可写权限
write_enable=YES
# 上传账号的umask
local_umask=002
# 可上传
anon_upload_enable=YES
# 可写
anon_mkdir_write_enable=YES
# 登录后，多久不操作 自动超时
idle_session_timeout=6000
# 链接超时时间
data_connection_timeout=300
# 最大客户端数
max_clients=5
# 最大ip书
max_per_ip=5

local_max_rate=500000
```
创建后，/etc/vsftpd 的目录机构
```
.
├── ftpusers
├── user_list
├── vsftpd.conf
├── vsftpd_conf_migrate.sh
├── vsftpd_login
├── vsftpd_login.db
└── vsftpd_user_conf
    └── testuser1
```
### 2.7、 修改配置文件 /etc/vsftpd/vsftpd.conf,内容如下：
```
anonymous_enable=NO
anon_upload_enable=NO
anon_world_readable_only=NO
anon_mkdir_write_enable=NO
anon_other_write_enable=NO
chown_uploads=NO
anon_umask=077

chroot_local_user=YES
local_enable=YES
write_enable=YES
local_umask=002
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES

pam_service_name=vsftpd
userlist_enable=NO
tcp_wrappers=YES
listen=YES
guest_enable=YES
guest_username=virftp
virtual_use_local_privs=yes
user_config_dir=/etc/vsftpd/vsftpd_user_conf
```
### 2.8、创建服务
启动成功后，可以通过 ftp客户端链接
```
 chkconfig --add vsftpd
 chkconfig vsftpd on
 service vsftpd start
```
### 2.9、授权非根目录下的目录给该账号访问
在 /home/virftp/test1 下创建 www目录
mkdir -pv /home/virftp/test1/www
执行命令：
mount --bind /home/virftp/test1/www /data/www/
执行完成后，用ftp工具登录，就可以访问到非当前目录的文件。

## 3、安装samb文件共享
### 3.1 安装
```
yum install -y samba samba-client
```
### 3.1 配置 /etc/samba/smb.conf
```
[global]
 # 文件系统共享的工作租
 workgroup = WORKGROUP 
 # 服务器版本，可以自定义
 server string = Samba Server Version %v
 #日志文件的位置
 log file = /var/log/samba/log.%m
 #日志文件最大多少
 max log size = 50
 # user 通过用户名验证， share 匿名可以访问 server 认证的工作给另外一个服务器验证 domain 通过有域控制控制
 security = user
 # 用户名和密码存储方式
 passdb backend = tdbsam
 
# 打印机配置，不该
load printers = yes
cups options = raw

# 家目录配置,不动
[homes]
        comment = Home Directories
        browseable = no
        writable = yes 
# 打印机，不动
[printers]
        comment = All Printers
        path = /var/spool/samba
        browseable = no
        guest ok = no
        writable = no
        printable = yes

#自定义配置共享的目录
[garylinux]
        comment = share for users
        path = /tmp/sambadir/
        writeable = yes
        browseable = yes
        public = no
```
### 3.2 新增登录账号
```
useradd sambuser #新增系统账号
pdbedit -a sambuser #新增账号映射samb的登录账号
pdbedit -L #查看samba所有账号
```
### 3.3 重启
```
/etc/init.d/smb restart
```
### 3.4 挂载
```
mount -t cifs -o username=sambuser,password=123456 //192.168.1.23/garylinux /data/samb/
```
### 3.5 smb-client 客户端登录
```
smbclient -U sambuser //localhost/garylinux/
```

