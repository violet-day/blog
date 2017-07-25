title: thrift-timeout
date: 2017-01-07 13:58:33
tags:
  - nodejs
  - thrift
  - socket
---

## socket timeout test

### net.socket的超时

### http.client如何实现调用超时

### core module class diagram
* stream
* socket
* http agent
* http common
* http incoming
* http outgoing
* http server

* OutgoingMessage.prototype.write
	* 检查是否finshed，如果是抛错；进入下一次tick 
	* 
* OutgoingMessage.prototype._send，相对底层的函数，直接写入socket或者写入buffer
* OutgoingMessage.prototype._writeRaw
	* socket可以写入
		* outputLengh>0，this._flushOutput
		* 直接写入socket
* OutgoingMessage.prototype._flushOutput
	* 读取ouput数据
	* cork->批量socket.write->uncork
	* 重置output
		 



## Reference

* [一个简单的python socket编程](http://openexperience.iteye.com/blog/145701)
* [socket.setTimeout](https://nodejs.org/dist/latest-v6.x/docs/api/net.html#net_socket_settimeout_timeout_callback)
* [thriftpy/transport/socket.py](https://github.com/eleme/thriftpy/blob/develop/thriftpy/transport/socket.py)
* [Backpressure](https://github.com/ReactiveX/RxJava/wiki/Backpressure)
* [如何形象的描述反应式编程中的背压(Backpressure)机制？](https://www.zhihu.com/question/49618581)

