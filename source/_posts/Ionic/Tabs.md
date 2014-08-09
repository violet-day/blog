title: ionic-Tabs
tags:
-	ionic
-	Tabs

categories:
- ionic

date: 2014-08-10 04:59:23

---

## ion-tabs
Delegate：`$ionicTabsDelegate`

提供多tab切换的接口。

可以任意设置`tabs class`和`animation class`来定义效果。

单个的tab细节详见`ionTab`指示器

注意: 不要再ion-content添加ion-tabs，这样会造成css错误

### 用法
```html
<ion-tabs class="tabs-positive tabs-icon-only">

  <ion-tab title="Home" icon-on="ion-ios7-filing" icon-off="ion-ios7-filing-outline">
    <!-- Tab 1 content -->
  </ion-tab>

  <ion-tab title="About" icon-on="ion-ios7-clock" icon-off="ion-ios7-clock-outline">
    <!-- Tab 2 content -->
  </ion-tab>

  <ion-tab title="Settings" icon-on="ion-ios7-gear" icon-off="ion-ios7-gear-outline">
    <!-- Tab 3 content -->
  </ion-tab>

</ion-tabs>
```

## ion-tab

Child of ionTabs

包含了tab的content。content只有当指定的tab被选中时才出现。

每个ionTab都有自己的view history

### 用法
```html
<ion-tab
  title="Tab!"
  icon="my-icon"
  href="#/tab/tab-link"
  on-select="onTabSelected()"
  on-deselect="onTabDeselected()">
</ion-tab>
```

更加完整的例子请看`inoTabs`

### API

|	属性	|	类型	|	详细说明
|:-----:|:--------------:|:------------------:
|	title	|	string	|	tab的标题	
|	href(可选)	|	string	|	The link that this tab will navigate to when tapped
|	icon(可选)	|	string	|	tab的图标，如果设置，则会变为icon-on和icon-off的默认图标
|	icon-on(可选)	|	string	|	选中时tab的图标
|	icon-off(可选)	|	string	|	未选中时tab的图标
|	badge(可选)	|	expression	|	tab上的徽章，通常为数字
|	badge-style(可选)	|	expression	|	tab上徽章的样式，tabs-positive等
|	on-select(可选)	|	expression	|	tab选中的回调函数
|	on-deselect(可选)	|	expression	|	tab取消选中时的回调
|	ng-click(可选)	|	expression	|	默认的，tab将会在click之后被选中。如果设置了ngClick，将不会被选中。你可以显示的通过`$ionicTabsDelegate.select()`切换

## $ionicTabsDelegate

Delegate for controlling the `ionTabs` directive.

`$ionicTabsDelegate`服务中的函数控制所有的`ionTabs`指示器，通过`$getByHandle`获取具体的`ionTabs`实例。

### 用法
```html
<body ng-controller="MyCtrl">
  <ion-tabs>

    <ion-tab title="Tab 1">
      Hello tab 1!
      <button ng-click="selectTabWithIndex(1)">Select tab 2!</button>
    </ion-tab>
    <ion-tab title="Tab 2">Hello tab 2!</ion-tab>

  </ion-tabs>
</body>
```

```javascript
function MyCtrl($scope, $ionicTabsDelegate) {
  $scope.selectTabWithIndex = function(index) {
    $ionicTabsDelegate.select(index);
  }
}
```

### 方法

#### ```select(index, [shouldChangeHistory])```

通过给定的index选中tab

|	参数名	|	类型	|	详细信息
|:-----------:|:-------:|:-----------:
|	index	|	number	|	选中tab的缩影
|	shouldChangeHistory(可选)	|	boolean	|	是否此次选中载入指定tab的历史，或者仅仅载入默认的页面。默认为false。提示：如果tab中有ionNavView，这个很可能为true

#### `selectedIndex`

*	返回：`number`,选中tab的索引，没有则为-1

#### `$getByHandle(handle)`

|	参数名	|	类型	|	详细信息
|:-----------:|:-------:|:-----------:
|	handle	|	string	|	

*	返回:`delegateInstance`Returns:  A delegate instance that controls only the ionTabs directives with delegate-handle matching the given handle.

示例: $ionicTabsDelegate.$getByHandle('my-handle').select(0);

<!-- more -->