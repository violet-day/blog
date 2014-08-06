title: Linux下Node生产环境部署
date: 2014-07-24 05:56:42
tags:
-   linux
-   node

categories:
-   linux

---

#	设置时间同步

```bash
yum -y install  vixie-cron crontabs
vi /etc/crontab
0 23 * * *  root /usr/sbin/ntpdate time.scau.edu.cn > /dev/null 2>&1
time.scau.edu.cn 
time.windows.com 
asia.pool.ntp.org
```

#	修改防火墙
```bash
iptables -L -n

vi /etc/sysconfig/iptables
-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT

-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 3000:5000 -j ACCEPT

service iptables start
service iptables stop
service iptables restart
```

#	安装mongo
```bash
wget http://downloads.mongodb.org/linux/mongodb-linux-x86_64-2.2.7.tgz
tar zxf mongodb-linux-x86_64-2.2.7.tgz
mv mongodb-linux-x86_64-2.2.7  /usr/local/mongodb
mkdir /usr/local/mongodb/data
touch /usr/local/mongodb/logs

#写入mongodb的配置文件
vi /etc/mongodb.conf

#mongodb.conf内容如下
dbpath = /usr/local/mongodb/data
logpath = /usr/local/mongodb/logs
auth = true
logappend = true
fork = true
port = 27017
directoryperdb = true
journal = true

#添加至/etc/rc.local
echo "/usr/local/mongodb/bin/mongod -f /etc/mongodb.conf &" >> /etc/rc.local
#将mongo/bin添加至环境变量
ln -s /usr/local/mongodb/bin/* /usr/sbin/
```

#	SVN
```bash
yum install -y subversion
svn help
```

#   Install Node
```bash
yum install gcc openssl-devel gcc-c++ compat-gcc-34 compat-gcc-34-c++
wget http://nodejs.org/dist/v0.10.17/node-v0.10.17.tar.gz
tar zxvf node-v0.10.17.tar.gz
cd node-v0.10.17
./configure --prefix=/usr/local/node
make
make install
#修改npm地址
npm config set registry http://registry.cnpmjs.org
#链接至环境变量，每次安装nmp -g安装的模块都需要执行一次
ln -s /usr/local/node/bin/* /usr/sbin/
```

#	Nginx
## install
```bash
wget http://nginx.org/download/nginx-1.2.6.tar.gz
yum install gcc-c++
yum -y install zlib zlib-devel openssl openssl--devel pcre pcre-devel
tar zxvf nginx-1.2.6.tar.gz
cd nginx-1.2.6.tar.gz
./configure --prefix=/usr/local/nginx
make
make install
ln -s /usr/local/nginx/sbin/nginx /usr/sbin/
```

## 修改conf
```bash
#删除nginx的原来config
rm -f /usr/local/nginx/conf/nginx.conf

#重新写入conf
vi /usr/local/nginx/conf/nginx.conf

#见nginx.conf

#添加vhost文件夹
mkdir /usr/local/nginx/conf/vhost

#添加配置
vi /usr/local/nginx/conf/vhost/#站点名称.conf
```

```conf
  	server {
        listen       80;
        server_name  #站点名称;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location /api/ { 
		proxy_pass  http://127.0.0.1:8080; # node.js app
		proxy_redirect     off;
		proxy_set_header   Host             $host;
		proxy_set_header   X-Real-IP        $remote_addr;
		proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
	}
	location ~* \.(html|jpg|jpeg|gif|png|css|js|json|woff|zip|tgz|gz|swf|ico|txt|xml)$ {
		expires max;
		index index.html; 
		root #路径;
	}
    }
```
## 守护进程
```bash
vi /etc/rc.d/init.d/nginx
# 复制守护进程
```

#	Forever
```bash
forever start -w --watchDirectory /pro/projects/oes/libs/ /pro/projects/oes/libs/app.js
```