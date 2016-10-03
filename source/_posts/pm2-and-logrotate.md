title: pm2 and logrotate
date: 2016-10-03 15:31:23
tags:
	- nodejs
	- pm2
	- linux
	- logrotate
---

nodejs的应用一般都用pm2托管，但是pm2本身的日志处理比较弱，时间久了日志文件会变得很大，需要一些日志切割的策略。

linux一般使用[logrotate](http://www.linuxcommand.org/man_pages/logrotate8.html)来作日志切割，相关介绍见[understanding-logrotate-utility](https://support.rackspace.com/how-to/understanding-logrotate-utility/)

demo配置如下，假设你的nodejs应用使用www-data启动：

```
${loggerRoot}/*.log {
    rotate 7
    daily
    dateext
    dateformat .%Y%m%d
    compress
    missingok
    notifempty
    sharedscripts
    postrotate
        su -l www-data -c 'pm2 reloadLogs'
    endscript

    su www-data www-data
};
```

原理就是在logrotate之后调用[pm2 reloadLogs](http://pm2.keymetrics.io/docs/usage/log-management/#reloading-all-logs)

