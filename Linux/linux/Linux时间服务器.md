# Linux时间服务器

标签（空格分隔）： linux

[TOC]

---
1、修改文件/etc/sysconfig/clock内容为：  ZONE="Asia/Shanghai"
2、复制文件：
  cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
3、同步系统时间： ntpdate time.windows.com




