title: Large Request-URI Too Large
date: 2014-07-24 05:59:54
tags:
-   Nginx
-   Request-URI Too Large

categories:
-   Nginx

---


客户端请求头缓冲区大小，如果请求头总长度大于小于128k，则使用此缓冲区，
请求头总长度大于128k时使用large_client_header_buffers设置的缓存区
`client_header_buffer_size 128k;`

large_client_header_buffers 指令参数4为个数，128k为大小，默认是8k。申请4个128k。
`large_client_header_buffers 4 128k;`

当http 的URI太长或者request header过大时会报414 Request URI too large或400 bad request错误。

可能原因:

1.	cookie中写入的值太大造成的，因为header中的其他参数的size一般比较固定，只有cookie可能被写入较大的数据
1.	请求参数太长，比如发布一个文章正文，用urlencode后，使用get方式传到后台。

```
GET http://www.264.cn/ HTTP/1.1
Host: www.264.cn
Connection: keep-alive
Cache-Control: max-age=0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.31 
Accept-Encoding: gzip,deflate,sdch
Accept-Language: zh-CN,zh;q=0.8
Accept-Charset: GBK,utf-8;q=0.7,*;q=0.3
Cookie: bdshare_firstime=1363517175366; 
If-Modified-Since: Mon, 13 May 2013 13:40:02 GMT
```

当请求头过大时，超过large_client_header_buffer时，nginx可能返回"Request URI too large" (414)或者"Bad-request"(400)错误,

如上例HTTP请求头由多行构成，其中"GET http://www.264.cn/ HTTP/1.1"表示Request line

当Request line的长度大于large_client_header_buffer的一个buffer(128k)时，nginx会返回"Request URI too large" (414)错误，对应上面的场景2。

请求投中最长的一行也要小于large_client_header_buffer，当不是Request line的最长行大于一个buffer(128k)时，会返回"Bad-request"(400)错误，对应上面的场景1。

解决办法：这时可以调大上述两个值。

在http节点处添加

client_header_buffer_size 512k;
large_client_header_buffers 4 512k;

