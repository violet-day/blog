title: RabbitMQ中的Back-off和retry
date: 2016-02-12 12:52:37
tags:
-	node
- 	rabbitmq
-  back-off
-  retry
	
---

使用第三方API的一个常见问题是峰值期间可用性问题。即便是非峰值期间，API请求仍可能被拒绝、超时或者失败。这篇文章中，我将讲解对于RabbitMQ消费者轻量级的[指数退避](http://en.wikipedia.org/wiki/Exponential_backoff)和重试机制。

## New API, new service

当用户从我们的一个网站删除了一张照片之后，我们会将它从我们的CDN服务中删除。我们的后端是面向服务的架构，这里使用了CDN purge服务。这个服务对核心web应用发生的事情做出反应，发送消息队列，并且调用CDN的API清除资源。最近我更新了这个服务来使用[Akamai CCU REST API](https://api.ccu.akamai.com/ccu/v2/docs/index.html)。

由于现有的服务已经存在很久了，我们决定重写它。对于一个初级开发者来说，这是一个很好的机会使用[RabbitMQ](http://www.rabbitmq.com/)和[Node.js](http://nodejs.org/)来实现后台任务。

正如Paul在他最近的blog中所说[RabbitMQ: Front the Front Line](http://dev.venntro.com/2014/04/rabbitmq-from-the-front-line/)，我们是RabbitMQ的重度使用者。尽管我们很多消费者都是实用Ruby来实现的，但是我们发现Node.js在很少的业务逻辑下也是非常实用。事件驱动和消息流很契合，并且适用于快速编码。我们实用[node-amqp](https://github.com/postwait/node-amqp)作为我们的RabbitMQ客户端。

一旦新的服务运行起来，很显然负载会很高，尤其是高峰8pm-2am。我们很快发现，我们的API被集中在晚上调用。这个API有一个内部队列用来处理purge请求，当队列满了的时候它会返回`507 queue is full`。

![CDN response pattern over one week
](http://dev.venntro.com/images/uploads/2014/07/week-of-api-responses.png)

观察调用成功和失败的API，很显然我们在高峰时期需要强大的退避和重试机制。

## Starting point

我们的平台发送多个不同的消息路由至CDN purge队列。CDN purge服务消费这些消息并且发送对应的请求给CDN API。下图描述了从平台到API的信息流。

![Diagram of our CDN purge process](http://dev.venntro.com/images/uploads/2014/07/cdn-purge-messages.png)

当API请求被拒绝的时候，我们想过一段时间再重试。与此同时，我们该如何处理消息？

## Let RabbitMQ do the work

实现退避和重试机制，我的第一直觉是创建一个新的`wait queue`，把失败的请求放在里面，过一段时间继续重试。由于我刚刚使用RabbitMQ，有这么几个问题需要考虑：

* 是否我需要消费者处理`wait queue`中的消息
* 是否我能控制每个消息的重试等待时间
* 是否我能追踪每个API请求我重试的次数
* 是否能在一个`wait queue`中处理多个平台的事情

感谢的是，RabbitMQ有很多[protocol extensions](https://www.rabbitmq.com/extensions.html)扩展了[AMQP](http://www.amqp.org/)规范。
`dead letter exchanges`和`per-message TTL`正是`wait queue`需要的功能。

## Dead letter exchanges (DLX)

术语[dead letter mail 死信邮件](http://en.wikipedia.org/wiki/Dead_letter_mail)在邮政行业仍在使用，用来描述信件没有办法被送达。在RabbitMQ中，消息有以下几种情况的时候会被认为dead-lettered

* 消息被拒绝
* 消息已经过期
* 队列已经满了

邮政服务会将死信邮件返还给发送者，和这很相似，RabbitMQ会做一些工作并且重新将`dead-lettered`的消息发送给我们选择的exchange-dead letter exchange。

既然我们想要一个`wait queue`，消息过期最容易触发`dead-lettering`。我们将控制消息短期内过期。

任何队列都能设置dead-letter消息。`dead letter exchange`是队列的一个参数，当你声明的时候可以通过`x-dead-letter-exchange`参数来设定。这是一个使用[node-amqp](https://github.com/postwait/node-amqp)的例子：

```js
var queueOptions = { arguments: { "x-dead-letter-exchange": "exchange" } };

connection.queue("wait-queue", queueOptions, function(waitQueue) {
  // Bind to exchange
});
```

尽管`dead letter exchanges`没有做什么特别的配置，我们仍是有了`wait queue`。下一步我们将设置消息过期，这样RabbitMQ就能为我们重新分配。

更多信息: [RabbitMQ docs on DLX](https://www.rabbitmq.com/dlx.html)

## Per-message TTL

消息可以使用默认的`expiry`或者`TTL`来声明。然而为了实现指数退避，我们需要对每个消息逐个设置过期时间。

当你发布消息的时候，你可以设置expiration字段的毫秒值：

```js
var messageOptions = { expiration: 10000 };

exchange.publish("routing-key", "body", messageOptions);
```

看起很简单，但是这也意味着，当一个purge请求失败的时候，因为需要`expiration`字段，我们的消费者需要复制消息。注意：如果你声明了你的队列使用消息确认机制，不要忘记确认原来的消息。

```js
var subscribeOptions = { ack: true };

queue.subscribe(subscribeOptions, function(message, headers, deliveryInfo, messageObject) {
  // Post request to API
  // ...

  // If the API request fails
  var messageOptions = {
    appId: messageObject.appId,
    timestamp: messageObject.timestamp,
    contentType: messageObject.contentType,
    deliveryMode: messageObject.deliveryMode,
    headers: headers,
    expiration: 10000
  };
  exchange.publish(deliveryInfo.routingKey, message, messageOptions);
  messageObject.acknowledge(false);
});
```

你需要确保你复制你自己的消息的详细信息。下面让我们增加每次API调用失败时的的超时时间。

更多信息: [RabbitMQ docs on per-message TTL](https://www.rabbitmq.com/ttl.html)

## Handling dead-lettered messages

当消息`dead-lettered`之后，RabbitMQ对它做了一些很有用的变化，在header中记录了这些变化。对于我们的`wait queue`，我们仅仅关心对`expiration`字段发生了什么。

`expiration`字段被移除了，并且重新在headers中的`x-death`中作为`original-expiration`值记录着。这允许我们找到上次的过期时间并且防止消息再次过期。重要的是，`x-death`header是有序数组，第一个是最近的值。

```js
var expiration;

if (headers["x-death"]) {
  expiration = (headers["x-death"][0]["original-expiration"] * 3);
} else {
  expiration = 10000;
}
// Apply some randomness to the expiration
// ...
```

这个例子中，第一个个过期时间是10000毫秒，每次重试的时候它都会乘以3。这是指数退避算法中比较常见的实践。我们的例子中，我们增加了一点随即值来提高API调用的成功可能性。

下面我们来组织我们的队列，这样他能管理多个平台的事件。

## Routing dead-lettered messages

我们的CDN purge服务对于多个平台发生的事情做出反应，每个都有它自己的routing key。最简单的方式路由多个routing key的是声明单独的`wait exchange`。

使用单独的`wait exchange`之后，你可以不用理会`routing keys`。当失败的消息发送到`wait exchange`时，你不需要改变`routing key`。仅仅需要将`wait queue`和`primary queue`绑定同样的`routing key`。

```js
var routingKeys = ["routing-key-a", "routing-key-b"];

connection.exchange("wait-exchange", waitExchangeOptions, function(waitExchange) {
  var waitQueueOptions = { arguments: { "x-dead-letter-exchange": "exchange" } };

  connection.queue("wait-queue", waitQueueOptions, function(waitQueue) {
    // Bind wait queue to all routing keys on wait exchange
    routingKeys.map(function(routingKey) {
      waitQueue.bind("wait-exchange", routingKey);
    });
  });
});

connection.exchange("primary-exchange", primaryExchangeOptions, function(primaryExchange) {
  connection.queue("primary-queue", primaryQueueOptions, function(primaryQueue) {
    // Bind primary queue to all routing keys on primary exchange
    routingKeys.map(function(routingKey) {
      primaryQueue.bind("primary-exchange", routingKey);
    });
    // Subscribe to messages
    // ...
  });
});
```

这样配置之后，当消息被`wait queue` `dead-lettered`之后，会重新发布至`primary exchange`，`routing key`会保持不变。这样以后添加和移除`routing key`会很简单。

## All together now

现在把所有的代码合并在一起。

```js
var amqp = require("amqp");

connection = amqp.createConnection({ host: "localhost" });

connection.on("ready", function() {
  var waitExchange,
      routingKeys = ["routing-key-a", "routing-key-b"];

  waitExchange = connection.exchange("wait-exchange", waitExchangeOptions, function(waitExchange) {
    var waitQueueOptions = { arguments: { "x-dead-letter-exchange": "primary-exchange" } };

    connection.queue("wait-queue", waitQueueOptions, function(waitQueue) {
      routingKeys.map(function(routingKey) {
        waitQueue.bind("wait-exchange", routingKey);
      });
    });
  });

  connection.exchange("primary-exchange", primaryExchangeOptions, function(primaryExchange) {
    connection.queue("primary-queue", primaryQueueOptions, function(primaryQueue) {
      var subscribeOptions = { ack: true };

      routingKeys.map(function(routingKey) {
        primaryQueue.bind("primary-exchange", routingKey);
      });

      primaryQueue.subscribe(subscribeOptions, function(message, headers, deliveryInfo, messageObject) {
        var expiration, messageOptions;
        // Post request to API
        // ...

        // If the API request fails
        if (headers["x-death"]) {
          expiration = (headers["x-death"][0]["original-expiration"] * 3);
        } else {
          expiration = 10000;
        }
        messageOptions = {
          appId: messageObject.appId,
          timestamp: messageObject.timestamp,
          contentType: messageObject.contentType,
          deliveryMode: messageObject.deliveryMode,
          headers: headers,
          expiration: expiration
        };
        waitExchange.publish(deliveryInfo.routingKey, message, messageOptions);
        messageObject.acknowledge(false);
      });
    });
  });
});
```

## Summary

我已经介绍了如何使用`dead letter exchanges`和`per-message TTL`来实现轻量级别的指数退避和重试机制。上面的示例代码展示了在Node.js中使用node-amqp如何实现。下图表述了这个机制原理：

![Diagram of back-off and retry mechanism](http://dev.venntro.com/images/uploads/2014/07/rabbitmq-back-off-and-retry.png)

如果你和第一个图比较，我希望可以清楚的解释其中原理。最后，下面对开始的问题做一些简要的回答：

### 是否我需要消费者处理`wait queue`中的消息?

否。RabbitMQ可以做这个工作，声明一个带`x-dead-letter-exchange`参数的`wait queue`，RabbitMQ会在他们过期的时候重新发布。

### 是否我能控制每个消息的重试等待时间?

可以。但是`per-message TTL`仅能在你发送消息的时候设置。所以的你的消费者需要复制消息体并且发送的时候需要携带`expiration`字段。注意：如果使用了消息确认机制，不要忘了确认原来的消息。

### 是否我能追踪每个API请求我重试的次数?

是。每次消息变为dead-lettered时，RabbitMQ都会记录有用的信息在它的`x-death`header。第一条记录的是最新的，并且会包含`original-expiration`

### 是否能在一个`wait queue`中处理多个平台的事情

是。最简单的管理多个`routing key`的做法是为你的`wait queue`声明一个单独的exchange，然后和`primary exchange`绑定同样的`routing key`。

## 原文 [Back-off and retry with RabbitMQ](http://dev.venntro.com/2014/07/back-off-and-retry-with-rabbitmq/)



