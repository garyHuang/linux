# Linux

标签（空格分隔）： linux-安装

---
## 重置ip地址
## ifconfig -a 查看所有的网卡
## dhclient  获取一个新的IP地址
## 配置ip地址，修改文件 /etc/sysconfig/network-scripts/ifcfg-eth0
```
DEVICE=eth0 
HWADDR=00:0C:29:44:D9:6D 
TYPE=Ethernet
UUID=ad62c4e2-14f9-4f1e-9721-fa5210e211c1
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=static
IPADDR=192.168.1.27
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=192.168.1.1
DNS2=114.114.114.114
```
其中HWADDR的值 和文件 /etc/udev/rules.d/70-persistent-net.rules 中的 ATTR{ADDRESS}==00:0C:29:44:D9:6D的值一样 
## 快捷键
Ctrl + u 清除当前光标之前的内容
Ctrl + k 清除当前光标之后的内容
Ctrl + d 快速退出登录
Ctrl + z 暂停进程
bg 将暂停的进程防砸后台运行
fg 恢复，暂停的任务
jobs 查看当前所有暂停和在后台允许的任务
Ctrl + s 锁定屏幕
Ctrl + q 退出锁屏

## Centos 7 网卡名称叫eno16777736，更改为 eth0
第一步：修改文件
  在文件 /etc/sysconfig/grub 中 GRUB_CMDLINE_LINUX 开头的末尾加上： net.ifnames=0 biosdevname=0
```
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet net.ifnames=0 biosdevname=0"
GRUB_DISABLE_RECOVERY="true"
```
紧接着 执行命令
```
grub2-mkconfig -o /boot/grub2/grub.cfg
```
然后创建文件名： /etc/sysconfig/network-scripts/ifcfg-eth0，设置ip等信息，重启电脑 查看ifconfig
