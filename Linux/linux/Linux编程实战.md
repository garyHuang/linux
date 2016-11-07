# Linux编程实战

标签（空格分隔）： linux

[toc]

---
## expect 自动登录脚本
### 安装expect
```
yum install -y expect
```
## 自动登录到另外一台机器
新建文件1.expect
```
#! /usr/bin/env expect
set host "192.168.11.102"
set passwd "123456"
spawn ssh  root@$host
expect {
"yes/no" { send "yes\r"; exp_continue}
"assword:" { send "$passwd\r" }
}
interact
```
### 自动同步脚本
```
#!/usr/bin/env expect
set passwd "123456"
spawn rsync -av root@192.168.11.18:/tmp/12.txt /tmp/
expect {
"yes/no" { send "yes\r"}
"password:" { send "$passwd\r" }
}
expect eof
```
### 传递参数
执行语法 ./2.expect 192.168.1.123 123456 "ls -alF /tmp"
```
#!/usr/bin/env expect
set password [lindex $argv 1]
set host [lindex $argv 0]
set cm [lindex $argv 2]
spawn ssh root@$host
expect {
 "yes/no" { send "yes\r" }
 "assword:" { send "$password\r" }
}
expect "]*"
send "$cm \r" 
expect "]*"
send "exit\r"
```

