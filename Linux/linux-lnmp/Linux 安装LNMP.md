# Linux 安装LNMP 

标签（空格分隔）： linux-lnmp

[TOC]

---
## 0、安装mysql在 Linux 安装 LAMP的 mysql安装

## 1、安装PHP 
### 1.1 编译PHP
```
./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php/etc --enable-fpm --with-fpm-user=php-fpm --with-fpm-group=php-fpm --with-mysql=/usr/local/mysql --with-mysql-sock=/tmp/mysql.sock --with-libxml-dir --with-gd --with-jpeg-dir --with-png-dir --with-freetype-dir --with-iconv-dir --with-zlib-dir --with-mcrypt --enable-soap --enable-gd-native-ttf --enable-ftp --enable-mbstring --enable-exif --enable-zend-multibyte --disable-ipv6 --with-pear --with-curl --with-openssl

make && make install
```
### 2.1 安装成功后创建用户
```
useradd -s /sbin/nologin -M php-fpm
```
### 2.2 修改配置文件
```
cp php.ini-production /usr/local/php/etc/php.ini
mv /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
> /usr/local/php/etc/php-fpm.conf
vim /usr/local/php/etc/php-fpm.conf
```
把下列内容放入文件中
```
[global]
pid = /usr/local/php/var/run/php-fpm.pid
error_log = /usr/local/php/var/log/php-fpm.log
[www]
listen = /tmp/php-fcgi.sock
user = php-fpm
group = php-fpm
listen.owner = nobody
listen.group = nobody
pm = dynamic
pm.max_children = 50
pm.start_servers = 20
pm.min_spare_servers = 5
pm.max_spare_servers = 35
pm.max_requests = 500
rlimit_files = 1024
slowlog = /tmp/slowlog.log
request_slowlog_timeout = 1 
php_admin_value[open_basedir]=/usr/local/nginx/html:/tmp
```
检查安装是否正确
```
/usr/local/php/sbin/php-fpm -t
```
将php-fpm设置自启动
```
cp /usr/local/src/php-5.3.27/sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
```

## 3、配置Nginx
### 3.1、下载linux版本ngnix
 http://nginx.org/en/download.html  //下面页面 
 http://nginx.org/download/nginx-1.2.9.tar.gz  直接下载文件，目前1.2.9版本比较稳定
### 3.2、安装pcre-devel openssl openssl-devel 服务
> yum -y install gcc pcre-devel openssl openssl-devel  

### 3.3、解压nginx-1.2.9.tar.gz
### 3.4、进入解压的目录执行命令
```
./configure --prefix=/usr/local/nginx --with-http_realip_module --with-http_sub_module --with-http_gzip_static_module --with-http_stub_status_module --with-pcre
```

### 3.5、执行命令
> make install  

这是nginx会自动安装到目录 /usr/local/nginx 下

进入 /usr/local/nginx目录，采用命令
sbin/nginx 命令启动nginx服务

这是可以通过 http://服务器ip/访问
![nginx.png-25.4kB][1]
 
### 3.7 配置nginx.conf
```
user nobody nobody ;
worker_processes 2;
error_log /usr/local/nginx/logs/nginx_error.log crit;
pid /usr/local/nginx/logs/nginx.pid;
worker_rlimit_nofile 51200;

events
{
    use epoll;
    worker_connections 6000;
}

http
{
    include mime.types;
    default_type application/octet-stream;
    server_names_hash_bucket_size 3526;
    server_names_hash_max_size 4096;
    log_format combined_realip '$remote_addr $http_x_forwarded_for [$time_local]'
    '$host "$request_uri" $status'
    '"$http_referer" "$http_user_agent"';
    sendfile on;
    tcp_nopush on;
    keepalive_timeout 30;
    client_header_timeout 3m;
    client_body_timeout 3m;
    send_timeout 3m;
    connection_pool_size 256;
    client_header_buffer_size 1k;
    large_client_header_buffers 8 4k;
    request_pool_size 4k;
    output_buffers 4 32k;
    postpone_output 1460;
    client_max_body_size 10m;
    client_body_buffer_size 256k;
    client_body_temp_path /usr/local/nginx/client_body_temp;
    proxy_temp_path /usr/local/nginx/proxy_temp;
    fastcgi_temp_path /usr/local/nginx/fastcgi_temp;
    fastcgi_intercept_errors on;
    tcp_nodelay on;
    gzip on;
    gzip_min_length 1k;
    gzip_buffers 4 8k;
    gzip_comp_level 5;
    gzip_http_version 1.1;
    gzip_types text/plain application/x-javascript text/css text/htm application/xml;

server
{
    listen 80;
    server_name localhost;
    index index.html index.htm index.php;
    root /usr/local/nginx/html;
    # 静态文件，不记录访问日志 
    location ~ \.(gif|png|css|js|jpg)$ {
       access_log off;
    }

    
    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/tmp/php-fcgi.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME /usr/local/nginx/html$fastcgi_script_name;
    }

}
}
```


### 3.7、配置启动服务
 vim /etc/init.d/nginx

```shell
#!/bin/bash
# chkconfig: - 30 21
# description: http service.
# Source Function Library
. /etc/init.d/functions
# Nginx Settings

NGINX_SBIN="/usr/local/nginx/sbin/nginx"
NGINX_CONF="/usr/local/nginx/conf/nginx.conf"
NGINX_PID="/usr/local/nginx/logs/nginx.pid"
RETVAL=0
prog="Nginx"

start() {
        echo -n $"Starting $prog: "
        mkdir -p /dev/shm/nginx_temp
        daemon $NGINX_SBIN -c $NGINX_CONF
        RETVAL=$?
        echo
        return $RETVAL
}

stop() {
        echo -n $"Stopping $prog: "
        killproc -p $NGINX_PID $NGINX_SBIN -TERM
        rm -rf /dev/shm/nginx_temp
        RETVAL=$?
        echo
        return $RETVAL
}

reload(){
        echo -n $"Reloading $prog: "
        killproc -p $NGINX_PID $NGINX_SBIN -HUP
        RETVAL=$?
        echo
        return $RETVAL
}

restart(){
        stop
        start
}

configtest(){
    $NGINX_SBIN -c $NGINX_CONF -t
    return 0
}

case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  reload)
        reload
        ;;
  restart)
        restart
        ;;
  configtest)
        configtest
        ;;
  test)
        configtest
        ;;
  *)
        echo $"Usage: $0 {start|stop|reload|restart|configtest}"
        RETVAL=1
esac
exit $RETVAL
```
chmod a+x /etc/init.d/nginx
chkconfig --add nginx
chkconfig nginx start
chkconfig nginx on

## Nginx 日志分割
```
#!/bin/bash
d=`date -d "-1 day" +%Y%m%d-$RANDOM%10000`  #系统当前日期加随机数，防止手工运行 fug
[ -d /tmp/nginx_log ] || mkdir /tmp/nginx_log
mv /tmp/access.log /tmp/nginx_log/$d.log
/etc/init.d/nginx reload 2> /dev/null
gzip -f /tmp/nginx_log/$d.log
```

[1]: http://static.zybuluo.com/Great-Chinese/tabid0i5fmjarz9bip4lvu3k/nginx.png

