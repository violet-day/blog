title: hubot-scripting
date: 2016-02-14 13:24:37
tags:
-	node
- 	hubot


---

## Anatomy of a script

当你创建了你的hubot之后，生成器同样创建了一个scripts目录。你如果打开看看，会发现一样示例脚本。为了让脚本生效，你需要：

* 脚本需要位于hubot的脚本加载目录中(默认为`src/scripts`和`scripts`)
* `.coffee`或`.js`文件
* export为一个函数

export为一个函数，即：

```js
module.exports = (robot) ->
  # your code here
```

参数`robot`是的一个robot的一个实例。现在我们可以干一些碉堡了的事情了。

## Hearing and responding

既然这是一个聊天机器人，那么最常见的互动方式就是基于消息的。Hubot可以`hear`房间中说的消息，也可以直接`respond`。这些方法都接受一个正则表达式和一个回调函数作为参数。例如：

```js
module.exports = (robot) ->
  robot.hear /badger/i, (res) ->
    # your code here

  robot.respond /open the pod bay doors/i, (res) ->
    # your code here
```

`robot.hear /badger/`，回调函数会在任何任何消息文本匹配时之行。比如：

* Stop badgering the witness
* badger me
* what exactly is a badger anyways


`robot.respond /open the pod bay doors/i`的回调函数仅在机器人的名字或别名在消息文本前面的时候执行。如果机器人的名字时HAL，别名时/，这些情况下回调触发：

* hal open the pod bay doors
* HAL: open the pod bay doors
* @HAL open the pod bay doors
* /open the pod bay doors

这些情况下不会触发：

* HAL: please open the pod bay doors
	* 因为`respond`需要文本信息之前跟着机器人名称
* has anyone ever mentioned how lovely you are when you open the pod bay doors?
	* 缺少机器人名称

## Send & reply

`res`参数是`Response`的一个实例(historically, this parameter was msg and you may see other scripts use it this way)。你可以通过它`send`消息回去，`emote`消息（如果你的adapter支持的话），或者`reply`那个发送消息的用户。例如：

```js
module.exports = (robot) ->
  robot.hear /badger/i, (res) ->
    res.send "Badgers? BADGERS? WE DON'T NEED NO STINKIN BADGERS"

  robot.respond /open the pod bay doors/i, (res) ->
    res.reply "I'm afraid I can't let you do that."

  robot.hear /I like pie/i, (res) ->
    res.emote "makes a freshly baked pie"
```

`robot.hear /badgers/`的回调不管谁发送的消息直接回复，"Badgers? BADGERS? WE DON'T NEED NO STINKIN BADGERS"。

如果一个用户Dave说 "HAL: open the pod bay doors", `robot.respond /open the pod bay doors/i`的回调函数就会发送消息"Dave: I'm afraid I can't let you do that."

## Capturing data

至今，我们的脚本都是静态的回复，相对比较无趣一点。`res.match`包含了消息中正则匹配的部分。这是JavaScript的特性，返回一个数组，索引为0的是匹配正则表达式的全文本。比如：

```js
  robot.respond /open the (.*) doors/i, (res) ->
    # your code here 
```

如果Dave说"HAL: open the pod bay doors", `res.match[0]`是"open the pod bay doors", `res.match[1]`就是"pod bay"。现在，我们可以做一些更动态的事情了：

```js
  robot.respond /open the (.*) doors/i, (res) ->
    doorType = res.match[1]
    if doorType is "pod bay"
      res.reply "I'm afraid I can't let you do that."
    else
      res.reply "Opening #{doorType} doors"
```

## Making HTTP calls *Unmaintained*

## JSON

## XML

## Screen scraping

## Random
一个常见的场景就是听到或者相应一个命令，从数组中随即发送图片或者文本。JavaScript和CoffeeScript并没有提供什么方法，所以Hubot提供了一个方便的方法:

```js
lulz = ['lol', 'rofl', 'lmao']

res.send res.random lulz
```

## Topic

如果adapter支持的话，Hubot可以对房间的主题变更作出相应的反应。

```js
module.exports = (robot) ->
  robot.topic (res) ->
    res.send "#{res.message.text}? That's a Paddlin'"
```

## Entering and leaving

如果adapter支持，Hubot可以看到用户进入和离开。

```js
enterReplies = ['Hi', 'Target Acquired', 'Firing', 'Hello friend.', 'Gotcha', 'I see you']
leaveReplies = ['Are you still there?', 'Target lost', 'Searching']

module.exports = (robot) ->
  robot.enter (res) ->
    res.send res.random enterReplies
  robot.leave (res) ->
    res.send res.random leaveReplies
```

## Custom Listeners

上面涵盖了普通用户的大部分功能需求 (hear, respond, enter, leave, topic)，有时候，我们需要特别的匹配逻辑。如果这样，你可以使用`listen`来制定自定义的函数，而不是正则表达式。

如果想回调函数执行，match函数必须返回可以转化为true的值，返回值会传递给`response.match`

```js
module.exports = (robot) ->
  robot.listen(
    (message) -> # Match function
      # Occassionally respond to things that Steve says
      message.user.name is "Steve" and Math.random() > 0.8
    (response) -> # Standard listener callback
      # Let Steve know how happy you are that he exists
      response.reply "HI STEVE! YOU'RE MY BEST FRIEND! (but only like #{response.match * 100}% of the time)"
  )
```

## Events

Hubot还可以对事件作出相应，这可以用来script之间传递数据。`robot.emit`和`robot.on`通过Nodej.js的`EventEmitter`封装。

一个例子就是有一个脚本和服务互动，当请求来的时候触发事件。比如，我们可以有一个脚本从Github的post-commit钩子接受数据，触发commit事件，另一个脚本处理这些commits。

```js
# src/scripts/github-commits.coffee
module.exports = (robot) ->
  robot.router.post "/hubot/gh-commits", (req, res) ->
    robot.emit "commit", {
        user    : {}, #hubot user object
        repo    : 'https://github.com/github/hubot',
        hash  : '2e1951c089bd865839328592ff673d2f08153643'
    }
```

```js    
# src/scripts/heroku.coffee
module.exports = (robot) ->
  robot.on "commit", (commit) ->
    robot.send commit.user, "Will now deploy #{commit.hash} from #{commit.repo}!"
    #deploy code goes here
```
    
如果提供事件，非常建议包含一个在数据中包含一个hubot用户或者房间，这样允许hubot来在聊天中通知用户或房间。

## Error Handling

没有代码是完美的，错误和异常都是可以接受的。先前，未捕获的异常会导致hubot实例crash。Hubot现在包含了`uncaughtException`处理，他来提供脚本来做一些异常处理。

```js
# src/scripts/does-not-compute.coffee
module.exports = (robot) ->
  robot.error (err, res) ->
    robot.logger.error "DOES NOT COMPUTE"

    if res?
      res.reply "DOES NOT COMPUTE"
```

这里你可以做任何处理处理，但是如果你想做一些额外的补救措施、记录日志的话，最好是异步的代码。否则，你可能会遇到递归错误而且还不知道。

会有一个`error`事件触发，用错误处理函数来消费这个事件。因此，不管会不会发生，你都要处理好自己的异常，并且自己触发他们。第一个参数是触发的错误对象，第二个是可选参数来描述错误。用上面一个例子：

```js
  robot.router.post '/hubot/chatsecrets/:room', (req, res) ->
    room = req.params.room
    data = null
    try
      data = JSON.parse req.body.payload
    catch err
      robot.emit 'error', err

    # rest of the code here


  robot.hear /midnight train/i, (res)
    robot.http("https://midnight-train")
      .get() (err, res, body) ->
        if err
          res.reply "Had problems taking the midnight train"
          robot.emit 'error', err, res
          return
        # rest of code here
```

第二个例子中，很值得思考下用户会看到什么样子的信息。如果你有一个错误处理函数来回复用户，你可能不需要添加一个自定义的错误提示给用户，但是这个也取决于你对异常提示有多公开。

## Persistence

Hubot有一个基于内存的key-value存储，通过`robot.brain`来存储、设置数据。

```js
robot.respond /have a soda/i, (res) ->
  # Get number of sodas had (coerced to a number).
  sodasHad = robot.brain.get('totalSodas') * 1 or 0

  if sodasHad > 4
    res.reply "I'm too fizzy.."

  else
    res.reply 'Sure!'

    robot.brain.set 'totalSodas', sodasHad+1
robot.respond /sleep it off/i, (res) ->
  robot.brain.set 'totalSodas', 0
  msg.reply 'zzzzz'

```
  
如果脚本需要寻找用户信息，有一些方法在`robot.brain`中可以用来通过id、name或者模糊匹配来查找一个或多个用户:`userForName`, `userForId`, `userForFuzzyName`, `usersForFuzzyName`。

```js

module.exports = (robot) ->

  robot.respond /who is @?([\w .\-]+)\?*$/i, (res) ->
    name = res.match[1].trim()

    users = robot.brain.usersForFuzzyName(name)
    if users.length is 1
      user = users[0]
      # Do something interesting here..

      res.send "#{name} is user - #{user}"
```

## LISTENER METADATA

出了正则表达式和回调函数，`hear`和`respond`接受一个可选的Object参数，可以很容易的对即将创建的Listener对象添加metadata信息。metadata可以很容易的扩展脚本信息而不需要修改脚本本身。

最重要而且最常见的metadata的key是`id`，每个Listener都应有被给予一个唯一的id(option.id，默认为null)。通过模块名来划分命名空间('my-module.my-listener')。这些名称允许其他脚本很轻松的定位listeners，扩展像authorization和rate limiting的功能也不需要额外的函数。

额外的扩展可能需要定义额外的metadata key。

提前看个例子：

```js
module.exports = (robot) ->
  robot.respond /annoy me/, id:'annoyance.start', (msg)
    # code to annoy someone

  robot.respond /unannoy me/, id:'annoyance.stop', (msg)
    # code to stop annoying someone
```

这些定义允许你扩展一些新的行为，比如：

* authorization策略，允许annoyers组的每个人执行`annoyers.*`命令
* rate limiting:30分钟内只能调用`annoyance.start`一次

## MIDDLEWARE

有三种类型的中间件: Receive, Listener and Response.

Receive 中间件在 listeners 检查之前运行
Listener 中间件运行在每个listener匹配消息之后
Response 中间件在每个消息发送出去时运行

### Execution Process and API

和[Express middleware](http://expressjs.com/api.html#middleware)相似, 
Hubot按定义顺序执行中间件。每个中间件通过`next`继续中间件，通过`done`打断中间件。

Middleware调用的时候有以下参数：

* `context`
	* 详见每个不同中间件的的API
* `next`
	* 一个没有任何额外属性的函数，用来被执行继续下一个中间件或者执行Listener的回调函数。
	* `next`调用时有一个可选参数，可以是`done`函数，或者是一个新的最终会调用`done`的函数。如果参数未指定，则认为传入了`done`
* `done`
	* 一个没有任何额外属性的函数。当需要结束中间件调用和结束整个函数时调用。
	* 调用时没有参数
	
每个中间键都接受相同的API签名，`context`、`next`和`done`。不同类型的中间件在context中接受不同的信息。

### Error Handling

对于同步的中间件（不会产生事件循环），hubot会自动捕获错误并且触发`error`事件。Hubot还会调用最近的`done`回调来结束中间件调用。异步中间件应该自己捕获异常，触发error事件，调用done函数。任何未捕获的异常都会打断中间件的所有回调。

## LISTENER MIDDLEWARE

Listener中间件在匹配消息和执行listener之间插入逻辑。这允许你为每个匹配的脚本创建扩展。比如，集中的认证策略、调用限制、日志、指标。Middleware的实现和其他脚本一样，但是并不是使用`hear`和`respond`，中间件使用`listenerMiddleware`

### Listener Middleware Examples

完整的例子可以参见[hubot-rate-limit](hubot-rate-limit)。

一个记录执行命令的中间件：

```js

module.exports = (robot) ->
  robot.listenerMiddleware (context, next, done) ->
    # Log commands
    robot.logger.info "#{context.response.message.user.name} asked me to #{context.response.message.text}"
    # Continue executing middleware
    next()
```    
   
这个例子中，每个匹配的消息都会写下日志信息。

更加复杂的调用限制的例子：

```js
module.exports = (robot) ->
  # Map of listener ID to last time it was executed
  lastExecutedTime = {}

  robot.listenerMiddleware (context, next, done) ->
    try
      # Default to 1s unless listener provides a different minimum period
      minPeriodMs = context.listener.options?.rateLimits?.minPeriodMs? or 1000

      # See if command has been executed recently
      if lastExecutedTime.hasOwnProperty(context.listener.options.id) and
         lastExecutedTime[context.listener.options.id] > Date.now() - minPeriodMs
        # Command is being executed too quickly!
        done()
      else
        next ->
          lastExecutedTime[context.listener.options.id] = Date.now()
          done()
    catch err
      robot.emit('error', err, context.response)
```

这个例子中，中间件检查listener是否在最近的1s中被调用。如果有，中间件里面调用`done`，阻止listenr的回调调用。如果listenr允许执行，中间件在next中添加`done`，这样就能调用记录结束时间。

这个例子同样展示了通过特定的metadata可以创建出很有用的扩展：使用限制中间件，脚本开发人员可以通过设置listener option参数创建调用限制的命令。

```js
module.exports = (robot) ->
  robot.hear /hello/, id: 'my-hello', rateLimits: {minPeriodMs: 10000}, (msg) ->
    # This will execute no faster than once every ten seconds
    msg.reply 'Why, hello there!'
```

### Listener Middleware API

Listener中间件的回调函数接受3个参数:`context`、`next`和`done`。

`context`包含这些字段：

* `listener`
	* 	options: 一个Object对象，包含listener中定义的metadata。
* `response`
	* 所有的response API都包含
	* 中间件用一些额外的信息来装饰（不是修改）response（比如为`response.message.user`添加LDAP groups信息）
	* 注意：文本信息(`response.message.text`)应该被考虑为不要改变

## RECEIVE MIDDLEWARE

Receive中间件在所有的listeners执行之前。很适合用来实现黑名单功能而不用考虑ID，metrics等。

### Receive Middleware Example

示例中间件禁止特定的用户使用包括`hear`在内的功能。如果一个用户想使用一个命令，会返回一条错误信息。

```js
BLACKLISTED_USERS = [
  '12345' # Restrict access for a user ID for a contractor
]

robot.receiveMiddleware (context, next, done) ->
  if context.response.message.user.id in BLACKLISTED_USERS
    # Don't process this message further.
    context.response.message.finish()

    # If the message starts with 'hubot' or the alias pattern, this user was
    # explicitly trying to run a command, so respond with an error message.
    if context.response.message.text?.match(robot.respondPattern(''))
      context.response.reply "I'm sorry @#{context.response.message.user.name}, but I'm configured to ignore your commands."

    # Don't process further middleware.
    done()
  else
    next(done)
```

## Receive Middleware API

Receive中间件的回调函数接受3个参数:`context`、`next`和`done`。

`context`包含这些字段：

* response
	* response此时并没有match属性，因为listeners还没有被执行所以还没有匹配。
	* 中间件用一些额外的信息来装饰（不是修改）response（比如为`response.message.user`添加LDAP groups信息）
	* 中间可以修改`response.message`对象

## RESPONSE MIDDLEWARE

Response中间件在hubot发送消息给聊天室的时候执行。对消息格式化，防止密码泄漏，指标等很有用。

### Response Middleware Example

示例修改了发送至聊天室的链接。

```js
module.exports = (robot) ->
  robot.responseMiddleware (context, next, done) ->
    return unless context.plaintext?
    context.strings = (string.replace(/\[([^\[\]]*?)\]\((https?:\/\/.*?)\)/, "<$2|$1>") for string in context.strings)
    next()
```

### Response Middleware API

Response中间件的回调函数接受3个参数:`context`、`next`和`done`。

`context`包含这些字段：

* `response`
	* response可以用来在中间件中发送新的消息。中间件会再次执行。小心无限循环。
* `strings`
	* 发送给聊天适配器的字符串数组。你可以修改这些信息，或者像`context.strings = ["new strings"]`这样来替换
* `method`
	* 字符串类型，表示listener发送的消息类型，比如`send`, `reply`, `emote` 或者 `topic`
* `plaintext`
	* `true`或者`undefined`。如果消息是正常的 plaintext类型，则会设为`true`。比如`send` 和`reply`。该属性只读。

## TESTING HUBOT SCRIPTS

[hubot-test-helper](https://github.com/mtsmfm/hubot-test-helper)是用来测试Hubot脚本的很好的测试框架。

安装包：
```bash
% npm install hubot-test-helper --save-dev
```

同时还需要安装:

* 一个测试框架，如 *Mocha*
* 断言库，如 *chai* 或者 *expect.js*

可能还需要安装:

* coffee-script (如果你想用CoffeeScript写测试)
* mock的库，比如 *Sinon.js* (如果你的脚本运行是webservice或者其他异步的服务)

这里是个简单的例子，使用 `Mocha`, `chai`, `coffee-script` 和 `hubot-test-helper`:

**test/example-test.coffee**

```js
Helper = require('hubot-test-helper')
chai = require 'chai'

expect = chai.expect

helper = new Helper('../scripts/example.coffee')

describe 'example script', ->
  beforeEach ->
    @room = helper.createRoom()

  afterEach ->
    @room.destroy()

  it 'doesn\'t need badgers', ->
    @room.user.say('alice', 'did someone call for a badger?').then =>
      expect(@room.messages).to.eql [
        ['alice', 'did someone call for a badger?']
        ['hubot', 'Badgers? BADGERS? WE DON\'T NEED NO STINKIN BADGERS']
      ]

  it 'won\'t open the pod bay doors', ->
    @room.user.say('bob', '@hubot open the pod bay doors').then =>
      expect(@room.messages).to.eql [
        ['bob', '@hubot open the pod bay doors']
        ['hubot', '@bob I\'m afraid I can\'t let you do that.']
      ]

  it 'will open the dutch doors', ->
    @room.user.say('bob', '@hubot open the dutch doors').then =>
      expect(@room.messages).to.eql [
        ['bob', '@hubot open the dutch doors']
        ['hubot', '@bob Opening dutch doors']
      ]
   
```

**sample output**

```bash
% mocha --compilers "coffee:coffee-script/register" test/*.coffee


  example script
    ✓ doesn't need badgers
    ✓ won't open the pod bay doors
    ✓ will open the dutch doors


  3 passing (212ms)

```


