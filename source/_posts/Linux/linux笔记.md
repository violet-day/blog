title: linux笔记
tags:
  - linux
categories:
  - linux
date: 2014-08-06 17:51:13

---

## 重启命令
1.	reboot
1.	shutdown -r now 立刻重启(root用户使用)
1.	shutdown -r 10 过10分钟自动重启(root用户使用) 
1.	shutdown -r 20:35 在时间为20:35时候重启(root用户使用)

如果是通过shutdown命令设置重启的话，可以用shutdown -c命令取消重启

## 关机命令
1.	halt   立刻关机
1.	poweroff  立刻关机
1.	shutdown -h now 立刻关机(root用户使用)
1.	shutdown -h 10 10分钟后自动关机

如果是通过shutdown命令设置关机的话，可以用shutdown -c命令取消重启

## 创建用户（useradd）：

在Linux下创建用户和删除用户，必须在root用户下，如果你当前不是用根用户登录，你可以打开终端，输入"su root"命令，再输入根口令，就可以进入root用户模式

1.	用useradd命令创建用户创建用户：
	语法： useradd [所要创建的用户名] ，回车
1.	用passwd命令为该用户创建密码：
	语法： passwd [用户名]  ，回车
1.	输入密码：一般密码至少要有六个字符，这里输入的密码是看不见的，所以看到屏幕没显示，不要以为是输入失败
1.	重新输一次密码：
这里我创建一个名为aillo的用户，如下所示：

##	删除用户（userdel）

语法：userdel [-r] [要删除的用户的名称]
例如：[root@localhost ~]userdel -r aillo