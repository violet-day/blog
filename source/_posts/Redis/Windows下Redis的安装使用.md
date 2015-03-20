title: Windows下Redis的安装使用
tags:
  - redis

categories:
  - redis

date: 2014-12-23 13:22:46

---

官方网站：http://redis.io/

Redis中文：http://www.redis.cn/

官方下载：http://redis.io/download 可以根据需要下载不同版本

部署服务器
```bash
git clone https://github.com/MSOpenTech/redis
cd redis/bin/relase
##启动redis服务器，一旦关闭cmd，服务停止
redis-server.exe redis.windows.conf
```

启动客户端

```bash
redis-cli.exe -h 127.0.0.1 -p 6379
```


<!-- more -->