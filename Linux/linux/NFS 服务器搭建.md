# NFS 服务器搭建

标签（空格分隔）： linux

[toc]

---
## Ubuntu系统搭建nfs服务器
### 安装 nfs-kernel-server 
apt-get install nfs-kernel-server
### 修改配置文件/etc/exports
```
/hksdata/shell 10.45.234.109(rw,sync,no_subtree_check)
  说明：/hksdata/shell 共享的目录
       10.45.234.109 IP地址
       rw ：可读可写
       sync： 资料同步写入内存和硬盘
       no_root_squash：root用户具有对根目录的完全管理访问权限。
       no_subtree_check：不检查父目录的权限。
```
### 启动rpcbind，和 nfs-kernel-server 服务器，当修改配置文件后，需要重启该服，重启restart
> /etc/init.d/rpcbind start
/etc/init.d/nfs-kernel-server start

### 另外一台机器挂载nfs目录
> mount -t nfs -onolock,nfsvers=3 10.45.236.4:/hksdata/shell /opt/shell/
showmount -e 10.45.236.4

## Centos 搭建nfs服务器
### 修改192.168.1.25机器上的文件 /etc/exports
```
/opt 192.168.1.30(rw,sync,no_subtree_check)
```

### 启动nfs相关服务
```
/etc/init.d/rpcbind start
/etc/init.d/nfs start
```

### 去 192.168.1.30 机器上执行命令,查看可以挂载的目录
> showmount -e 192.168.1.25 

### nfs共享的根目录必须是 777 权限，否则不能创建文件夹，和文件
### 挂载命令：
```
mount -t nfs -onolock,nfsvers=3 192.168.1.25:/opt /opt/
```





