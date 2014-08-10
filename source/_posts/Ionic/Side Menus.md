title: Side Menus
tags:
  - ionic
  - side menus

categories:
  - ionic

date: 2014-08-10 18:24:51

---

## ion-side-menus 
Delegate: `$ionicSideMenuDelegate`

包含侧滑菜单和主内容的容器。菜单可位于左边或右边，并且允许通过拖拽主内容来切换。

为了自动关闭菜单，您可以通过添加`menuClose`属性。`menu-close`通常被添加在`ion-side-menu`内容中links或button上，这样就可以通过点击对应的元素自动关闭侧滑菜单。

更多信息请查看
*	`ionSideMenuContent`
*	`ionSideMenu`
*	`menuClose`

### 用法
使用侧滑菜单，需要添加父元素`<ion-side-menus>`，`<ion-side-menu-content>`作为中心内容区域，还有一个或多个`<ion-side-menu>`

```html
<ion-side-menus>
  <!-- Center content -->
  <ion-side-menu-content ng-controller="ContentController">
  </ion-side-menu-content>

  <!-- Left menu -->
  <ion-side-menu side="left">
  </ion-side-menu>

  <!-- Right menu -->
  <ion-side-menu side="right">
  </ion-side-menu>
</ion-side-menus>
```

```javascript
function ContentController($scope, $ionicSideMenuDelegate) {
  $scope.toggleLeft = function() {
    $ionicSideMenuDelegate.toggleLeft();
  };
}
```

### API
|	属性	|	类型	|	详细说明
|:-----:|:--------------:|:------------------:
|	delegate-handle(可选)	|	string	|	The handle used to identify this side menu with `$ionicSideMenuDelegate`

## ion-side-menu-content 
Child of `ionSideMenus`

主要可见内容区域的容器，与`ionSideMenu`平级

### 用法

```html
<ion-side-menu-content
  edge-drag-threshold="true"
  drag-content="true">
</ion-side-menu-content>
```

更加完整的例子，请看`ionSideMenus`

### API
|	属性	|	类型	|	详细说明
|:-----:|:--------------:|:------------------:
|	drag-content	|	boolean	|	内容区域是否可以被拖拽，默认为true
|	edge-drag-threshold	|	boolean,number	|	内容区域的拖拽是否仅在小于屏幕边距一定阈值的范围内有效。默认为false。
接受三种类型的数值	
*	如果给定一个非零的数字，距离屏幕边距指定的像素允许拖拽
*	如果给定true，则默认25像素
*	如果给定false或者0，边距拖拽阈值无效，可以从内容区域的任何区域拖拽

## ion-side-menu 
Child of `ionSideMenus`

侧滑菜单的容器，与`ionSideMenuContent`平级。

### 用法

```html
<ion-side-menu
  side="left"
  width="myWidthValue + 20"
  is-enabled="shouldLeftSideMenuBeEnabled()">
</ion-side-menu>
```

更加完整的例子，请看`ionSideMenus`

### API
|	属性	|	类型	|	详细说明
|:-----:|:--------------:|:------------------:
|	side	|	string	|	菜单的位置，left或者right
|	is-enabled(可选)	|	boolean	|	菜单是否可用
|	width(可选)	|	number	|	菜单的宽度，默认275

## menu-toggle
切换给定边的侧滑菜单

### 用法
下方的示例是在nav bar中link。点击link就会自动打开给定的侧滑菜单
```html
<ion-view>
  <ion-nav-buttons side="left">
   <button menu-toggle="left" class="button button-icon icon ion-navicon"></button>
  </ion-nav-buttons>
 ...
</ion-view>
```

## menu-close
关闭当前的侧滑菜单

### 用法
下面的实例是在side menu中的link中，点击这个link则会自动关闭当前打开的菜单
```html
<a menu-close href="#/home" class="item">Home</a>
```

## $ionicSideMenuDelegate
Delegate for controlling the `ionSideMenus` directive.

通过`ionicSideMenuDelegate`服务调用的函数会控制所有的side menus。通过调用`$getByHandle`获取`ionSideMenus`实例。

### 用法
```html
<body ng-controller="MainCtrl">
  <ion-side-menus>
    <ion-side-menu-content>
      Content!
      <button ng-click="toggleLeftSideMenu()">
        Toggle Left Side Menu
      </button>
    </ion-side-menu-content>
    <ion-side-menu side="left">
      Left Menu!
    <ion-side-menu>
  </ion-side-menus>
</body>
```
```javascript
function MainCtrl($scope, $ionicSideMenuDelegate) {
  $scope.toggleLeftSideMenu = function() {
    $ionicSideMenuDelegate.toggleLeft();
  };
}
```

### 方法
#### toggleLeft([isOpen])
如果存在左侧滑菜单，则切换。

|	参数名	|	类型	|	详细信息
|:-----------:|:-------:|:-----------:
|	isOpen(可选)	|	boolean	|	是否打开或关闭菜单，默认切换菜单。

#### toggleRight([isOpen])
用法同`toggleLeft([isOpen])`

#### getOpenRatio()
返回已打开菜单的宽度占菜单宽度的比例。

*	返回：`float `。没有任何打开则为0；左侧菜单打开或打开中为0至1之间，右侧淡菜打开或打开中为0至-1之间

#### isOpen()
*	返回：`boolean`。左侧或右侧菜单处于打开状态

#### isOpenLeft()
*	返回：`boolean`。左侧菜单处于打开状态

#### isOpenRight()
*	返回：`boolean`。右侧菜单处于打开状态

#### canDragContent([canDrag])

|	参数名	|	类型	|	详细信息
|:-----------:|:-------:|:-----------:
|	canDrag(可选)	|	boolean	|	设置是否可以通过拖拽打开侧滑菜单
*	返回：`boolean`。是否可以通过拖拽打开侧滑菜单

#### edgeDragThreshold(value)

|	参数名	|	类型	|	详细信息
|:-----------:|:-------:|:-----------:
|	value	|	boolean,number	|	
*	返回：`boolean`。是否可以在屏幕边距阈值内使用拖拽

#### $getByHandle(handle)

|	参数名	|	类型	|	详细信息
|:-----------:|:-------:|:-----------:
|	handle	|	string	|	

*	返回:`delegateInstance`Returns:  A delegate instance that controls only the ionTabs directives with delegate-handle matching the given handle.

示例: $ionicSideMenuDelegate.$getByHandle('my-handle').toggleLeft();


<!-- more -->