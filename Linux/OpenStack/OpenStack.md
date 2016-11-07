# OpenStack

标签（空格分隔）： OpenStack

[TOC]

---
# 安装Centos7
  要求：两块硬盘，系统盘20G，数据盘50G，主机内存 4G，1核，附属机器2G内存1核，具体情况根据物体机情况而定
  并设置主机hostname为 master
  从机器hostname设置为slave01
# 前期准备
### 安装Centos插件centos-release-openstack
  yum install -y centos-release-openstack
### yum升级
 yum upgrade -y
### 安装openstack软件
 yum install -y centos-release-openstack-liberty openstack-selinux
 
# 安装sql服务
  安装mariadb，可以参考安装mysql的安装方式,mariadb是mysql的一个开源分支，mysql被sun公司收购，sun又被oracle收购，开始走商业化路线
# 安装rabbitmq
 yum install -y rabbitmq-server #安装服务
 systemctl enable rabbitmq-server.service #设置开机启动服务器
 systemctl start rabbitmq-server #启动服务器
 rabbitmqctl add_user openstack 123456 #新增openstack账号
 rabbitmqctl set_permissions openstack ".*" ".*" ".*" 给账号设置可配置，可读可写的权限
 
# 安装OpenStack
## 1、创建keystone数据库
create database keystone ;
grant all privileges on keystone.* to 'keystone'@'%' identified by '123456'
## 2、安装相关包
yum install -y openstack-keystone mod_wsgi memcached python-memcached httpd 
## 3、启动 memcached
systemctl enable memcached.service
systemctl start memcached
## 4、编辑keystone配置文件
vim /etc/keystone/keystone.conf
```
[DEFAULT]
a #显示日志
[databases]
connection=mysql://keystone:123456@master/keystone
#数据库链接参数，表示链接的是mysql，账号keystone，密码123456，主机master数据库名keysteone
[memcache]
servers = master:11211
#memcached 的host和端口
[revoke]
driver=sql
```
## 5、导入keystone相关数据
```
su -s /bin/sh -c "keystone-manage db_sync" keystone
```
# 安装httpd
下载 https://pypi.python.org/packages/28/a7/de0dd1f4fae5b2ace921042071ae8563ce47dac475b332e288bc1d773e8d/mod_wsgi-4.5.7.tar.gz
安装编译
```
./configure --with-apxs=/usr/local/httpd/bin/apxs --with-python=/usr/bin/python
```
# 新建文件 conf/extra/wsgi-keystone.conf
```
Listen 5000
Listen 35357
<VirtualHost *:5000>
  WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
  WSGIProcessGroup keystone-public
  WSGIScriptAlias / /usr/bin/keystone-wsgi-public
  WSGIApplicationGroup %{GLOBAL}
  WSGIPassAuthorization On
  <IfVersion >= 2.4>
    ErrorLogFormat "%{cu}t %M"
  </IfVersion>
  ErrorLog /usr/local/httpd/logs/keystone.log
  CustomLog /usr/local/httpd/keystone_access.log combined
  <Directory  /usr/bin>
  <IfVersion >= 2.4>
    Require all granted
  </IfVersion>
  <IfVersion < 2.4>
    Order allow,deny
    Allow from all
  </IfVersion>
 </Directory>
</VirtualHost>
<VirtualHost *:35357>
  WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}

  WSGIProcessGroup keystone-admin
  WSGIScriptAlias / /usr/bin/keystone-wsgi-admin
  WSGIApplicationGroup %{GLOBAL}
  WSGIPassAuthorization On
  <IfVersion >= 2.4>
    ErrorLogFormat "%{cu}t %M"
  </IfVersion>
  ErrorLog /usr/local/httpd/logs/keystone.log
  CustomLog /usr/local/httpd/keystone_access.log combined
  <Directory  /usr/bin>
  <IfVersion >= 2.4>
    Require all granted
  </IfVersion>
  <IfVersion < 2.4>
    Order allow,deny
    Allow from all
  </IfVersion>
 </Directory>

</VirtualHost>
```

## 启动openstackf服务
```
export OS_IDENTITY_API_VERSION=3
export OS_URL=http://master:35357/v3
export OS_TOKEN=123456
openstack service create --name keystone --description "OpenStack Identity" identity
```
## 创建端点
```
openstack endpoint create --region RegionOne identity public http://master:5000/v2.0
openstack endpoint create --region RegionOne identity internal http://master:5000/v2.0
openstack endpoint create --region RegionOne identity admin http://master:5000/v2.0
```
## 创建admin租户
```
openstack project create --domain default --descript "admin Project" admin
```
## 创建admin用户
```
openstack user create --domain default --password-prompt admin
```
## 创建admin角色
```
openstack role create admin
```
## 将admin角色添加到admin项目和用户
```
openstack role add --project admin --user admin admin
```
## 创建demo项目和用户
```
openstack project create --domain default --descript "demo Project" demo
openstack user create --domain default --password-prompt demo
```
## 登录OpenStack用户
```
openstack --os-auth-url http://master:35357/v3 --os-project-domain-id default --os-user-domain-id default --os-project-name admin --os-username admin --os-auth-type password token issue
```
## 创建OpenStack登录脚本 admin-opendr.sh
```
export OS_PROJECT_DOMAIN_ID=default
export OS_USER_DOMAIN_ID=default
export OS_PROJECT_NAME=admin
export OS_TENANT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=123456
export OS_AUTH_URL=http://master:35357/v3
export OS_IDENTITY_API_VERSIONI=3

## 申请令牌
openstack token issue
```


  [1]: http://static.zybuluo.com/Great-Chinese/08dmnkojsplos3luiwhp247p/001.png
  [2]: http://static.zybuluo.com/Great-Chinese/osrsh0vkpgo6au962yo7770l/002.png
  [3]: http://static.zybuluo.com/Great-Chinese/rni4msvhu3obhjmo9621zjv5/003.png