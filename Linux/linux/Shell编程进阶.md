# Shell编程进阶

标签（空格分隔）： linux

[toc]

---
## if 判断语句
### 判断是否存在某个账号
```
if awk -F ':' '{print $1}' /etc/passwd | grep -E '^user3$' >/dev/null ; then echo "user 3 exists" ; fi

if [ -n "$a" ] ; then echo '$a 不为空' ; fi ;
if [ $a -lt 10 ] ; then "$a < 10" ; fi ; 
if [ $a -gt 5 ] ; then "$a > 5" ; fi ;
if [ $a -eq 5 ] ; then "$a = 5" ; fi ;
```
## case 使用
```
#!/bin/bash

read -p "请输入一个数字:" n 
rest=`echo $n | sed 's/[0-9]//g'` ;
if [ -n "$rest" ]; then echo "input is not a number ;"; exit 1 ; fi ;
m=$[$n%2];
case $m in
     1)
        echo "这是一个基数。"
        ;;
     0)
        echo "这是一个偶数."
       ;;
     *)
       echo "这不是一个整数" ;
esac
```
## for 使用
```
# 序号 一个目录下的文件夹
for a in `ls /usr/local/sbin/test/` ; do echo $a ; done ; 
# 简单循环10次
for i in {1..10} ; do echo $i ; done ;
# 文件空格，和换行昨晚分隔符
for line in `cat 3.sh` ; do echo $line ; done ;
```
## shell函数
```
#!/bin/bash
mysum() { 
  local sum=$[$1+$2] ; #加上local 只能在函数内部访问。不加则可以在函数外部访问该变量
  echo $sum;
}
mysum 1 3;
```
## shell 数组
```
#定义
a=(1 2 3 4)
#循环输出
for i in ${a[*]} ; do echo $i ; done ;
#更改索引的值
a[0]=7
#获取数组长度
echo ${#a[@]}
#取消某个索引的值
unset a[4]
#去第2开始，往后取4个数
echo ${a[*]:1:4}
```