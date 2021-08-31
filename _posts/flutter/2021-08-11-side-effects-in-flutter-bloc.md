---
title: 如何处理flutter-bloc 中的side-effect
tags: flutter flutter-bloc bloc
layout: page
---



如何处理flutter-bloc 中的side-effect。
(https://github.com/felangel/bloc/issues/812)[https://qiaojun2016.github.io/blog/index.html]

问题描述: 在 BlocListener 中处理类似于弹框的side-effect 会覆盖掉数据状态(Loaded)中已经加载的数据。

**不是所有的事情都要数据驱动变化**

例如你要导航到一个新的页面，你或许会想到，啊 改变页面状态 然后在BlocListener里面监听这个状态变化，然后导航到该页面。这就有点脱裤子放屁感觉。

但是是否跳转也是有条件的，如果按下按钮一定会跳转，我何必多次一举呢

你真的没有必要多此一举

```dart
confirmDIsmiss: (direction)async {
    if (direction == DismissDirection.startToEnd) {
        // No logic so just navigate directly...no need for a bloc
        // 跳转到新页面
        return false;
    }
    // 做其他的是
}
```

这种side-effects 一般发生在我需要 但是真实的数据状态并没有发生变化。
**案例**
购物车列表上每个item 都显示一个数值，用户点击这部分，会弹出一个输入框来输入信息。那么为了显示输入框。那么这种情况下，我直接显示输入框就好了吗？ 如果用户做了一些其他操作那么输入框是不会弹起的。

**解决方案**
现在我们为每个Item准备一个ItemBloc的东西。那么这个每个ItemBloc 就会仅仅持有当前Item的数据。当List创建Item时候就会创建新的ItemBloc。当需要通知ListBloc时候我们只需要发射一个GroupChanged的Event即可。

``` dart
abstract class CartItemState {
}
```

建议 

UI中状态做好不是具体的与UI相关的操作【**SnackBarState**，**InputDialogState**】要保持状态的一般性，他描述的应该是UseCase处理结果，当状态互斥时候，用子类来区分这些状态；因为你不可能同时成功和失败。但是如果你就是需要同时存在，即遇到两种状态叠加在一起，那么就把这两种状态合并成一个状态。
```dart
class WeatherState {
    final Weather weather;
    final bool hasError;
}
```
然后你就可以利用 `copyWith` 方法发射这个状态 `state.copyWith(hasError: true)` 这样你就会保持你之前的数据不会丢失了。





