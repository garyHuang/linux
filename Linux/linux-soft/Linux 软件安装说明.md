# Linux 软件安装说明

标签（空格分隔）： linux-soft

---
1、ubuntu 乌班图
```
apt-cache search libmcrypt #搜索软件
apt-get install -y libmcrypt-dev #安装软件
```
2、Centos 
```
yum search libmcrypt  #搜索软件
yum install -y libmcrypt-devel #安装软件

部分软件无法安装需要安装一个插件
yum install -y epel-release

```