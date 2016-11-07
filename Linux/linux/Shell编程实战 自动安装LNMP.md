# Shell编程实战 自动安装LNMP

标签（空格分隔）： linux

[toc]

---
### 共用函数脚本funs.sh
```
#!/usr/bin/env bash
## 检查上一步命令释放执行成功
check_ok() {
 if [ $? != 0 ]
 then
  echo "Error,check the error log"
  exit 1
 fi
}
## 获取用户输入
read_input(){
 read -p "$1" val
 echo $val >> /dev/null
}
## 使用yum安装工具
yum_install(){
 echo "checking....$1"
 if ! rpm -qa|grep -q "^$1" ; then
  echo "installing... $1"
   yum install -y $1
   check_ok
 fi
}
ar=`arch`

select_soft(){
  #clear
  print_info
  echo "请选择安装的软件:"
  select item in MySql PHP Nginx EXIT
  do
   case $item in
     MySql)
       install_mysql
       break ;
       ;;
     PHP)
      install_php
      break;
      ;;
     Nginx)
     install_nginx
      break;
       ;;
     Apache)
      break;
       ;;
     EXIT)
      exit 0
      ;;
     *)
      echo "选择错误，请重新选择"
      ;;
   esac
  done
  select_soft
}
print_info(){
 echo "
*****************************************************
* 1 安装Mysql                                       *
* 2 安装PHP                                         *
* 3 安装Nginx                                       *
* 4 退出                                            *
*****************************************************
"
}
```
### 安装mysql函数脚本 mysql.sh
```
#!/usr/bin/env bash
## 安装mysql的函数
install_mysql(){
 mysql_data_dir=/data/mysql
 passwd=123456
 ## 新增mysql账号
 if ! grep '^mysql:' /etc/passwd ; then
    useradd -M mysql -s /sbin/nologin
 fi
 
 if [ -d $install_dir/mysql ] ; then
   echo -e "\e[31m 已经安装Mysql，请移除 \e[m"
 else
  ## 安装插件
  yum_install compat-libstdc++-33 
  [ -d $mysql_data_dir ] || mkdir -p $mysql_data_dir
  ## 把文件夹的目录授权给mysql
  chown -R mysql:mysql $mysql_data_dir 

  echo "选择安装mysql的版本"
  select mysql_version in 5.1 5.6
  do
   case $mysql_version in
      5.1)
       [ -f $src_dir/mysql-5.1.72-linux-$ar-glibc23.tar.gz ] || wget -P $src_dir http://mirrors.sohu.com/mysql/MySQL-5.1/mysql-5.1.72-linux-$ar-glibc23.tar.gz
       check_ok
       tar -xvf $src_dir/mysql-5.1.72-linux-$ar-glibc23.tar.gz -C $src_dir
       check_ok
       mv $src_dir/mysql-5.1.72-linux-$ar-glibc23 $install_dir/mysql
       sed -i '/^\[mysqld\]$/a\datadir = /data/mysql' $install_dir/mysql/support-files/my-huge.cnf
       cp $install_dir/mysql/support-files/my-huge.cnf /etc/my.cnf
      break ;
      ;;
      5.6)
      [ -f $src_dir/mysql-5.6.32-linux-glibc2.5-$ar.tar.gz ] || wget -P $src_dir http://mirrors.sohu.com/mysql/MySQL-5.6/mysql-5.6.32-linux-glibc2.5-$ar.tar.gz
      check_ok
      tar -xvf $src_dir/mysql-5.6.32-linux-glibc2.5-$ar.tar.gz -C $src_dir
      check_ok
      mv $src_dir/mysql-5.6.32-linux-glibc2.5-$ar $install_dir/mysql 
      check_ok
      sed -i '/^\[mysqld\]$/a\datadir = /data/mysql' $install_dir/mysql/support-files/my-default.cnf
      cp $install_dir/mysql/support-files/my-default.cnf /etc/my.cnf
      break;
      ;;
      *)
      echo "选择错误"
      ;;
   esac
  done
 
  $install_dir/mysql/scripts/mysql_install_db --user=mysql --datadir=$mysql_data_dir --basedir=$install_dir/mysql
  check_ok
  sed -i 's#^basedir=$#basedir='$install_dir'/mysql#g' $install_dir/mysql/support-files/mysql.server
  check_ok
  sed -i 's#^datadir=$#datadir=/data/mysql#g' $install_dir/mysql/support-files/mysql.server
  cp -f $install_dir/mysql/support-files/mysql.server /etc/init.d/mysqld
  chmod 755 /etc/init.d/mysqld
  chkconfig --add mysqld
  service mysqld start
  check_ok
  $install_dir/mysql/bin/mysqladmin -u root password $passwd ;
  $install_dir/mysql/bin/mysql -u root -p$passwd -e "delete from mysql.user where host != 'localhost'; delete from mysql.user where user=''; update mysql.user set host = '%' ; flush privileges;"
 fi
}
```
### 安装PHP php.sh
```
#!/bin/bash

install_php(){
  for p in libxml2-devel openssl-devel bzip2-devel bzip2-devel freetype freetype-devel libmcrypt-devel libjpeg-turbo libjpeg-turbo-devel libpng-devel
  do 
   yum_install $p
  done
  [ -f $src_dir/php-5.6.25.tar.gz ] || wget -P $src_dir http://101.110.118.75/am1.php.net/distributions/php-5.6.25.tar.gz
  tar -xvf $src_dir/php-5.6.25.tar.gz -C $src_dir
  current_pwd=`pwd`
  cd $src_dir/php-5.6.25
  $src_dir/php-5.6.25/configure --prefix=$install_dir/php --with-config-file-path=$install_dir/php/etc  --with-mysql=$install_dir/mysql --with-libxml-dir --with-gd --with-jpeg-dir --with-png-dir --with-freetype-dir --with-iconv-dir --with-zlib-dir --with-bz2 --with-openssl --with-mcrypt --enable-soap --enable-gd-native-ttf --enable-mbstring --enable-sockets --enable-exif --disable-ipv6
  check_ok
  make && make install
  check_ok
  cd $current_pwd
  cp $src_dir/php-5.6.25/php.ini-production $install_dir/php/etc/php.ini
}
```
### 安装nginx nginx.sh
```
#!/usr/bin/env bash

install_nginx(){
   
   [ -f $src_dir/nginx-1.11.3.tar.gz ] || wget -P $src_dir http://nginx.org/download/nginx-1.11.3.tar.gz
   cPwd=`pwd`
   cd $src_dir
   tar -xvf $src_dir/nginx-1.11.3.tar.gz -C $src_dir
   check_ok
   cd nginx-1.11.3
   ./configure --prefix=$install_dir/nginx --with-http_realip_module --with-http_sub_module --with-http_gzip_static_module --with-http_stub_status_module  --with-pcre
   check_ok
   make && make install
   cd $cPwd
   cp -f conf/nginx /etc/init.d/nginx
   sed -i 's#nginx_home_dir#'$install_dir'/nginx#g' /etc/init.d/nginx
   chmod +x /etc/init.d/nginx
   chkconfig --add nginx
}
```
### 安装脚本 install.sh
```
#!/usr/bin/env bash
. /etc/init.d/functions
. funs.sh
. mysql.sh
. php.sh
. nginx.sh
# 需要使用的参数
install_dir=/usr/local
src_dir=$install_dir/src
## 关闭selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

## 获取当前setlinux状态，转换成为小写
selinux_s=`getenforce | tr [A-Z] [a-z]`

##如果状态为 enforcing ，设设置成 Permissive
if [ "$selinux_s" == "enforcing" ] ; then
 setenforce 0 ;
fi


## 清空iptables防火墙规则
iptables-save > /etc/sysconfig/iptables_`date +%s` 
iptables -F
service iptables save
## 删除 epel-release
if rpm -qa | grep epel-release > /dev/null ; then
 rpm -e epel-release > /dev/null
fi
## 获取Centos的版本，如果是6.xx下载 epel-6.repo ,如果是7.xx 下载 epel-7.repo
epel_file_v=`cat /etc/redhat-release | sed 's/[^0-9.]//g' | cut -d'.' -f1`
epel_file_name=epel-"$epel_file_v".repo
## 删除 epel-6.repo 相关文件
if ls /etc/yum.repos.d/$epel_file_name >/dev/null 2>&1
then
    rm -f /etc/yum.repos.d/$epel_file_name
fi

if [ "7" == "$epel_file_v" ] ; then
 yum_install perl-Module-Install
fi

## 检索是否安装了下列软件，没有就安
yum_install wget


wget -P /etc/yum.repos.d/ http://mirrors.aliyun.com/repo/$epel_file_name
#检查wget下载epel-6.rep文件是否下载成功
check_ok

select_soft
```
