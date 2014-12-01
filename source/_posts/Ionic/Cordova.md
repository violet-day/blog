title: Can't install Cordova plugins from Git on CLI
tags:
  - cordova

categories:
  - cordova

date: 2014-08-09 06:53:03

---

错误信息
```shell
C:\Users\UserName\Desktop\hello>cordova plugin add https://github.com/danwilson/google-analytics-plugin.git
Fetching plugin "https://github.com/danwilson/google-analytics-plugin.git" via git clone
Error: Command failed: fatal: could not create work tree dir 'C:\Users\DAVIDH~1\AppData\Local\Temp\plugman\git\1397683376354'.: No such file or directory
```

解决办法

```shell
mkdir C:\Users\DAVIDH~1\AppData\Local\Temp\plugman\git
```

参考：[Can't install Cordova plugins from Git on CLI](http://stackoverflow.com/questions/23121382/cant-install-cordova-plugins-from-git-on-cli)
