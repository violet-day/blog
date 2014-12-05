title: Windows中安装git-flow

tags:

  - git
  - gitflow

categories:
  - git

date: 2014-12-01 21:11:32

---

[git-flow 备忘清单](http://danielkummer.github.io/git-flow-cheatsheet/index.zh_CN.html)

http://msysgit.github.io 下载到最新的git bash(一个针对windows平台而封装的git命令行管理工具), 其实它还自带了超级难用的一个可视化操作工具git ui.

安装完成git, 即可使用如下命令将gitflow克隆到本地:

```bash
$ git clone --recursive git://github.com/nvie/gitflow.git
```

其他方式具体可以参考gitflow官方介绍: https://github.com/nvie/gitflow.

然后分别下载下面两个文件:

*	[util-linux-ng-2.14.1-bin](http://sourceforge.net/projects/gnuwin32/files/util-linux/2.14.1/util-linux-ng-2.14.1-bin.zip/download)

*	[util-linux-ng-2.14.1-dep](http://sourceforge.net/projects/gnuwin32/files/util-linux/2.14.1/util-linux-ng-2.14.1-dep.zip/download)

下载完成后分别将两个压缩包解压文件下bin目录中的getopt.exe和libintl3.dll文件存放到git安装目录下的bin目录中.

最后使用windows系统自带的命令行工具, 进入到我们已经克隆好的gitflow目录中执行以下命令:

```bash
D:\gitflow contrib>msysgit-install.cmd "C:\Program Files\Git"
```

注意这里, 这里以C盘为例, 首先是找到gitflow目录下contrib子目录中的msysgit-install.cmd表示我们要执行安装命令, 如果单独执行这个命令不带指定安装目录则会报: 找不到msysgit安装目录的错误, 并且会要求在命令行提供一个目录名称(安装目录). 因此在执行安装操作的后面需要指定一个安装目录.

那么安装完成之后到git bash中执行git flow会列出git-flow提供的命令. 此时, 大功告成.

题外话: git虽是基于命令行操作的. 但是也有可视化的工具如source tree就是一款非常好的工具, 而且source tree自身就提供了git-flow的功能. 可以根据自己的喜好而选择合适的方式使用git.

[source tree](http://www.sourcetreeapp.com/)下载


<!-- more -->