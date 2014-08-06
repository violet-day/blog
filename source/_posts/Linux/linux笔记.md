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