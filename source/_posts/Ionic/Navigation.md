title: Navigation
tags:
  - ionic
  - Navigation

categories:
  - ionic

date: 2014-08-10 21:48:52

---

## ion-nav-view

当用户浏览整个应用程序，Ionic能够跟踪他们的浏览历史。通过了解他们的历史、视图之间的转换正确滑动或左或右，或没有过渡的。另一个优点是Ionic的导航系统是其管理多个历史能力。

Ionic使用AngularUI Router模块，所以app的接口可以组织为"states"。和Angular的核心$route服务一样,URLs可以用来控制视图。然而，AngularUI Router提供了更加强大的状态管理功能，状态与name绑定,并且有嵌套或并行的视图，在一个页面允许多个视图。并且，每个状态没有一定需要绑定URL，可以更加灵活的将数据附加至状态。

在你的应用程序用，ionNavView指示器用来渲染模板，每个模板都是一个状态的一部分。状态通常映射至URL，并且通过angular-ui-router来定义。（记住在示例中将ui-view替换为ion-nav-view）

### 用法

在这个例子中，我们将创建一个包含我们的不同状态的应用程序中的导航视图。

要做到这一点，在我们的标记，我们使用ionNavView顶级的指令。要显示一个标题栏，我们使用`ionNavBar`指令，更新，因为我们通过导航堆栈导航。

在navView的animation中，你可以使用任何`animation`的类。

推荐的效果: 'slide-left-right', 'slide-left-right-ios7', 'slide-in-up'.

```html
<ion-nav-bar></ion-nav-bar>
<ion-nav-view animation="slide-left-right">
  <!-- Center content -->
</ion-nav-view>
```

接下来，我们需要设置状态
```javascript
var app = angular.module('myApp', ['ionic']);
app.config(function($stateProvider) {
  $stateProvider
  .state('index', {
    url: '/',
    templateUrl: 'home.html'
  })
  .state('music', {
    url: '/music',
    templateUrl: 'music.html'
  });
});
```

当程序启动时，stateProvider会查看url，匹配index state，则会尝试去加载home.html至<ion-nav-view>

页面通过给定的URLs加载。在Angulr中创建templates的一个简单的办法就直接通过`<script type="text/ng-template">`语法在页面中创建。

```html
<script id="home" type="text/ng-template">
  <!-- The title of the ion-view will be shown on the navbar -->
  <ion-view title="'Home'">
    <ion-content ng-controller="HomeCtrl">
      <!-- The content of the page -->
      <a href="#/music">Go to music page!</a>
    </ion-content>
  </ion-view>
</script>
```

这样的好处是因为缓存加载会非常快，而不是通过异步加载。


## ion-view 
Child of `ionNavView`

### 用法
下面是一个例子，我们的页面将包含“My Page”为标题导航栏加载。
```html
<ion-nav-bar></ion-nav-bar>
<ion-nav-view class="slide-left-right">
  <ion-view title="My Page">
    <ion-content>
      Hello!
    </ion-content>
  </ion-view>
</ion-nav-view>
```

### API

|	属性	|	类型	|	详细说明
|:-----:|:--------------:|:------------------:
|	title(可选)	|	string	|	显示在`ionNavBar`的标题
|	hide-back-button(可选)	|	boolean	|	是否隐藏`ionNavBar`默认的后退按钮
|	hide-nav-bar(可选)	|	boolean	|	是否隐藏默认的`ionNavBar`

## ion-nav-bar 
Delegate: `$ionicNavBarDelegate`

如果我们已经有了`ionNavView`,我们同样可以创建`<ion-nav-bar>`来更新程序的状态。

我们可以在其中放置`ionNavBackButton`来添加返回按钮

我们可以根据当前的可见视图使用`ionNavButtons`添加按钮

通过在`animation`属性中添加animation class来达到title变化的动画效果。(推荐: 'nav-title-slide-ios7').

注意：`ion-nav-bar`仅仅在ionView中存在的情况下才起效果

### 用法
```html
<body ng-app="starter">
  <!-- The nav bar that will be updated as we navigate -->
  <ion-nav-bar class="bar-positive" animation="nav-title-slide-ios7">
  </ion-nav-bar>

  <!-- where the initial view template will be rendered -->
  <ion-nav-view>
    <ion-view>
      <ion-content>Hello!</ion-content>
    </ion-view>
  </ion-nav-view>
</body>
```

### API

|	属性	|	类型	|	详细说明
|:-----:|:--------------:|:------------------:
|	delegate-handle(可选)	|	stirng	|	用来被`$ionicNavBarDelegate`识别
|	align-title(可选)	|	string	|	 'left', 'right', 'center'. 默认为'center'.
|	no-tap-scroll(可选)	|	boolean	|	默认的，点击的时候，会自动将内容滚动至顶部。通过设置为false可禁用这一行为。


### 另外用法
你可能在每个视图中放置`ion-nav-bar`，这就要求你拥有整个nvabar，不仅仅contents、transition、view。

这与在ion-view中使用header bar相似，但他有一个导航栏的功能。

如果你这么做，很简单的就是在navbar放置nav buttons；不要使用`<ion-nav-buttons>`

```html
<ion-view title="myTitle">
  <ion-nav-bar class="bar-positive">
    <ion-nav-back-button>
      Back
    </ion-nav-back-button>
    <div class="buttons right-buttons">
      <button class="button">
        Right Button
      </button>
    </div>
  </ion-nav-bar>
</ion-view>
```

## ion-nav-buttons 
Child of `ionNavView`

在`ionView`通过`ionNavButtons`来设置你`ionNavBar`上的按钮。

任何你声明的按钮都会被放置在nvabar对应的地方，当离开视图的时候被销毁

### 用法
```html
<ion-nav-bar>
</ion-nav-bar>
<ion-nav-view>
  <ion-view>
    <ion-nav-buttons side="left">
      <button class="button" ng-click="doSomething()">
        I'm a button on the left of the navbar!
      </button>
    </ion-nav-buttons>
    <ion-content>
      Some super content here!
    </ion-content>
  </ion-view>
</ion-nav-view>
```

### API
|	属性	|	类型	|	详细说明
|:-----:|:--------------:|:------------------:
|	side	|	sting	|	放置在ionNavBar的方向。'left' or 'right'.

## ion-nav-back-button 
Child of `ionNavBar`

在`ionNavBar`中创建返回的按钮.

当用户可以返回的时候会显示出来。

默认的，点击会返回。如果你希望有更多高级的功能，请看下面的例子。

### 用法

默认的点击行为
```html
<ion-nav-bar>
  <ion-nav-back-button class="button-clear">
    <i class="ion-arrow-left-c"></i> Back
  </ion-nav-back-button>
</ion-nav-bar>
```

自定义点击行为，使用`$ionicNavBarDelegate`
```html
<ion-nav-bar ng-controller="MyCtrl">
  <ion-nav-back-button class="button-clear"
    ng-click="goBack()">
    <i class="ion-arrow-left-c"></i> Back
  </ion-nav-back-button>
</ion-nav-bar>
```
```javascript
function MyCtrl($scope, $ionicNavBarDelegate) {
  $scope.goBack = function() {
    $ionicNavBarDelegate.back();
  };
}
```

在button上显示之前的标题，再次使用`$ionicNavBarDelegate`
```html
<ion-nav-bar ng-controller="MyCtrl">
  <ion-nav-back-button class="button-icon">
    <i class="icon ion-arrow-left-c"></i>{{getPreviousTitle() || 'Back'}}
  </ion-nav-back-button>
</ion-nav-bar>
```
```javascript
function MyCtrl($scope, $ionicNavBarDelegate) {
  $scope.getPreviousTitle = function() {
    return $ionicNavBarDelegate.getPreviousTitle();
  };
}
```

## nav-clear

nav-clear作为属性的指示器，应该使用在使视图发生变化的元素上，例如<a href>或者<button ui-sref>.

当nav-clear作用的元素点击时，会 disable the next view transition. This directive is useful, for example, for links within a sideMenu.

### 用法
下面是一个在side menu中的添加了nav-clear的链接，点击的时候会禁用原来的动画效果
```html
<a nav-clear menu-close href="#/home" class="item">Home</a>
```

## $ionicNavBarDelegate
Delegate for controlling the `ionNavBar` directive.

### 用法
```html
<body ng-controller="MyCtrl">
  <ion-nav-bar>
    <button ng-click="setNavTitle('banana')">
      Set title to banana!
    </button>
  </ion-nav-bar>
</body>
```
```javascript
function MyCtrl($scope, $ionicNavBarDelegate) {
  $scope.setNavTitle = function(title) {
    $ionicNavBarDelegate.setTitle(title);
  }
}
```

### 方法

#### back([event])
在视图历史中返回

|	参数名	|	类型	|	详细信息
|:-----------:|:-------:|:-----------:
|	event(可选)	|	DOMEvent	|	The event object (eg from a tap event)

#### align([direction])
设置button中title的方向

|	参数名	|	类型	|	详细信息
|:-----------:|:-------:|:-----------:
|	direction(可选)	|	string	|	'left', 'right', 'center'. 默认: 'center'.

#### showBackButton([show])
设置/获取`ionNavBackButton`是否显示

|	参数名	|	类型	|	详细信息
|:-----------:|:-------:|:-----------:
|	show(可选)	|	boolean	|	是否显示返回按钮

*	返回:`boolean`.是否显示返回按钮

#### showBar(show)
细节同上

#### setTitle(title)
设置`ionNavBar`的标题

#### changeTitle(title, direction)
Change the title, transitioning the new title in and the old one out in a given direction.

|	参数名	|	类型	|	详细信息
|:-----------:|:-------:|:-----------:
|	title	|	string	|	The new title to show.
|	direction	|	string	|	The direction to transition the new title in. Available: 'forward', 'back'.

#### getTitle()
*	返回:`string`.当前navbar的标题.

#### getPreviousTitle()
*	返回:`string`.前一个navbar的标题.

#### $getByHandle(handle)

|	参数名	|	类型	|	详细信息
|:-----------:|:-------:|:-----------:
|	handle	|	string	|	

*	返回:`delegateInstance`Returns:  A delegate instance that controls only the ionTabs directives with delegate-handle matching the given handle.

示例: $ionicNavBarDelegate.$getByHandle('myHandle').setTitle('newTitle')

<!-- more -->