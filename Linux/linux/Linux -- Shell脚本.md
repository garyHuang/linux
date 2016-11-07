# Linux -- Shell脚本 

标签（空格分隔）： linux 

[toc]

---
## shell 后台运行操作
```
alias 查看当前系统设置的别名
alias ld='ls -la'  设置ld的别名命令
unlias ld 取消ld的快捷命令  
---
 sleep 100 //休眠100秒钟,按Ctrl+z 暂停任务
 fg 唤醒 当前暂停的任务
 bg 1 将1好任务 放在后台运行
 jobs  查看当前所有的任务
```
## shell变量
- [ ] 查看系统环境变量 用env，查看所有上下文变量用set
- [ ] 变量取名使用小写，不能是数字开头，不要使用特殊关键字，避免引起歧义
- [ ] 使用\`引用一条命令的结果例如：b=`which vim`; echo $b;打印结果为：/usr/bin/vim
- [ ] unset b 命令可以从当前上下文中删除b这个变量。

## 环境变量
- [ ] root设置全局环境变量，可以修改 /etc/profile。也可以在 /etc/profile.d/ 文件夹下新建一个sh文件，将环境变量设置在这里，umask 也可以写在这里.
- [ ] 普通用户设置环境变量, ~/.bash_profile 和 ~/.bashrc 文件中,用户退出执行的文件： ~/.bash_logout
- [ ] 用户的历史命令文件~/.bash_history 默认保存1000条，通过环境变量 HISTSIZE 来设置。

## 常用命令
### cut命令
- [ ] cut -d: -f 1,4,5  /etc/passwd //指定分隔符为某个字符
 - [x] -d 指定分隔符，如果有特殊符号用'' 包裹 
 - [x] -f 指定显示的数组索引，多个都逗号个隔开,连续的1-4表示
- [ ] cut -d' ' -f 2 1.txt   //指定分隔符为空格字符串
- [ ] cut -c 1-5 /etc/passwd  //指定获取 1-5个字符

### sort命令
- [ ] sort -t' ' -k3 -n -r -u 1.txt
  - [x] -t 指定分隔符 -k指定排序的索引
  - [x] -r 倒序排序，没有-t正序排序
  - [x] -n 根据数字排序，字符串排序 10会在2的前面，加-n后2会在10的前面
  - [x] -u 指定去掉重复记录
- [ ] sort  -un 1.txt
    -[x] 当去重复和数字排序组合在一起的时候，会将所有的字符串行只保留一行，其他的当作重复去掉

### wc 命令
- [ ] wc -l 1.txt
  - [x] -l 统计1.txt文档的行数
- [ ] wc -w 1.txt 
  - [x] -w 统计1.txt中的单词个数
- [ ] wc -m 1.txt
 - [x] -m 统计1.txt中有多少个字符

### uniq 命令
- [ ] sort 1.txt | uniq -c
 - [x] uniq 去掉靠在一起的重复行，如果中间有间断，是不会去掉重复的，所以需要结合sort一起使用
 - [x] -c在第一个字段中显示重复的次数

### tr替换字符串 split文件拆分命令
- [ ] ls *.txt | tr 'a-z' 'A-Z'  
   - [x]  讲当前目录所有txt文件名的小写字母转换成为大写字母
- [ ] echo 'abcdefghijkmnlopqrstuvwxyz' | tr 'abcd' 'ABCD'  
  - [x] 将字符串abcd替换成为ABCD，tr后面的字符串是一一对应的
- [ ] split -b 100 1.txt  new_
  - [x] 将文件以100b的大小进行拆分，拆分的文件以new_开头，默认以x开头
  - [x] -l 制定文件按照行数进行拆分

### perl文件内容替换
- [ ] perl -p -i -e "s/aming/AMING/g" *.txt

### 69 shell 中的连接符（并且和或者）
- [ ] ls 1.txt && ls 2.txt
  - [x] &&左边命令执行成功后才执行右边的命令，当左边执行失败，就不会执行右边命令
- [ ] ls 1.txt || ls 2.txt  
 - [x]  或者符号左右边的执行失败才会执行右边的命令，或者左边执行成功，就不会执行右边的命令
- [ ] ls 1.txt ; ls 2.txt ;
 - [x] 左边执行成功与否，都会执行右边的命令

### grep 使用
- [ ] grep -c root -A 2 -B 2 /etc/passw
 - [x] -c 统计出现了root字符串的行数
 - [x] -n 显示匹配的行索引
 - [x] -A 显示匹配数据的下面两行
 - [x] -B 显示匹配数据的上面两行
 - [x] -C 显示匹配数据的上下各两行
 - [x] -r 匹配文件内容，显示文件名
 - [x] grep -E 命令 和 egrep 命令使用方式一样
 
### sed 命令
- [ ] sed 's/\/sbin\/nologin/\/bin\/bash/'g 1.txt 
  - [x] -i 替换写入源文件
  - [x] s 表示替换语法
  - [x] g表示匹配所有
  - [x] 不需要转义的写法: sed 's#/sbin/nologin#/bin/bash#'g 1.txt
- [ ] sed -n '/\(rr\)\+/'p 1.txt 
  - [x] -n 打印指定的行，不加-n会打印全部文档，把匹配上的行重复打印一次
  - [x] 使用正则表达式需要使用\转移特殊字符，例如： ( ) + - 等 
  - [x] 数字表示 打印指定行, sed -n '1,4'p 1.txt 

- [ ] sed -n '/root/p; /user1/p' 1.txt
 - [x] 表示匹配root和user1的行，这个表示搜索两次，如果一行中同时出现勒两个关键字，结果中救护i出现两行
- [ ]  sed -n -r '/root|user1/p' 1.txt
  - [x] 表示存在root或者user1的行，和上面不同的就是一行中出现两个关键字，结果只出现一次，不会重复
  
### awk 
```
 awk -F ':' 'OFS="#" {print $1 , $4}' 1.txt
   -F 指定分隔符
   OFS 指定输入的链接符号,默认是空格
   {print $1,$4} 指定打印第1段和第4段
 awk -F ':' ' $1~/r*o/ {print $1} ' 1.txt
     $1~/ro/ 表示第一段字符串中有ro的，支持正则表达式
 awk -F ':' ' $3>1000 {print $1} ' 1.txt
   表示第三段大于1000的过滤出来
 awk -F ':' '$3<$4 {print $1} ' 1.tx
  表示第三段小于第四段的过滤出来
 awk -F ':' '$1=="root"' 1.txt
  获取第一段登录root字符串的行 
 awk -F ':' '$7!~/nolog/' 1.txt
  第7段不包含nolog的行
 awk -F ':' 'OFS=":" { if (NR==10) print $1 , $7}' 1.txt
  打印第10行的第一段和第七段
 awk -F ':' 'OFS=":" {print NF}' 1.txt 
   打印行数组拆分的个数 
 awk -F ':' 'OFS=":" {print NR , $NF}' 1.txt 
   NR 表示行数,NF表示数组拆分个数，$NF标识打印最后一个段的值
 awk -F ':' 'OFS=":" {if ($3>$4) $7=$3+$4 ; print $0}' 1.txt  
  判断，当第三段大于第四段的时候，将第七段的值设置第三段加第四段
 awk -F ':' '{(sum=sum+$3)};END {print sum}' 1.txt 
     循环求和，END表示循环结束，打印结果sum
 awk 'ORS=" "{print}' 1.txt 
   其中ORS指定 打印结果的换行符
Shell数组
array=(`cat /etc/issue | awk '{if(NR==1){print $1 , $2}}'`)
echo "${array[0]}"  "${array[1]}"
# 统计Ip访问次数
awk '{a[$1]++}END{for (i in a)print a[i],i}' bak.log
```

### exec
- [ ] find /hksdata/newHKSTomcat/webapps/goodImg/ -type f -name "*.pdf" -exec sudo rm -rf {} \;
 - [ ]  exec 后面一定要加 \;

### linux启动级别
修改/etc/inittab,文件修改启动级别
0级别表示关机
1级别表示单用户模式
2级别表示无NFS服务的3级别
3级别表示命令行启动
4级别为保留级别
5为桌面系统
6级别为重启
命令切换 init 5 ，init 3
