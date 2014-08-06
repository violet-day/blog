title: nginx的配置文件
date: 2014-08-05 23:34:30
tags:
-	nginx
categories:
-	nginx

---

```conf
#user  nobody;
worker_processes  4;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;
worker_rlimit_nofile  65535;   #nginx能打开文件的最大句柄数,最好与ulimit -n的值保持一致,使用ulimit -SHn 65535 设置

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    include       vhost/*.conf;
    default_type  application/octet-stream;
    sendfile on; 
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;


    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    gzip on; #启用gzip功能  
    gzip_static on; #检查有没有静态文件的.gz版本，没有自行gzip  
    gzip_disable "MSIE [1-6]\.(?!.*SV1)"; #IE6以下禁用gzip  
    gzip_comp_level 7; #级别越高，CPU占用越多，压缩的越小  
    gzip_min_length 1024; #到达1K的文件才gzip

    #解决414 Request URL Too Large
    client_header_buffer_size 512k;
    large_client_header_buffers 4 512k;
}

```