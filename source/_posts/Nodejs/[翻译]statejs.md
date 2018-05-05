title: statejs
date: 2014-07-24 05:59:31
tags:
---
States
=====
 `State` 的实例封装了`owner`在指定时刻下的条件和行为。 一个`State`是由methods, arbitrary data, events, guards, substates和 transition expressions的集合组成。
一个owner通常拥有多个`State`s, 并且指向其中一个作为 current state, 在current state中owner会表现出相应的方法和行为。通过行为来触发 transition使得由一个状态转变为另外一个状态。
Object model
==========
`State` objects are modeled by a set of relational references that define three distinct dimensions or “axes”.状态对象是由一组定义三个不同的维度或“轴”关系的参考蓝本。
从根本上来说，`State` 模型是分层次的，从 owner 拥有着唯一的root state开始，每个`state`都作为 superstate被 substates 继承。一个 `State` 可能从零个或者多个 parastates继承，提供了定义成分的关系，通过C3线性多重继承实现的一种手段。
如果owner的`state`树是由其他owner的prototypal继承而来 ，继承的owner将可以view their prototype’s States as protostates from which their own epistates inherit.
```javascript
var p = {};
state(p, {
    A: state,
    B: state({
        BA: state,
        BB: state
    })
});

var o = Object.create(p);
state(o, {
    A: state({
        AA: state.extend('X, Y')
    }),
    X: state,
    Y: state
});
```
State S 的继承顺序如下 relation precedence:
1.S的protostate继承链
2.S的parastates，按照声明的先后顺序
3.S的superstate 
其中parastate和superstate线性加载，或者并行，protostate则按照顺序加载
The root state
-------------------
所有被 `State` 影响的 owner 都有单独的 root state。root state的名称为唯一的空字符串 ''.
```javascript
owner.state().root === owner.state('');   // >>> true (invariant)
owner.state('->');
owner.state();                            // >>> State ''
```
默认的，owner的初始current state被设置为 root state — 除非 root state 被标记为 abstract attribute，或者另外一个 `State` 被标记为 initial attribute.
root state 还可以用来存储 methods 并且可以被owner的 `State`s重写。这些方法被替换至root state。如果没有被重写，则会调用默认的方法。
Superstates and substates
------------------------------------
**owner** 表现得行为由 **substates** 决定 ，这些**substates** 由 **superstates** 生成
```javascript
var o = {};
state( o, {
    A: state({
        AA: state,
        AB: state
    }),
    B: state
});

var root = o.state('');                    // >>> RootState
o.state() === root;                        // >>> true
o.state('-> AA');
o.state() === o.state('AA');               // >>> true
o.state().superstate === o.state('A');     // >>> true
o.state().superstate.superstate === root;  // >>> true
```
Parastates and composition
-------------------------------------
除了通过 **superstates** 继承，`State` 模型还支持通过 **parastate** 关系实现多继承。
最孤单的就是RootState，它不承担任何关系。
Parastates 通过 `state.extend` 函数声明, 参数为一个或多个字符串 to parastates, along with optional parameters attributes and expression, and returns a StateExpression that can be used to produce a State with the named parastates.
```javascript
var o = {};
state( o, {
    A: state({
        AA: state.extend('X, Y')
    }),
    X: state,
    Y: state
});
```
状态表达式(Expressions)
====================
`State` 通过 *state expression* 数据结构以声明的方式定义。
`StateExpression` 通过调用不包含`owner`参数的 `state()` 函数创建，仅提供一个 `expression` 参数，还可以提供由空格分隔的 `attributes`参数。
在其内部， 状态表达式的内容根据对象的 categories 设置，其中包含 `data`, `methods`, `events`, `guards`, `substates` 和 `transitions`。`state()` 的 `expression`可以根据这些 categories形成结构化，或者可以通过类型推断来简化实现一些更加简单的缩写。

结构化状态表达式(Structured state expressions)
----------------------------------------------------------------
以入门的例子为基础，我们可以写一个包含明确分类的expression，包含子状态、方法、事件等。
```javascript
var longformExpression = state({
    methods: {
        greet: function () { return "Hello."; }
    },
    states: {
        Formal: {
            methods: {
                greet: function () { return "How do you do?"; }
            },
            events: {
                enter: function () { this.wearTux(); }
            }
        },
        Casual: {
            methods: {
                greet: function () { return "Hi!"; }
            },
            events: {
                enter: function () { this.wearJeans(); }
            }
        }
    }
});
// >>> StateExpression
```
简写(Shorthand)
---------------------
明确的分类是很准确的，但是这样显得很冗余。所有 state() 同样接受更加简便的表达形式。效果同上面的一样。
```javascript
var shorthandExpression = state({
    greet: function () { return "Hello."; },

    Formal: {
        enter: function () { this.wearTux(); },
        greet: function () { return "How do you do?"; }
    },
    Casual: {
        enter: function () { this.wearJeans(); },
        greet: function () { return "Hi!"; }
    }
});
// >>> StateExpression
```
在这个例子中，`state()` 根据以下规则解释：
*识别缺乏键名的类别 ，推断 `Formal` 和 `Casual` 为状态
*将 `enter` 识别为内置的事件类型，将关联的函数作为 `enter` 事件的监听器，别且在被所在的 `state` 触发。
*推断 `greet` 不是一个内置的事件类型，所以被认为是一个方法。
所有显示的定义可以通过简写来实现，但是要避免出现含糊不清的情况 (例如，创建一个名为 `data` 的状态，或者方法名叫做 `enter`。

表达式推断(Interpreting expression input)
------------------------------------------------------
提供给 `state()`的 `StateExpression`根据以下规则推断解释：
1.如果输入的值的类型为`StateExpression`或`TransitionExpression`并且推断如此，则使用输入的键名为它的名称，如果输入的值是`state`函数自身，则推断为名称为键名的空状态。
2.如果键名是类名，它的值是一个对象或null，那么它将被解释，因为它会在长期形成的结构化格式。
3.如果键名同 built-in event type 匹配，或者值是string， 则推断值作为一个事件监听函数，, an array of event listeners, or a named transition target to be bound to that event type.
4.Otherwise, if an entry’s key matches a guard action (i.e., admit, release), interpret the value as a guard condition (or array of guard conditions).
5.Otherwise, if an entry’s value is an object, interpret it as a substate whose name is the entry’s key, or if the entry’s value is a function, interpret it as a method whose name is the entry’s key.






Transitions过渡
===========
Whenever an object’s current state changes, a transition state is created, which temporarily assumes the role of the current state while the object is travelling from its origin or source state to its target state.
每当一个对象的当前状态的变化，会创建一个过渡状态，这暂时假定当前状态的角色，而对象从它的源状态变化到目标状态。
Transition expressions
------------------------------
A state expression may include any number of transition expressions, which define some action to be performed, either synchronously or asynchronously, along with selectors for the origin/source and target states to which the transition should apply, and guards to determine the appropriate transition to employ.
状态表达式可以包括任何数量的过渡表达式，它定义要执行的操作，同步或异步，以及选择的原产地/来源和目标状态的过渡应适用的，和警卫来确定合适的过渡聘请。
Before an object undergoes a state change, it examines the transition expressions available for the given origin and target, and selects one to be enacted. To test each expression, its origin state is validated against its admit transition guards, and its target state is validated against its release transition guards. The object then instantiates a Transition based on the first valid transition expression it encounters, or, if no transition expression is available, a generic actionless Transition.
对象经历一个状态的改变之前，它会检查过渡表达式对于源状态和目标状态是否可用，并选择一个制定。要测试每个表达式，它的起源状态进行验证针对其承认过渡卫士，并将其目标状态及其对过渡释放警卫进行了验证。该对象，然后实例化基于它遇到的第一个有效的过渡表达一个过渡，或者，如果没有过渡表达式是可用的，一个通用的无为过渡。
Where transition expressions should be situated in the state hierarchy is largely a matter of discretion. In determining the appropriate transition expression for a given origin–target pairing, the search proceeds, in order:
过渡表达式在状态层次的设置应该非常慎重。
1.at 过渡表达式的`target`状态(CSS3过渡是相对于类声明的方式)
2.at 过渡表达式的`origin`状态
progressively up the superstate chain of target
progressively up the superstate chain of origin
Transitions can therefore be organized in a variety of ways, but ambiguity resolution is regular and predictable, as demonstrated with the Zig transition in the example below:
过渡可以实现为不同的方式，但是这是有规律并且可预测的，如示例中`Zig`过渡
```js

```


