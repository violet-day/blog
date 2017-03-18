title: 如何优雅关闭/重启server
date: 2017-03-17 17:35:49
tags:

---

## Signal

> 软中断信号（signal，又简称为信号）用来通知进程发生了异步事件。进程之间可以互相通过系统调用kill发送软中断信号。内核也可以因为内部事件而给进程发送信号，通知进程发生了某个事件。注意，信号只是用来通知某进程发生了什么事件，并不给该进程传递任何数据。 

在[UnixSignal](https://en.wikipedia.org/wiki/Unix_signal)中提到了很多Signal，这里主要关注以下两个:

* `SIGTERM` 用来通知进程停止。`Kill` 命令默认就是发送该信号。这是一种相对*礼貌*的方式，进程可以对改信号做出处理，比如释放正在使用的资源之后再退出，或者直接忽略。
* `SIGKILL` 用来让进程立即停止。`Kill -s SIGKILL $pid`。和`SIGTERM`不同，该信号不能被进程监听或者忽略。

## Graceful Shutdown

生产环境中我们的应用一般通过守护进程来托管。下面看下不同的守护进程是否如何使用上面的信号的

* [supervisord](http://supervisord.org/configuration.html) 停止服务时，默认使用`SIGTERM`，并且等待10s，如果进程还在运行，则使用`SIGKILL`。参考`stopsignal`和`stopwaitsecs`配置
* [docker](https://docs.docker.com/engine/reference/commandline/stop/) 容器中的主进程会接受的`SIGTERM`，默认10s之后，发送`SIGKILL`
* [pm2](https://github.com/Unitech/pm2/blob/master/lib/God/Methods.js#L216) 发送`SIGINT`给相关子进程，没有和上面两个做类似的动作，不过[官方文档](http://pm2.keymetrics.io/docs/usage/signals-clean-restart)中提到了应用如何自己实现优雅关闭

上面提到的都是守护进程方面如何优雅的停止进程。但是仍然有一个问题，以web server为例，在`SIGTERM`超时到`SIGKILL`过程中，还是会有请求不断的过来。那么在集群环境下，如何做到重启一个节点对生产无感知?

以zk作为服务发现为例，可以用以下步骤:

1. zk通知consumer节点，有一个server即将关闭。即将关闭的消息需要同步给很多consumer，如果确保所有consumer节点均收到消息之后修改本地负载均衡池的话再执行server关闭的话，保证一致性带来的成本会比较高，可以简单设置一个超时时间。
2. 在同步下线消息之后，consumer应该确保不再向即将下线的server发起新的请求。
3. 一般server端都会有服务端超时时间，即处理一个请求，超过一定时间，即抛出Timeout异常。上文中提到的`SIGTERM`到`SIGKILL`超时10s对于一个请求来说已经绰绰有余。server接受到关闭的信号之后，尽可能处理好进程中正在处理的请求，到了超时时间之后再执行关闭动作

## Referecne

* [Unix signal](https://en.wikipedia.org/wiki/Unix_signal)
* [Linux 信号signal处理机制](http://www.cnblogs.com/taobataoma/archive/2007/08/30/875743.html)
* [Graceful shutdown in node.js](http://joseoncode.com/2014/07/21/graceful-shutdown-in-node-dot-js/)

