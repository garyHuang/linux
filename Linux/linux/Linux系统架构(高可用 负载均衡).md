# Linux系统架构(高可用 负载均衡)

标签（空格分隔）： linux

[toc]

---
## 一、Linux高可用
### 1.1 说明
  高可用，当A机器内存爆满B机器无法ping通A机器，这时候B机器就会将B机器的对应的服开启。
### 1.2 安装
```
yum install -y heartbeat
yum install -y libnet
```
### 1.3 配置文件
拷贝三个文件,HA机器的机器保持配置一致
```
cd /usr/share/doc/heartbeat-3.0.4
cp authkeys ha.cf haresources /etc/ha.d/
cd /etc/ha.d/
```
authkeys 文件内容
```
auth 3
3 md5 Hello!
```
ha.cf文件
```
debugfile /var/log/ha-debug
logfile /var/log/ha.log
logfacility local0
# 监听机器个数
keepalive 2
#死亡时间
deadtime 30
# 警告时间
warntime 10
initdead 60
# 监听端口
udpport 694
# 通过那个网卡和其他ha机器保持通信
bcast eth0
auto_failback on
#当前ha的多个节点，master需要在 /etc/hosts 中配置
node master
node slave
# 路由器ip
ping 192.168.1.1
# 销毁ip的命令
respawn hacluster /usr/lib64/heartbeat/ipfail
```
haresource 文件
```
master 192.168.1.109/24/eth0:1 nginx
# master ha主机的 hostname
# 192.168.1.109 移动的ip地址
# 24 ip地址通道
# eth0 网卡名
# :1 同一张网卡的另一个ip
# nginx 监控的服务名,多个空空格隔开
```
### 1.4 启动服务
第一次启动会很慢，稍等几分钟后，在放访问 192.168.1.109 自动会跳转成功，HA集群的机器都需要启动这个服务。
```
/etc/init.d/heartbeat start
```

## 二、负载均衡 之 NAT 模式
### 2.1 说明
   优点：可以简单快速配置负载均衡
   缺点：单机器有瓶颈 
    附属机器的网关（GATEWAY）必须是主机内网ip
### 2.2 安装软件
```
yum install -y ipvsadm
```
### 2.3 通过脚本配置
脚本：/usr/local/sbin/lvs_nat.sh
```
#!/bin/bash
# director 服务器上开启路由转发功能
echo 1 > /proc/sys/net/ipv4/ip_forward

## 关闭icmp的重定向
echo 0 > /proc/sys/net/ipv4/conf/all/send_redirects
echo 0 > /proc/sys/net/ipv4/conf/default/send_redirects
echo 0 > /proc/sys/net/ipv4/conf/eth0/send_redirects

# director 设置nat防火墙功能
iptables -t nat -F
iptables -t nat -X
iptables -t nat -A POSTROUTING -s 192.168.200.0/24 -j MASQUERADE

# director 设置 ipsadm
IPVSADM='/sbin/ipvsadm'
$IPVSADM -C #清空ipvsadm的规则
# 192.168.145.128 外网
$IPVSADM -A -t 192.168.145.128:80 -s rr
$IPVSADM -a -t 192.168.145.128:80 -r 192.168.1.27:80 -m
$IPVSADM -a -t 192.168.145.128:80 -r 192.168.1.28:80 -m
```

## 三、负载均衡 之 DR 模式
### 3.1 说明
   优点：对服务器没有任何瓶颈，需要通过虚拟ip访问，主机的eth0:0 IP和期附属机器的lo:0的ip需要保持一直
### 3.2 执行脚本
主机执行脚本
```
#!/bin/bash
# director 服务器上开启路由转发功能
echo 1 > /proc/sys/net/ipv4/ip_forward
ipv=/sbin/ipvsadm
vip=192.168.1.100
rs1=192.168.1.27
rs2=192.168.1.28
ifconfig eth0:0 $vip broadcast $vip netmask 255.255.255.255 down
ifconfig eth0:0 $vip broadcast $vip netmask 255.255.255.255 up
route add -host $vip dev eth0:0
$ipv -C
$ipv -A -t $vip:80 -s rr
$ipv -a -t $vip:80 -r $rs1:80 -g -w 1
$ipv -a -t $vip:80 -r $rs2:80 -g -w 1

```
附属机器执行脚本
```
#!/usr/bin/env bash
vip=192.168.1.100
ifconfig lo:0 $vip broadcast $vip netmask 255.255.255.255 down
ifconfig lo:0 $vip broadcast $vip netmask 255.255.255.255 up
route add -host $vip lo:0
echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce
echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
```
配置完成后，可以访问 192.168.1.100 切换两个不同机器的80端口。

## 四、keepalived 高可用，监控web服务器状态
### 4.1 安装 两台机器，一组一丛
```
yum install -y keepalived
```
主机器配置，虚拟ip为：192.168.1.100
```
vrrp_instance VI_1 {
    state MASTER  # 备用服务器上为BACKUP
    interface eth0
    virtual_router_id 51
    priority 100 #备用服务器上为90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.100
    }
}

virtual_server 192.168.1.100 80 {
    delay_loop 6 # 每个6秒查询Realserver状态
    lb_algo rr   # lvs算法
    lb_kind DR   # Direct Route
    persistence_timeout 0 # 同一IP链接60秒内被分配到同一台realserver
    protocol TCP            #用tcp协议检查realserver状态

    real_server 192.168.1.27 80{
        weight 100
        TCP_CHECK {
            connect_timeout 10 #10秒钟无响应超时
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }

    real_server 192.168.1.28 80{
        weight 100
        TCP_CHECK {
            connect_timeout 10 #10秒钟无响应超时
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
}
```
keepalived从机器上
```
vrrp_instance VI_1 {
    state BACKUP  # 备用服务器上为BACKUP
    interface eth0
    virtual_router_id 51
    priority 90 #备用服务器上为90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.100
    }
}

virtual_server 192.168.1.100 80 {
    delay_loop 6 # 每个6秒查询Realserver状态
    lb_algo rr   # lvs算法
    lb_kind DR   # Direct Route
    persistence_timeout 0 # 同一IP链接60秒内被分配到同一台realserver
    protocol TCP            #用tcp协议检查realserver状态

    real_server 192.168.1.27 80{
        weight 100
        TCP_CHECK {
            connect_timeout 10 #10秒钟无响应超时
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }

    real_server 192.168.1.28 80{
        weight 100
        TCP_CHECK {
            connect_timeout 10 #10秒钟无响应超时
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
}
```
启动主和备用机器的keepalived服务
/etc/init.d/keepalived start
查看端口转发规则
ipvsadm -ln 

