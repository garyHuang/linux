# Linux入门

标签（空格分隔）： linux

---
#创建帐户命令
###1.1新建工作组
groupadd hks

###1.2创建用户输入 hks这个工作组
useradd hks -g hks -m -s /bin/bash

###1.3格式化硬盘
mke2fs -t ext4 /dev/xvdb
###1.4将 /dev/xvdb 挂载到 /hksdata 这个目录
mount /dev/xvdb /hksdata/
###1.5查看linux用户列表
    more /etc/passwd
    cat /etc/group

###1.6修改用户名
    usermod -l zhoujian -d /home/zhoujian -m zj

###1.7查看磁盘空间
    df -hl

###1.8查看这个文件夹的大小
    du -hs /hksdata/java

###1.8查找文件个数
    ls -lR | find -type f | wc -l
    查找当前文件夹下，的所有文件，-lR 标识递归， find -type f 标识过滤 查找出来的所有文件，wc -l 表示个数

###1.9查看系统的版本号
    lsb_release -a

#系统文件权限说明：
    chown hks:hks /hksdata 指定文件所在的文件组
    chmod 755 /hkdata      指定文件访问权限
    4为可以读取
    2可以修改
    1可以执行权限
    4+1=5 可以读可移执行权限，其他一次来推
    第一个7表示当前用户权限
    第二个5表示同一个组用户权限
    第三个5标识所有用户权限
文件夹需要可执行的权限，不然ftp打开文件夹看不到文件夹下面的内容。
文件有可执行的权限，就可以运行shell脚本。
文件所有用户都可以修改的时候，执行ls命令的时候 文件名会加一个背景颜色

#linux防火墙设置
###1.1安装防火墙功能
    apt-get install ufw
###1.2启用防火墙
    ufw enable
###1.3开放22端口，如果是云服务器一定要执行这个，否则下次不能使用ssh登录该服务器
    ufw allow 22 
###1.4设置白名单端口，外围可以访问（例如：开放80端口）
    ufw allow 80
###1.5设置白名单IP（设置允许改ip可以访问该机器所有端口）
    ufw allow from 115.29.141.32
###1.6查看当前防火墙的规则
    ufw status 
```
root@hks03:/hksdata/newHKSTomcat# ufw status 
Status: active

To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
22 (v6)                    ALLOW       Anywhere (v6)
```
###1.6删除linux防火墙规则(其中1指规则里面的行号)
    ufw delete 1

### 系统安装目录说明
/boot  标准分区 启动关键文件，固定大小100M就足够了
/swap  交换分区，进行和内存交换，当内存不够用的时候 会使用这块区域，一般为系统内存两倍最多不要超过8个G
/所有的目录放在这里。

