# Linux基本语法

标签（空格分隔）： linux

[toc]

---
## 基本操作
- [ ] mkdir  /tmp/1/2/3/4/5   //文件5文件夹，如果父目录不存在，则会创建失败
- [ ] mkdir -pv /tmp/1/2/3/4/5  //创建5文件夹，如果父目录不存在，也会创建
- [ ] rmdir /tmp/1/2/3/4/5   //删除非空目录 5，如果不为空，提示创建失败
- [ ] rmdir -pv /tmp/1/2/3/4/5  //递归删除，/tmp /tmp/1 /tmp/1/2 /tmp/1/2/3 /tmp/1/2/3/4 /tmp/1/2/3/4/5， 首先删除最里层的文件夹
- [ ] ls -la /tmp/1 //查看这个目录的详情
- [ ] echo '111111' >> 1.txt //在文件1.txt中追加内容 111111
- [ ] echo '11111'  > 1.txt //把文件1.txt的内容都删除，重新写入 11111
- [ ] su - hks //切换账号，加减号：centos显示改账号最后一次登录时间， ubuntu 不显示
- [ ] chmod a+g 1 //当目录的权限是777的时候，任何一个普通账号是可以修改和删除这个目录下的任何文件的，当设置这个权限后，用户自己修改和删除当前账号有权限修改的账号，这时候 目录的权限是 drwxrwxrwt 
- [ ] chattr +a 1.txt 文件不可修改，只能使用 echo追加(root才能执行)
- [ ] chattr +i 1.txt 文件不可修改，不可移动，不可删除(root才能执行)
- [ ] lsattr 1.txt 查看文件的隐藏属性
- [ ] ln -s ~/datas/hf.txt ~/ln/1.txt 创建软链接，目录可以随意指定，可以跨磁盘
- [ ] ln ~/datas/hf.txt ~/ln/hf.txt 创建硬链接，两个文件保持一致，拥有同一个inode,当 datas/hf.txt修改，ln/hf.txt 也会一起修改，当删除datas/hf.txt时， ln/hf.txt 的硬链接就会自动删除

### 密码文件 /etc/shadow
```
 hks:$6$uh510KJ5$ovpKd0g0PzrLeArX94uLqrGgxzdB/nT1sU0pRPkYJNxDfgxeKw6eerS9ksLXBkCGjls5wZOzBnvZSCvULlxmt1:16960:0:99999:7:::
分段解说： 
第一段：hks表示是哪个账号的密码
第二段：$6$uh510KJ5$ovpKd0g0PzrLeArX94uLqrGgxzdB/nT1sU0pRPkYJNxDfgxeKw6eerS9ksLXBkCGjls5wZOzBnvZSCvULlxmt1 表示当前账号的密码，当然是已经加密处理过的
第三段：16960 表示从1970年1月1日离最后一次修改密码的天数
第四段：0 要过多少天后才可以修改密码，0表示随时可以修改密码
第五段：0 密码要过多少天会过期，默认是99999
第六段：7 表示密码到期前多少天登录提醒
第七段： 表示到期多少天后账号会锁定不能登录
第八段： 表示账号有效期天数，到期账号自动锁定
第九段： 保留字段，暂无意义
```
### Linux分组
#### 新增组命令：
groupadd -g 512 grp1 //注明： -g 512 是给组指定组id，一般情况可以省略
#### 查看组文件：
cat /etc/group
#### 文件解释：
```
grp1:x:1007:
第一段：组名
第二段：密码
第三段：组id
```
### 删除组：
groupdel grp2
### Liunx用户
#### 新增用户命令，指定组
useradd -u 512 -g grp1 -d /home/user1 -s /bin/bash user1
#### 删除账号
userdel user1 //删除账号，不删除家目录
userdel -r user1 //删除账号，删除家目录
#### usermod 修改用户属性
usermod -g grp2 -G grp1 user1 //修改用户的组，并新增一个扩展组,-G参数说明多个扩展组用逗号隔开，
例如： usermod -g grp2 -G grp1,grp3 user1 
#### 锁定一个用户不让登录
usermod -L user1 //这是查看密码文件，会在账号密码前面多一个!号，如果账号没有设置密码，密码出会有两个!
usermod -U user1 //解锁帐户
### 修改用户密码
- [ ] passwd user1 直接设置密码
echo "huangfei" |  passwd --stdin user1 //一条命令，直接设置密码
(ubuntu:echo user:pass | /usr/sbin/chpasswd)
### su使用
su user1 //切换账号，使用当前登录账号的环境变量，切勿这样使用
su - user1 //切换账号，使用user1用户的环境变量
su - -c 'ls /home/user1' user1 使用 su 账号执行一条命令
su - -c 'mkdir -pv /tmp/111/222' user1 //使用 user1账号新增创建目录

### umask
系统默认 umask 的值为 022，查看账号当前的umask值，直接输入umask，设置为002 则为umask 002，
这样设置每次重新登录后，就会默认为022，需要永久生效。
 修改 家目录中的文件：.bash_profile，或者 .profile 中加入一行 umask 002，即可

### 查看磁盘，文件夹大小

>* df -h                //查看磁盘使用空间
>* du -sh /opt/redis/   //查看目录大小,当一个文件小鱼4k时这条命令获取文件大小也是4k，因为linux磁盘格式化的时候，是按块格式化的，每一个块的大小默认是4K，如果文件小于4K，文件占用磁盘的空间就是4K。
>* du -sb /opt/redis/  //可以查看那文件真实的大小
>* df -T -h //查看磁盘分区的格式

### ls命令详解
-i 查看inode
-ld 查看明细
-a 查看所有文件
-d .* 查看所有文件名.开头的文件
### 磁盘格式化、分区、挂载、卸载
##### 查看新加载的磁盘
fdisk -l

##### 磁盘格式化，快速格式化：
 mkfs.ext3 /dev/sdb
 
##### mke2fs格式化

- [ ] mke2fs -m 1 -L gary -t ext4 /dev/sdb
- [ ] -m 保留区域的百分比
- [ ] -L 给磁盘加标签
- [ ] -t 指定磁盘格式化类型

##### 挂载磁盘,卸载
mount /dev/sdb /home/user1/data
其中/home/user1/data 目录需要手工创建
umount /home/user1/data

#### 开机自动挂载磁盘
查看磁盘UUID，blkid 查看磁盘UUID
修改文件/etc/fstab
```
UUID=982a85a1-a7f4-4ea1-81c1-d02967d94f28 /hksdata ext4 default 0 0
```

##### 查看文件特殊命令
- [ ] cat -A /etc/passwd 文件一行结尾加上$符号
- [ ] tac -A /etc/passwd 从文件最后一行显示
- [ ] more /etc/passwd   按回车键，一屏一屏的往下翻
- [ ] less /etc/passwd   支持上下翻页pageup，和puagedown，回车键一行一行的显示
- [ ] tail -3 /etc/passwd 查看文档前面3行

##### vim 使用说明
- [ ] vim +10 1.txt  //将光标定位在文件第10行第一列
- [ ] vim 1.txt 光标移动说明： page up 向上翻页，page down 向下翻页，Shift+4($) 定位在一行的末尾 Shift+6(^) 定位一行最开始部位空格的地方，0(又小括号),定位在一行最开始的地方***注明：这些操作必需在非编辑状态下操作,如果当前是编辑状态下，按ESC可退出编辑状态，按I字母键，进入编辑状态***
- [ ] vim下的复制粘贴：两下yy是复制光标当前的行，按p是粘贴，两下D是删除光标当前的行，4+两下dd标识删除删除光标以下的4行，按4+yy，也是复杂四行

###### vim搜索和替换：
- [ ] 搜索语法:/string,按小写字母n，想下搜索下一个关键字,按大写字母N，搜索上一个关键字
- [ ] 替换语法： 1,20s/string/newString/g 第一行到20行内的string全部替换城newString。如果要替换整个文档语法有两种如下：%s/string/newString/g 和  1,$s/string/newString/g
 
##### bzip2 gzip xz 压缩解压
- [ ] gzip 1.txt //压缩1.txt,原文件直接修改为1.txt.gz
- [ ] gzip 1.txt > 1.txt.gz //压缩1.txt，源文件保留，新生成一个1.txt.gz
- [ ] gcat 1.txt.gz 查看压缩文件
- [ ] gzip -1 1.txt > 1.txt.gz //最快，压缩比例1%的比例压缩，默认基本是6%
- [ ] bzip2 --fast  -v 1.txt //最快的方式压缩
- [ ] bzcat 1.txt.bz2 //查看压缩的bz2文件
- [ ] xz 1.txt //压缩后更改文件为 1.txt.xz
- [ ] xzcat 1.txt.xz //查看xz压缩文件
- [ ] xz -d 1.txt.xz //解压1.txt.xz

##### tar 打包压缩工具
- [ ] tar -cvf a.tar.gz a/ b/ b.txt 压缩多个目录或者文件，
   - [x] c 创建 v 可视化，看见打包的过程,f递归
- [ ] tar -xvf a.tar.gz //解压到当前目录
- [ ] tar -C /tmp/123456/ -xvf a.tar.gz //加上-C 指定打包的目录
- [ ] tar -zcvf a.tar.gz a/ b/ b.txt //打包并用gzip方式压缩文件
- [ ] tar -zxvf a.tar.gz //加压并用gzip方式解压
   - [x] tar -tf a.tar.xz
   - [x] tar -zxvf a.tar.gz -C 333  解压tar.xz文件到333目录下
- [ ] tar -jcvf a.tar.bz2 a/ //打包压缩成bz2格式文件
   - [x] tar -tf a.tar.bz2
   - [x] tar -jxvf a.tar.bz2 -C 333  解压tar.xz文件到333目录下
- [ ] tar -Jcvf a.tar.xz a/
   - [x] tar -tJf a.tar.xz
   - [x] tar -Jxvf a.tar.xz -C 333  解压tar.xz文件到333目录下

##### mkpasswd 随机密码
- [ ] mkpasswd -l 22 -s 14 -C 2 -d 3 -c 3
    - [x] -l 指密码长度
    - [x] -s 特殊字符长度
    - [x] -d 数字个数
    - [x] -c 小写字母个数
    - [x] -C 大写字母个数

##### rpm安装和卸载
- [ ] rpm -ivh --nodeps yp-tools-*.*.*-3e2.3.2.rpm //安装软件
    - [x] --nodeps 不需要关注是否依赖 
- [ ] rpm -e --nodeps yp-tools //卸载软件
  - [x] --nodeps 不需要关注是否依赖
- [ ] rpm -qa | grep 'vim'  //查找vim安装的包
- [ ] rpm -ql coreutils-8.22-15.el7_2.1.x86_64 //查找一个包安装了哪些文件
- [ ] rpm -qf /usr/bin/ls //查找这个文件是由哪个包安装来的


##### yum （rpm管理工具）
- [ ] yum -y install vim-X11  安装 vim-X11软件
- [ ] yum list | grep 'vim' 搜索vim相关的软件包
- [ ] yum remove vim-X11 卸载vim-X11软件
- [ ] yum grouplist
- [ ] yum deplist vim-X11 查看依赖包
- [ ] yum install vim-X11 --downloadonly --downloaddir=/tmp 下载指定包到/tmp目录
- [ ] rpm -qf `which setenforce`
#### yum 安装yum-download工具
- [ ] yum install -y yum-downloadonly
- [ ] yum install -y yum-utils
#### yum 工具修复 
```
rpm --rebuilddb
yum clean all
```
