title: 在centos下安装配置基于gitosis的git服务
tags:
  - git
  - gitosis

categories:
  - git

date: 2014-12-03 12:30:04

---

###	[服务器]安装 openssh服务器与客户端工具
```bash
sudo yum install openssh-server openssh-client -y

#安装python的setup tool
sudo yum install python-setuptools -y

#安装git服务器
sudo yum install git-core -y

#获取并安装gitosis
cd /tmp
git clone https://github.com/res0nat0r/gitosis.git
cd gitosis
sudo python setup.py install
```

### [客户端]上传公钥
```bash
#创建个人公钥和私钥（用于后面的git服务的管理员管理）
ssh-keygen -t rsa

# 通过ssh使用git用户把本机的公钥发送到/homt/git目录下
scp /home/simen/.ssh/id_rsa.pub git@192.168.88.88:/home/git
```

### [客户端]配置git服务器
```bash
# 增加git用户 不要给git设置密码 否则只要知道git密码可以绕过gitosis的验证
sudo useradd -m -s /bin/bash -d /home/git git  
sudo passwd git

#切换到git用户 注意中间有一个 -
su - git
#初始化gitosis
gitosis-init < /home/git/id_rsa.pub
#设置权限让gitosis-admin仓库可clone
sudo chmod 755 /home/git/repositories/gitosis-admin.git/hooks/post-update
```

### [客户端]clone gitosis-admin
```bash
#写相对路径即可
git clone ssh://git@221.224.215.171:4272/gitosis-admin.git
```

### 管理gitosis配置

gitosis.conf 对应的内容
```
[gitosis]

#显示日志
#loglevel = DEBUG

[group gitosis-admin]
writable = gitosis-admin
#多个用户用空格间隔
members = root@localhost.localdomain  

``` 

### 新增仓库
```bash
su git
cd /home/git/repositories
git init xxx --bare
```

<!-- more -->