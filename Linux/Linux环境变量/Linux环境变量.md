# Linux环境变量

标签（空格分隔）： Linux环境变量
##1.1 取别名  
```
alias ls='ls --color=auto'
alias melody='ls -alt/var/' (只在当前窗口生效，其它窗口不生效）
```

## 1.2 如何在多个窗口生效
> 把别名放入 vi ~/.bashrc 此文件下，就可以在多个窗口中生效
```
vi ~/.bashrc 
```

##1.2 配置变量
###1.2.1 重新定义PATH
```
PATH=$PATH:/tmp/
```

###1.2.2 如何在多个窗口生效
把重新定义的PATH加入 vi /etc/profile 此配置文件中，然后重启系统即可在多个窗口中生效
```
vi /etc/profile
```

###1.2.3 即时生效
```
source /etc/profile
```


##1.3 相关命令
> 
which ls 查询文件所在路径
alias 查询所有别名
echo $PATH
cp
ins
mv
chmod +x /usr/bin/install.log


