title: 解决SecureCRT中文编码问题
tags:
  - linux
  - SecureCRT

categories:
  - linux

date: 2014-09-10 17:08:59

---

SecureCRT中文乱码、复制粘贴乱码解决办法

SecureCRT的默认配置对中文支持不好。很容易出现中文乱码。
即使显示出来没有乱码，将文本复制粘贴到其他windows程序中也会是乱码，或者从windows复制进SecureCRT会乱码，很不方便。

这个归结起来还是字符编码的问题，需要进行以下简单设置：

1.	首先进入 Option 菜单 >> Session Option 
1.	Terminal >> Emulation，在右边 Terminal下拉菜单中选择"Linux", "ANSI Color"前面打上勾。
1.	Terminal >> Appearance，
	点右边的Font 按钮， 1) 选择新宋体，2) 字符集选"中文 GB2312"
	Charactor 下拉菜单中选择 "Default", 去掉"Use Unicode line drawing character"前面的勾。


保存设置，重启SecureCRT，进入刚才配置的Session，终于可以跟烦人的乱码说BYEBYE了

<!-- more -->