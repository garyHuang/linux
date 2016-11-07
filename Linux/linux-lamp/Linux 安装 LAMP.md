# Linux 安装 LAMP

标签（空格分隔）： linux-lamp

[toc]

---
## 一、安装Mysql
### 1、下载mysql安装包 到 /usr/local/src
 下载地址： http://dev.mysql.com/downloads/mysql/5.5.html#downloads
 http://cdn.mysql.com//Downloads/MySQL-5.5/mysql-5.5.51-linux2.6-x86_64.tar.gz

### 2、解压安装包
```
tar -zxvf mysql-5.5.51-linux2.6-x86_64.tar.gz
```
### 3、移动并重命名
```
mv mysql-5.5.51-linux2.6-x86_64 /usr/local/mysql
```
### 4、创建不能登录的mysql账号
```
useradd -s /bin/nologin -M mysql
```
### 5、安装mysql
```
mkdir -pv /data/mysql
./scripts/mysql_install_db --user=mysql --datadir=/data/mysql
```
### 6、拷贝my-large.cnf 到 /etc/my.cnf
```
cp support-files/my-large.cnf /etc/my.cnf
```
### 7、复制 mysql.server 到 /etc/init.d/mysqld 文件
```
 cp support-files/mysql.server /etc/init.d/mysqld
```
### 8、添加到系统进程,设置开机启动
```
chmod 755 /etc/init.d/mysqld
 chkconfig --add mysqld
 chkconfig mysqld on
```
### 9、修改mysqld文件
```
basedir=/usr/local/mysql
datadir=/data/mysql
```
### 10、启动mysql
```
service mysqld start
```
### 11、登录mysql和给账号授权
```
登录mysql
mysql -u gary -p 127.0.0.1 -pgary123 
账号授权
grant all on discuz.* to 'gary'@'%' identified by 'gary123' ;

flush privileges;
```

## 二、安装apache
### 说明
  安装apache需要安装 apr和apr-util
### 下载安装 apr 和apr-util
  下载地址： http://apr.apache.org/download.cgi
   编译： apr
   ./configure --prefix=/usr/local/apr/
   make & make install
   编译 : apr-util
   ./configure --prefix=/usr/local/apr-util/ --with-apr=/usr/local/apr/
   make & make install

### 下载编译apache
下载地址 http://mirrors.cnnic.cn/apache/httpd/
## 编译：
./configure --prefix=/usr/local/httpd --enable-so --enable-deflate=shared --enable-expires=shared --enable-rewrite=shared --with-pcre --with-apr=/usr/local/apr/ --with-apr-util=/usr/local/apr-util/
make & make install

### 启动，重启
bin/apachectl start
bin/apachectl restart
bin/apachectl stop
bin/apachectl graceful #重新加载配置，不重启


## 三、安装编译php
### 1、安装编译php所需软件
```
yum install -y libxml2-devel openssl openssl-devel bzip2 bzip2-devel libpng libpng-devel  freetype freetype-devel 
```
### 2.安装 libmcrypt
```
yum -y install epel-release
yum clean all #清理下资源
yum list | grep libmcrypt
yum install -y libmcrypt libmcrypt-devel
```
### 3、下载php
http://php.net/downloads.php#v5.5.38

### 4、编译php
```
 ./configure --prefix=/usr/local/php --with-apxs2=/usr/local/httpd/bin/apxs --with-config-file-path=/usr/local/php/etc  --with-mysql=/usr/local/mysql --with-libxml-dir --with-gd --with-jpeg-dir --with-png-dir --with-freetype-dir --with-iconv-dir --with-zlib-dir --with-bz2 --with-openssl --with-mcrypt --enable-soap --enable-gd-native-ttf --enable-mbstring --enable-sockets --enable-exif --disable-ipv6
```
### 5、make 
### 6、make install 
 
## 四、apache 支持php
### 查看基本配置是否安装成功
   查看modules/libphp5.so文件是否存在，如果不存在重新安装php，查看conf/httpd.conf,中是否有如下配置,如果没有就新增一条记录：
```
LoadModule php5_module        modules/libphp5.so
```
修改配置:
```
 <IfModule dir_module>
    DirectoryIndex index.html index.htm index.php
</IfModule>
```
在 AddType application/x-gzip .gz .tgz 新增一行
```
AddType application/x-httpd-php .php
```
开启Virtual hosts
```
Include conf/extra/httpd-vhosts.conf
```
修改配置文件 conf/extra/httpd-vhosts.conf: 
```
<VirtualHost *:80>
    DocumentRoot "/data/www/"
    ServerName www.aaa.com
    ServerAlias www.bbb.com
    ErrorLog "logs/dummy-host.example.com-error_log"
    CustomLog "logs/dummy-host.example.com-access_log" common
</VirtualHost>
```
创建目录 /data/www 把目录的所有权改为账号 daemon
```
chown -R daemon /data/www/
```
在目录/data/www下创建 1.php 文件
```
<?php
 echo "huangfei"
?>
```
测试是否安装成功,访问地址： http://www.aaa.com/1.php

## 五、apache配置301跳转
### 5.1 检查是否支持url重写
```
bin/apachectl -M
```
 是否支持标志，查看上面命令是否有 rewrite_module (share)，如果没有，就需要在 conf/httpd.conf 文件在加入：
 ```
 LoadModule rewrite_module modules/mod_rewrite.so
 ```
```
<VirtualHost *:80>
    #ServerAdmin webmaster@dummy-host.example.com
    DocumentRoot "/data/www/"
    ServerName www.aaa.com
    ServerAlias www.bbb.com
    ServerAlias www.ddd.com
    ErrorLog "logs/dummy-host.example.com-error_log"
    CustomLog "logs/dummy-host.example.com-access_log" combine
    <IfModule mod_rewrite.c>
      RewriteEngine on
      RewriteCond %{HTTP_HOST} ^www.bbb.com$ [OR]
      RewriteCond %{HTTP_HOST} ^www.ddd.com$
      RewriteRule ^(.*)$ http://www.aaa.com$1 [R=301,L]
    </IfModule>
</VirtualHost>

```
## 六、日志分割配置,每天生成一个日志文件
```
 CustomLog "|/usr/local/httpd/bin/rotatelogs -l /usr/local/httpd/logs/www.aaa.com-%Y-%m-%d_access_log 86400" combine
```
## 七、不记录静态文件到访问日志文件里
```
SetEnvIf Request_URI ".*\.css$" image-request
SetEnvIf Request_URI ".*\.js$" image-request
SetEnvIf Request_URI ".*\.jpg$" image-request
SetEnvIf Request_URI ".*\.png$" image-request
SetEnvIf Request_URI ".*\.gif$" image-request
CustomLog "|/usr/local/httpd/bin/rotatelogs -l /usr/local/httpd/logs/www.aaa.com-%Y-%m-%d_access_log 86400" combined env=!image-request
```
## 八、配置静态缓存,配置在 http-vhost.conf 的 VirtualHost 节点下
```
<IfModule mod_expires.c>
      ExpiresActive on
      ExpiresByType image/gif "access plus 1 days"
      ExpiresByType image/jpeg "access plus 24 hours"
      ExpiresByType text/css "now plus 12 hours"
      ExpiresByType image/png "access plus 24 hours"
      ExpiresByType application/x-javascript "now plus 2 hours" 
</IfModule>
```

## 九、设置防盗链
```
SetEnvIfNoCase Referer "^http://.*\.aaa\.com" local_ref
SetEnvIfNoCase Referer "^http://.*\.apelearn\.com" local_ref
<filesmatch "\.(txt|doc|mp3|rar|zip|jpg|gif|png|css|js)" >
    Order Allow,Deny
    Allow from env=local_ref
</filesmatch>
```

## 十、访问控制
```
<Directory "/data/www/">
    AllowOverride None
    Options None
    Order allow,deny #标识执行顺序 首先执行allow的规则，然后执行deny规则
    Allow from all   # 标识允许所有
    Deny from 127.0.0.1 # 禁止127.0.0.1的端口
</Directory>
```
## 十一、限制某个目录下的php文件不被解析
```
<Directory "/data/www/data/">
    php_admin_flag engine off
    <filesmatch "(.*)php" >
        Order allow,deny
        Deny from all
    </filesmatch>
</Directory>
```
## 十二、限制ip访问 httpd-vhost.conf，最前面
```
<VirtualHost *:80>
  ServerName test123.com
  <filesmatch ".*">
     Order Allow,Deny
     Allow from all
     Deny from all
  </filesmatch>
</VirtualHost>
```
## 十三、rewrite限制访问某个目录
- [ ] RewriteRule .* - [F]   禁止访问，返回403错误
- [ ] RewriteRule ^(.*)$ http://www.aaa.com/ 指定跳转到 首页
```
 <IfModule mod_rewrite.c>
      RewriteEngine on
      RewriteCond %{HTTP_HOST} ^www.bbb.com$ [OR]
      RewriteCond %{HTTP_HOST} ^www.ddd.com$
      RewriteRule ^(.*)$ http://www.aaa.com$1 [R=301,L]
      RewriteCond %{REQUEST_URI} ^.*/tmp/.* [NC]
      #RewriteRule .* - [F]  
      RewriteRule ^(.*)$ http://www.aaa.com/
      RewriteCond %{REQUEST_URI} /index\.(html|jsp|html|php) [NC]
      RewriteRule ^(.*)$ http://www.aaa.com/ [R=301,L]
    </IfModule>
```
## 十四、限制浏览器访问
```
RewriteCond %{HTTP_USER_AGENT} ^.*curl.* [NC]
RewriteRule .* - [F]
```

## 十五、php.ini 配置文件
### php配置日志
 php5.3以前（包括5.3版本），php.ini 文件在PHP_HOME/etc 目录下，5.4版本以后（包括5.4） php.ini 文件需要从源码目录拷贝到 PHP_HOME/etc。php.ini 默认有两个版本 php.ini-development(开发版)和php.ini-production（生产版本）
配置禁用函数
```
disable_functions = eval,assert,popen,passthru,escapeshellarg,escapeshellcmd,exec,system,chroot,scandir,chgrp,chown,shell_exec,proc_get_status,ini_alter,ini_restore,dl,pfsockopen,openlog,syslog,readlink,symlink,leak,popepassthru,stream_socket_server,proc_open,proc_close
```
配置开启日志
```
display_errors = On 
On表示开始日志，Off表示关闭日志
log_errors = On 
On 表示日志写到文件里面，Off表示日志不写到文件
error_lor = /data/www/logs/php_error.log
日志文件写的路径，绝对路径,注明 /data/www/logs/ 的所属必须是 daemon， 否则没有权限够
```
设置open_basedir，设置php有权限操作的目录
```
open_basedir = /data/www:/tmp
```
apache 设置每个虚拟主机有权限访问的目录
```
 php_admin_value open_basedir "/data/www:/tmp"
```

## 十六、php扩展包安装
 去php源码目录下的，ext 目录，找到要安装的扩展包，进入该目录，然后运行命令：
 ```
 /usr/local/php/bin/phpize 
 ```
 运行完成后会生成一个 configure 文件 ，然后运行 
 ./configure --with-php-config=/usr/local/php/bin/php-config
 make && make install
 会在/usr/local/php/lib/php/extensions/,生成一个目录
 最后在 php.ini 文件中加入,完成插件安装
 ```
 extension=curl.so
 ```
## 十七、MySql忘记密码重置方法
 在/etc/my.conf,mysqld节点下加 skip-grant，然后重启mysql服务，执行下面命令。就可以重置mysql命令，重置完成 不要忘记去掉my.conf 文件中的 skip-grant 
 ```
 mysqladmin flush-privileges password '123456'
 ```
 ## 十八、nginx缓存配置
 ```
  #图片等静态文件缓存时间为 1天
  location ~ \.(gif|png|jpg|jpeg|bmp)$ {
       access_log off;
       expires 1d ;
    ｝
    # js和css缓存时间为两小时
    location ~ \.(js|css) {
       access_log off;
       expires 2h; 
    }
 ```
 ## 十九、防盗链
 ```
 location ~ \.(gif|png|jpg|jpeg|bmp)$ {
   access_log off;
   expires 1d ;
   valid_referers none blocked *.aaa.com;
   if ($invalid_referer){
       return 403;
   }
}
 ```
 ## 访问控制
 ```
 deny 127.0.0.1;  # 拒绝某个ip访问
 deny 192.168.31.0/34; #拒绝 192.168.31.0 整个网段
 allow 192.168.31.27 # 允许这个ip访问
 ```
 ## 拒绝某些蜘蛛爬虫
 ```
 if ($http_user_agent ~* 'curl|baidu|111111') {
      return 403;
 }
 ```
 