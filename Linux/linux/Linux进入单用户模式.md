# Linux进入单用户模式

标签（空格分隔）： linux

[toc]

---
##说明
  如果忘记root密码，无法登录该linux的时候，就需要登录到单用户模式。
 
##进入单用户模式
###第一步：重启linux，在重启界面有个3秒倒计时，在倒计时结束前在键盘上快速按任意键,如下图：
![001.png-8.8kB][1]
###第二步： 在这个界面输入e字母
![002.png-5.7kB][2]
###第三步：讲光标移到第二个选项，输入e字母
![003.png-7.7kB][3]
###第四步：在这个光标位置输入1，后回车
![004.png-5.6kB][4]
###第五步：输入b字母，就会自动进入单用户模式
![005.png-7.6kB][5]
###第六步：在控制台输入 runlevel ，如果打印 1 S 说明单用户模式启动成功，可以使用passwd命令修改root密码，修改完成 reboot重启即可
![最后一个界面][6]


  [1]: http://static.zybuluo.com/Great-Chinese/i47hrjxdu1rwa077oqcv5zqs/001.png
  [2]: http://static.zybuluo.com/Great-Chinese/kzvdcrguhxnoixcmn3npmktu/002.png
  [3]: http://static.zybuluo.com/Great-Chinese/lz8kvonojs3y4s4qrlw269qf/003.png
  [4]: http://static.zybuluo.com/Great-Chinese/u37618y9yjfp8mp8kgqkwfug/004.png
  [5]: http://static.zybuluo.com/Great-Chinese/61p4azlluf2r96m682trzja9/005.png
  [6]: http://static.zybuluo.com/Great-Chinese/m4pyfzck5gzgxb9gfgmh04ys/006.png