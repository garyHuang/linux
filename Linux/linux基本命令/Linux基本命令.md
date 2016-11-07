# Linux基本命令

标签（空格分隔）：Linux基本命令

---

##1.1CD命令

> cd 进入当前用户的家目录
whoami 查看当前用户的名字
id  查看uid/gid/组
echo $HOME  查看用户的家目录
pwd  查看当前目录的完整路径
cd - 进入上一次的目录下
cd ~  当前用户的家目录
cd .  当前目录
cd .. 上级目录

##1.2 LS命令
> ls 查看目录下面的文件
ls -d 查看目录本身 
ls -ld
ls -l 查看文件，显示详细信息
ls -a 列出所有文件，包括隐藏的文件  $$$ (touch.1.txt  创建隐藏文件） 
ls -t 按时间顺序排序，最新的在最上面
ls -lt
ls -i 查看inode号，文件，目录
ls -li
