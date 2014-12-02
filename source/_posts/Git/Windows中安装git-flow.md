title: Windows中安装git-flow

tags:

  - git
  - gitflow

categories:
  - git

date: 2014-12-01 21:11:32

---

[git-flow 备忘清单](http://danielkummer.github.io/git-flow-cheatsheet/index.zh_CN.html)

生命不息, 折腾不休. 本文记录一下在windows平台安装git-flow的一些注意事项.

git 是什么这里就不作介绍了. 但是不得不说git对于程序员来说是一把利剑. 相对于我们通常在windows上选择svn这种可视化的集中式版本管理系统而言, git是一个非常灵活的分布式管理系统. 撇开其他的优点不说, 它的离线操作特性让我非常喜欢. 还有非常重要的分支管理特性, 相对于一直在开发环境使用的svn而言, 这简直就是救世主.

由于血统的原因, git虽然非常强大, 灵活, 但也并不是那么复杂!

言归正传! git相关的文章有时间再慢慢的发. 今天只记录下windows平台下的一些东西. 早期windows平台对git的支持性并不好, 更别谈git-flow这种后来的新特性了.

首先, 可以在 http://msysgit.github.io 下载到最新的git bash(一个针对windows平台而封装的git命令行管理工具), 其实它还自带了超级难用的一个可视化操作工具git ui.

安装完成msysgit, 即可使用如下命令将gitflow克隆到本地:

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

注意这里, 这里以D盘为例, 首先是找到gitflow目录下contrib子目录中的msysgit-install.cmd表示我们要执行安装命令, 如果单独执行这个命令不带指定安装目录则会报: 找不到msysgit安装目录的错误, 并且会要求在命令行提供一个目录名称(安装目录). 因此在执行安装操作的后面需要指定一个安装目录.

那么安装完成之后到git bash中执行git flow会列出git-flow提供的命令. 此时, 大功告成.

题外话: git虽是基于命令行操作的. 但是也有可视化的工具如source tree就是一款非常好的工具, 而且source tree自身就提供了git-flow的功能. 可以根据自己的喜好而选择合适的方式使用git.

[source tree](http://www.sourcetreeapp.com/)下载


<!-- more -->