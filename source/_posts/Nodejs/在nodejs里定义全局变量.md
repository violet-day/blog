title: 在nodejs里定义全局变量
date: 2014-08-05 05:44:12
tags:
-	node
-	全部变量

categories:
-	node

---
如果你正在使用一系列node模块，或许是一个像Express.js一样的框架，突然需要使用几个全局变量。怎样在nodejs里创建全局变量呢？
对此最常见的建议是“不使用‘var’关键字声明一个变量”或“给object对象添加一个变量”或“给OBJECT对象添加一个变量”。你会使用哪种方式呢？
首先，让我们分析下global对象。打开一个终端，启动一个node命令提示界面：
```shell
$ node
> 
```
在命令提示界面下看看关于global的所有信息：
```shell
> global
```
飞天怪物的神圣母亲!!!那是一个超级大的对象！事实上，你看到了node的核心。所有在node进程里的对象都挂在这个对象上。如果你非常熟悉javascript所在的浏览器环境，global对象是等同于window对象。
现在我们已经指导global对象是声明，让我们把玩下它：
```shell
> global.name
undefined
> global.name = 'El Capitan'
> global.name
'El Capitan'
> GLOBAL.name
'El Capitan'
> delete global.name
true
> GLOBAL.name
undefined
> name = 'El Capitan'
'El Capitan'
> global.name
'El Capitan'
> GLOBAL.name
'El Capitan'
> var name = 'Sparrow'
undefined
> global.name
'Sparrow'
```

令人兴奋的观察！

global和BLOBAL看起来是一个同一个东西且是一回事。确实，Global是global一个别名。

令我们超感兴趣的是一个使用还是不使用var关键字声明的变量附加到global对象上。在node里创建一个全局变量的最基本的方式就是通过不使用var关键字声明一个变量。这种做法与使用一个模块(module)略有不同，接下来我讲解释它。

当你启动一个node进程，将会启动一个模块，那么所有的模块将会被包含在它里面，所有模块都共享同一个global对象。应用上面的测试观察结合实际，你就会明白全局变量怎样在node里工作。然而有些轻微的变化，如果使用var关键字声明的变量将会保留在本地模块里；这些声明的变量没有附加到global对象里。

那么现在你已经知道”没有使用var关键字声明的变量“，”向global里添加一个变量“，”给GLOBAL对象添加一个变量“，所有这些都是一回事。

在一个module里全局声明的变量能够被其他任何模块使用它们的名字来引用，没必要从global对象引用它们。但是这不意味这你就可以这样做。为什么呢？请看这样:
```js
var company = 'Yahoo';
console.log(global.company); // 'Google'
console.log(company); // 'Yahoo'
```

当你使用global.company时,你知道你使用的是全局变量，但是它的备用名字company在module是当作局部变量来使用的。

如果你打算在你的node应用里使用全局变量，那么我们讨论的创建变量方法会工作的很好。然而，请不要过度使用它。话虽如此，还有可以不使用全局对象的替代解决方案吗？

是的，有这么一个，它涉及到module.exports的使用。让我使用例子来演示：

File: main.js
```js
exports.company = 'Google';
var m = require('./mod');
```

File: mod.js
```js
var company = require('./main').company;
console.log(company);
```

现在看看执行结果：
```shell
$ node main.js
Google
```

这样你就实现了，一个其他模块的的变量你可以在另外的模块中使用他而没有使用global对象。你可以include main.js在其他的module来访问company名字。

注意：引用（include）一个已经被其他模块引用过的模块时，仅仅创建一个指向之前包体的引用，因此这意味着不会极度消耗内存。也因为没有重新创建一个真正的包体，在module里的所有初始化方法没有再执行。

因此，一个包体中，有2钟创建node全局变量的方法，一个是使用global对象，另一个是使用modules.exports。哪个是我推荐的呢?global方法适用小的应用，modules.exprots适用于大的应用。

原文：[http://www.hacksparrow.com/global-variables-in-node-js.html](http://www.hacksparrow.com/global-variables-in-node-js.html)