# Flutter PageView/TabBarView等控件保存状态的问题解决方案

PageView + BottomNavigationBar 或者 TabBarView + TabBar 的时候大家会发现当切换到另一页面的时候, 前一个页面就会被销毁, 再返回前一页时, 页面会被重建, 随之数据会重新加载, 控件会重新渲染 带来了极不好的用户体验， 跟原生的Pager 显示的效果不太一样。





#### 解决方案

**1. 官方推荐：AutomaticKeepAliveClientMixin**

由于TabBarView内部也是用的是PageView, 因此两者的解决方式相同. 下面以PageView为例



```dart
class _Test6PageState extends State<Test6Page> with AutomaticKeepAliveClientMixin {
  @override
  void initState() {
    super.initState();
    print('initState');
  }
 
  @override
  void dispose() {
    print('dispose');
    super.dispose();
  }
 
  @override
  Widget build(BuildContext context) {
    return ListView(
      children: widget.data.map((n) {
        return ListTile(
          title: Text("第${widget.pageIndex}页的第$n个条目"),
        );
      }).toList(),
    );
  }
 
 //方法返回true
  @override
  bool get wantKeepAlive => true;

```



**2. 替换PageView 使用 IndexedStack**

IndexedStack 继承 至 Stack，可以根据indexed来决定显示哪个child

```Dart
IndexedStack(
  index: currentIndex,
  children: bodyList,

 ));
```

缺点是：

- 第一次**加载时便实例化了所有的子页面State**，因此比较适合固定页面的布局
- 无法像pagerView一样通过手势左右滑动





为什么Flutter提供的PagerView 默认不实现 保存状态，而要通过 AutomaticKeepAliveClientMixin 这类来处理呢？

> Flutter中为了节约内存不会保存widget的状态，widget都是临时变量。当我们使用TabBar，TabBarView是我们就会发现，切换tab后再重新切换回上一页面，这时候tab会重新加载重新创建，体验很不友好。Flutter出于自己的设计考虑并没有延续android的[ViewPager](https://so.csdn.net/so/search?q=ViewPager&spm=1001.2101.3001.7020)这样的缓存页面设计，毕竟控件两端都要开发，目前还在beta版本有很多设计还不够完善，但是设计的拓展性没得说，flutter还是为我们提供了解决办法。我们可以强制widget不显示情况下保留状态，下回再加载时就不用重新创建了。
>
> 
> AutomaticKeepAliveClientMixin
>
> 
>
> `AutomaticKeepAliveClientMixin` 是一个抽象状态，使用也很简单，我们只需要用我们自己的状态继承这个抽象状态，并实现 `wantKeepAlive` 方法即可。
>
> 
>
> 继承这个状态后，widget在不显示之后也不会被销毁仍然保存在内存中，所以慎重使用这个方法。





### AutomaticKeepAlive

AutomaticKeepAlive 的组件的主要作用是将列表项的根 RenderObject 的 keepAlive **按需自动标记** 为 true 或 false。为了方便叙述，我们可以认为根 RenderObject 对应的组件就是列表项的根 Widget，代表整个列表项组件，同时我们将列表组件的 Viewport区域 + cacheExtent（预渲染区域）称为**加载区域** ：

1. 当 keepAlive 标记为 false 时，如果列表项滑出加载区域时，列表组件将会被销毁。
2. 当 keepAlive 标记为 true 时，当列表项滑出加载区域后，Viewport 会将列表组件缓存起来；当列表项进入加载区域时，Viewport 从先从缓存中查找是否已经缓存，如果有则直接复用，如果没有则重新创建列表项。

那么 AutomaticKeepAlive 什么时候会将列表项的 keepAlive 标记为 true 或 false 呢？答案是开发者说了算！Flutter 中实现了一套类似 C/S 的机制，AutomaticKeepAlive 就类似一个 Server，它的子组件可以是 Client，这样子组件想改变是否需要缓存的状态时就向 AutomaticKeepAlive 发一个通知消息（KeepAliveNotification），AutomaticKeepAlive 收到消息后会去更改 keepAlive 的状态，如果有必要同时做一些资源清理的工作（比如 keepAlive 从 true 变为 false 时，要释放缓存）。







# Reference

https://blog.csdn.net/jdsjlzx/article/details/126791964

https://blog.csdn.net/jdsjlzx/article/details/126792119

