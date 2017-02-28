title: statsd源码笔记
date: 2017-01-25 18:37:51
tags:

- nodejs
- statsd
- metric

---

## Key Concepts

* *buckets*  每个stat都有各自的`bucket`，无需事先定义
* *values* 每个stat都会有一个value

## Line Protocol

```
<metricname>:<value>|<type>
```

```bash
echo "foo:1|c" | nc -u -w0 127.0.0.1 8125
```

## Workflow

![](/images/statsd.png)

## Project Struct

```
├── CONTRIBUTING.md
├── Changelog.md
├── Dockerfile
├── LICENSE
├── README.md
├── backends	在得到聚合的metric后，针对不同的后端各自的逻辑
├── bin
├── debian
├── docker-compose.yml
├── docs
├── exampleConfig.js
├── exampleProxyConfig.js
├── examples
├── lib
│   ├── config.js	加载config处理的逻辑
│   ├── helpers.js	help function
│   ├── logger.js	自定义的logger类
│   ├── mgmt_console.js	mangenment consoel的逻辑函数
│   ├── mgmt_server.js	mangement server启动函数
│   ├── process_metrics.js		加工处理metrics
│   ├── process_mgmt.js	对主进程的一些基本设置
│   └── set.js		自己实现的set结构
├── package.json
├── packager
├── proxy.js
├── run_tests.sh
├── servers	metric server，包含tpc和udp两种
├── stats.js	入口文件，包含大部分启动逻辑
├── test
└── utils
```

### stats.js

应用启动的入口文件，主要:

1. 声明一些全局存储变量
2. 声明一些根据config启动server的函数
3. 根据process.args[2]即配置文件路径启动应用

具体的代码的逻辑大致如下:

1. config.configFile(configPath:String)，加载配置
2. process_mgmt.init(config:Object)，设置进程相关的一些基本参数
3. 根据config.prefixStats(String:'statsd')来设置一些内置metric的前缀
4. 初始化counter内bad_lines_seen、packets_received、metrics_received为0
5. 通过config.keyNameSanitize判断是否需要转义metric key
6. 声明`handlePacket(msg:Buffer,rinfo:Object)`，主要作用是接受报文，根据line protocol解析，然后将结果写入一开始申明的全局变量
7. 根据config.servers的配置，从servers目录中加载并启动server，server监听的回调函数就是上一步中声明的handlePacket
8. mgmt_server.start(conifg:Object,on_data_callback:Functioin,on_error_callback)，启动一个tcp的mangement的server
9. 设置percentThreshold，默认[90]
10. 设置flushInterval，默认10s
11. 根据config.backends设置后端的server，默认为`backends/graphite`。*注意：如果我们需要测试，可以设置为`backends/console`*
12. 通过在flushMetrics函数中使用递归调用setTimeout实现flush timer。flushMetrics的大致逻辑内部逻辑为：
	1. 先构造metrics_hash对象
	2. 在backendEvents:EventEmiter中注册flush事件监听，如果config设置了相关的delete配置，则删除相关key，否则置为0或者空数组
	3. 通过process_metrics对构造的metrics_hash对象根据配置做一些加工计算处理，结束之后触发的backendEvents的flush事件
	4. 再次注册setTimeout，进行下一次flush


## lib/process_metrics.js

主要作用是对stats.js中flushMetrics函数中传入的metric_hash做计算加工，并将结果返回出去

具体代码逻辑如下：

1. 声明基本的局部存储变量
2. 遍历counter，将将每秒的结果计算至counter_rates中
3. 遍历timers，每个timer对应的是数组，形如`bar: [200, 198, 199]`这样。因为timer表示的内容比较丰富，所以计算会多一些，timers中的计算结果最终会计算至timer_data中。比如`timers: { bar: [ 100, 200 ] }`，至少会计算出

```js
std: 50,
upper: 200,
lower: 100,
count: 2,
count_ps: 0.2,
sum: 300,
sum_squares: 50000,
mean: 150,
median: 150
```

然后根据`pctThreshold`参数，计算出各个不同分位的指标，像这样

```
count_$pct
mean_$pct
upper_|lower_$pct
sum_$pct
sum_squares_$pct
```

值得注意的是，关于`$pct`的相关的指标计算是将排序后，计算落在`$pct`内的点的计算

做个简单的测试


```bash
echo "foo:1|c\nbar:1|ms\nbar:2|ms\nbar:3|ms\nbar:4|ms\nbar:5|ms\n" | nc -u -w0 127.0.0.1 8125
```

console的backend显示如下:

```json
{ counters:
   { 'statsd.bad_lines_seen': 0,
     'statsd.packets_received': 1,
     'statsd.metrics_received': 6,
     foo: 1 },
  timers: { bar: [ 1, 2, 3, 4, 5 ] },
  gauges: {},
  timer_data:
   { bar:
      { count_45: 2,
        mean_45: 1.5,
        upper_45: 2,
        sum_45: 3,
        sum_squares_45: 5,
        std: 1.4142135623730951,
        upper: 5,
        lower: 1,
        count: 5,
        count_ps: 0.5,
        sum: 15,
        sum_squares: 55,
        mean: 3,
        median: 3 } },
  counter_rates:
   { 'statsd.bad_lines_seen': 0,
     'statsd.packets_received': 0.1,
     'statsd.metrics_received': 0.6,
     foo: 0.1 },
  sets: {},
  pctThreshold: [ 45 ] }
```

## backends/console.js

Flush stats to [graphite](http://graphiteapp.org/)

每个backend中的代码均需要实现`init`方法，在`stats.js`中通过`loadBackend`函数调用。`init`方法中传入的参数有：
	1. startup_time, 启动时间
	2. config, 
	3. backendEvents, 
	4. l，日志对象
`init`函数按照约定都需要同步返回`true`

比如`backends/console.js`，`init`中初始化了`ConsoleBackend`的一个实例，在构造函数中注册了对参数`backendEvents`的`flush`和`flush`事件

## backends/[graphite.js](http://graphiteapp.org/)  

**Metric**

对发送至graphite的指标数据做了封装

```
constructor(key:String,val:Number,ts:Timestamp)
toPickle():String	在graphite使用pickle protocol时使用
toText():String
```

**Stats**

指标数据的集合，提供了`add`方法和`toText`和`toPickle`这两个序列化实现

**init**

1. 设置默认host、port、protocol
2. 设置不同指标的默认的前缀和后缀

**flush_stats**

1. 分别遍历metrics数据中的counters、timer_data、gauges、sets，统一添加至Stats的类实例中，方便后续统一做序列化处理
2. 在添加时stats实例中，会根据统一的前缀和后缀和命名空间做一些指标名称的处理

**post_stats** 

1. 根据config中的host、port建立socket连接
2. 连接成功时，在stats实例中添加统计信息，根据protocol选择序列化方式并写入socket
3. 写入成功之后更新graphiteStats的统计信息，失败也是一样


## 总结：

1. 代码很简单，由于是一个定时的flush的机制，将很多状态直接存储在对象中
2. `backend/*`的解耦了`process metrics`和`flush backend`，`backend/console`在开发时用起来很方便
3. 作为一个`daemon process`，通过`mangent server`提供简单`tcp`接口来反馈当前的统计信息。
4. 指标的量很大，所以在协议设计方面尽可能简单，并且对于不论是`handlePacket`和`flush backend`时，都尽可能使用了`batch`
5. 在metrics的架构中，通常作为缓冲层存在。将大量的point的请求在时间维度做聚合，也是`batch`思想的体现


