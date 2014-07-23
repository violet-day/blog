title: Linux下Node生产环境部署
date: 2014-07-24 05:56:42
tags:
-   Linux
-   Node

categories:
-   Linux

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

service iptables start
service iptables stop
service iptables restart

-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 3000:5000 -j ACCEPT
```

#	安装mongo
```bash
wget http://downloads.mongodb.org/linux/mongodb-linux-x86_64-2.2.7.tgz
tar zxf mongodb-linux-x86_64-2.2.7.tgz
mv mongodb-linux-x86_64-2.2.7  /usr/local/mongodb
mkdir /usr/local/mongodb/data
touch /usr/local/mongodb/logs
echo "/usr/local/mongodb/bin/mongod --dbpath=/usr/local/mongodb/data --logpath=/usr/local/mongodb/logs --logappend  --auth --port=27017" >> /etc/rc.local
ln -s /usr/local/mongodb/bin/* /usr/sbin/
```

#	SVN
```bash
yum install -y subversion
svn help
```
Install Node
=============
```bash
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
```bash
wget http://nginx.org/download/nginx-1.2.6.tar.gz
yum install gcc-c++
yum -y install zlib zlib-devel openssl openssl--devel pcre pcre-devel
tar zxvf nginx-1.2.6.tar.gz
cd nginx-1.2.6.tar.gz
./configure --prefix=/usr/local/nginx
make
make install
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
vi /etc/rc.d/init.d/nginx                
```
```bash
#设置nginx开启启动，编辑启动文件添加下面内容
#################################################################
#!/bin/bash
# nginx Startup script for the Nginx HTTP Server
# it is v.0.0.2 version.
# chkconfig: - 85 15
# description: Nginx is a high-performance web and proxy server.
# It has a lot of features, but it's not for everyone.
# processname: nginx
# pidfile: /var/run/nginx.pid
# config: /usr/local/nginx/conf/nginx.conf
nginxd=/usr/local/nginx/sbin/nginx
nginx_config=/usr/local/nginx/conf/nginx.conf
nginx_pid=/usr/local/nginx/logs/nginx.pid
RETVAL=0
prog="nginx"
# Source function library.
./etc/rc.d/init.d/functions
# Source networking configuration.
./etc/sysconfig/network
# Check that networking is up.
[ ${NETWORKING} = "no" ] && exit 0
[ -x $nginxd ] || exit 0
# Start nginx daemons functions.
start() {
    if [ -e $nginx_pid ];then
        echo "nginx already running...."
        exit 1
    fi
    echo -n $"Starting $prog: "
    daemon $nginxd -c ${nginx_config}
    RETVAL=$?
    echo
    [ $RETVAL = 0 ] && touch /var/lock/subsys/nginx
    return $RETVAL
}
# Stop nginx daemons functions.
stop() {
    echo -n $"Stopping $prog: "
    killproc $nginxd
    RETVAL=$?
    echo
    [ $RETVAL = 0 ] && rm -f /var/lock/subsys/nginx /usr/local/nginx/logs/nginx.pid
}
reload() {
    echo -n $"Reloading $prog: "
    #kill -HUP `cat ${nginx_pid}`
    killproc $nginxd -HUP
    RETVAL=$?
    echo
}
# See how we were called.
case "$1" in
    start)
    start
    ;;
    stop)
    stop
    ;;
    reload)
    reload
    ;;
    restart)
    stop
    start
    ;;
    status)
    status $prog
    RETVAL=$?
    ;;
    *)
    echo $"Usage: $prog {start|stop|restart|reload|status|help}"
    exit 1
    esac
exit $RETVAL

:wq!           #保存退出
```
```bash
chmod 775  /etc/rc.d/init.d/nginx       #赋予文件执行权限
chkconfig nginx on            #设置开机启动
/etc/rc.d/init.d/nginx restart             #重启
```

#	Forever
```bash
forever start -w --watchDirectory /pro/projects/oes/libs/ /pro/projects/oes/libs/app.js
```    

