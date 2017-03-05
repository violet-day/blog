title: object-pool实现之node版本分析
date: 2017-03-02 21:25:19
tags:

- generic-pool
- nodejs

---

## 前言

对象池作为一种设计模式，很多语言都有对应的实现。比较著名的应该就是[commons-pool](https://github.com/apache/commons-pool)。实际开发中，可能接触的比较的多的就是db的连接池。以下摘自[wiki](https://en.wikipedia.org/wiki/Object_pool_pattern)：

>	The object pool pattern is a software creational design pattern that uses a set of initialized objects kept ready to use – a "pool" – rather than allocating and destroying them on demand. A client of the pool will request an object from the pool and perform operations on the returned object. When the client has finished, it returns the object to the pool rather than destroying it; this can be done manually or automatically.
>	Object pools are primarily used for performance: in some circumstances, object pools significantly improve performance. Object pools complicate object lifetime, as objects obtained from and returned to a pool are not actually created or destroyed at this time, and thus require care in implementation.

通过[node-pool](https://github.com/coopernurse/node-pool)简单了解下`object-pool`的具体实现

## Project Struct

```
├── CHANGELOG.md
├── Makefile
├── README.md
├── index.js
├── lib
│   ├── DefaultEvictor.js
│   ├── Deferred.js
│   ├── Deque.js
│   ├── DequeIterator.js
│   ├── DoublyLinkedList.js
│   ├── DoublyLinkedListIterator.js
│   ├── Pool.js
│   ├── PoolDefaults.js
│   ├── PoolOptions.js
│   ├── PooledResource.js
│   ├── PooledResourceStateEnum.js
│   ├── PriorityQueue.js
│   ├── Queue.js
│   ├── ResourceLoan.js
│   ├── ResourceRequest.js
│   ├── errors.js
│   ├── factoryValidator.js
│   └── utils.js
├── package.json
└── test
    ├── doubly-linked-list-iterator-test.js
    ├── doubly-linked-list-test.js
    ├── generic-pool-acquiretimeout-test.js
    ├── generic-pool-test.js
    ├── resource-request-test.js
    └── utils.js
```

整个结构可以划分如下：
* 基本数据结构
	* DoublyLinkedList.js [双链表](https://en.wikipedia.org/wiki/Doubly_linked_list)
	* DoublyLinkedListIterator.js
	* Deque.js [双端对列](https://en.wikipedia.org/wiki/Double-ended_queue)
	* DequeIterator.js
	* Queue.js 
	* PriorityQueue.js
* Promise相关的封装
	* Deferred.js
* Pool相关
	* PooledResource.js
	* PooledResourceStateEnum.js
	* ResourceRequest.js
	* ResourceLoan.js
	* PoolDefaults.js
	* PoolOptions.js
	* DefaultEvictor.js 默认的evictor policy实现
	* Pool.js pool的主要逻辑实现

![](/images/generic-pool.jpeg)

## pool.js

### attribute

* waitingClientsQueue:ProtorityQueue 请求资源中的clients
* factoryCreateOperations:Set<Promise> 正在处理的创建资源的请求
* factoryDestroyOperations:Set<Promise> 正在处理的销毁资源的请求
* avaiableObjects:Deque 状态为idel的资源，因为存在fifo的配置选项，所以存储时选择的数据结构为双端对列
* testOnBorrowResources:Set 在获取之前处于testing中的资源
* testOnReturnResources:Set 在返回pool之前的testing资源
* validationOperations: Set<Promise>
* allObjects:Set 当前pool中所有的没有被destroy的资源
* resourceLoans：Map，结构为<resource,loan>

xxxOperations:Set<Promise>的作用为暂存执行create、desitory、validation中的请求

### getter

* potentiallyAllocableResourceCount： 可以被分配的资源的数量，为以下之和
	* availableObjects
	* testOnBorrowResources
	* testOnReturnResources
	* factoryCreateOperations
* count = allObjects + factoryCreateOperations：当前已经创建的资源和即将创建的资源之和
* spareResourceCapacity = config.max - count：当前还可以创建的数量

### function

**dispatchPooledResourceToNextWaitingClient(pooledResource)**

1. 从watingClientQueue dequeue一个request，如果request不存在，则将resource标记为idle并添加至availableObjects中，返回fasle
2. 如果存在request，构造ResourceLoan，并添加loan对象至resourceLoans中
3. pooledResouce标记为allocated
4. 执行request.resolve
5. 返回true

**dispatchResource(pooledResouce)**
	
1. 从availableObjects中取出resource，并执行dispatchPooledResourceToNextWaitingClient

**dispense()**

1. 记录waitingClientsQueue.length为局部变numWaitingClients，如果numWaitingClients<1则直接return
2. 记录numWaitingClients-potentiallyAllocableResourceCount的差值为resourceShotfall，即对于watingClient实际缺少的resource的数量
3. resourceShotfall和spareResourceCapacity中取最小值，即本次执行函数时，可以创建resource的数量
4. for循环执行createResource
5. testOnBorrow设置为true，计算出实际多少resource需要转入test状态，for循环执行testOnBorrow
6. testOnBorrow设置为false时，取availableObjects和numWaitingClients中较小的数值，for循环执行dispatchResource

**createResource()**

1. 执行factory.create，resolve之后
	1. 构造PooledResource实例
	2. allObjects.add
	3. dispatchPooledResourceToNextWaitingClient
2. 如果reject，则执行dispense，实现递归

**testOnBorrow()**

1. availableObjects中取出resource并标记状态为test
2. 添加址testOnBorrowResources中
3. 执行factory.validate，promise reoslve的结果为boolean类型，resolve结束时，从testOnBorrowResources移除
	1. 结果为true，执行dispatchPooledResourceToNextWaitingClient
	2. 结果为false，标记resource为invalidate；destroy resource；dispense，实现递归

**acqurie(priority)**

1. 判断是否draining，如果是则reject error
2. 判断waitingClientsQueue是否已经大于maxWaitingClients，如果是，则reject error
3. 构造ResourceRequest，并enqueue至waitingClientsQueue中
4. 执行dispense
5. 返回resourceRequest.promise

**release(resource)**

1. 在createResource的逻辑中，会将分配出去的resource添加至resourceLoans中，release的主要作用也是从resourceLoans将resouce移除
2. 标记resource状态为idel，并归还至availableObjects

## 总结

1. 整个项目3.x版本重构过了，使用了es6的class来构建模块，可以通过类图理解结构，好评
2. factory接口中定义的create、destory、validate的均为异步函数，关于在判断数量时均需要考虑正在create、destory和validte的数量。这里是factory中的promise通过set保存
3. resource对象有很多状态，对应的pool中就要有相应的容器来存放相应状态的resource或者promise，理解pool的关键就在此。
4. 感觉好难用图来表述逻辑，只能用文字了。

## Reference

* [Generic Pool](https://github.com/coopernurse/node-pool)
* [commons-pool](https://github.com/apache/commons-pool)


