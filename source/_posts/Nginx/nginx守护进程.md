title: nginx守护进程
date: 2014-08-05 23:37:54
tags:
-	nginx
-	守护进程

categories:
-	nginx

---

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