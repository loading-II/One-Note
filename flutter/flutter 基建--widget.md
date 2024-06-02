### Widget

在Flutter 的世界里 Everything is Widget，widget的概念广泛，不仅表示UI元素，也可以表示功能组件，主题数据的Theme等等

Widget并非是最后绘制在屏幕上的显示元素，它是‘描述一个UI元素的配置信息’，在使用Widget的所有的参数都只是配置信息的一部分，那么谁才是真正的界面显示的元素呢，下面看一下widget的生命周期

```
1、createState: 在StatefulWidget 对应的 StatefulElement的构造器中，实现了 StatefulWidget 所对应的 State的创建工作，也就是 createState的调用
2、initState：初始化State，StatefulElement继承自ComponentElement，在ComponentElement 进行mount的函数中，会进行FirstBuild的调用，那么此次firstBuild就实现了 initState和 didChangeDependencies
3、didChangeDependencies: 意思是如果在builder前，使用到了 inheritedWidget，那么当inheritedWidget 中发生了变化，那么在下次builder的时候，inheritedWidget的子widget 就会回调 didChangeDependencies
4、Build：用户构建widget 的子树
5、reassemble: 次回调在开发模式下会用到，那么release模式下永远不会用到：在热更新的时候会用到；
6、didUpdateWidget: 在Widget中有个静态函数canUpdate用于构建Widget的时候，判断是是否新建Element，如果canUpdate返回true则会调用didUpdateWidget,意味着：使用新Widget配置直接升级Element的配置，不会新建Element
7、deactive：当 State 对象从树中被移除时，会调用此回调；
8、dispose：意味着State对象从树种永久移除，也就是释放资源；

当我们打开新路由的生周期变化：createState-->initState-->didChangeDependencies-->build
当移除某个widget生命周期变化：reassemble-->deactive-->dispose
当setState刷新路生命周期变化：reassemble-->didUpdateWidget-->build

通过源码可知，Widget的生命周期的每一次回调都是由 Element 调用产生，也就是Element才是核心逻辑的处理部分 
```

##### State

一个StatefulWidget对应一个State对象，State中保存着StatefulWidget的状态信息，State中有两个重要的属性：widget 和 Context

每次重新构建，widget是新建立的对象，但是state.widget都会指向对应的新的widget

问题1：在widget中，如何获取对应的State对象

方式一：通过Context，Context 提供了一个函数： findAncstorStateOfType() 函数，该方法可以通过widget树向上查看指定泛型的State对象，示例

```
Scaffold(
      appBar: AppBar(
        title: Text("子树中获取State对象"),
      ),
      body: Center(
        child: Column(
          children: [
            Builder(builder: (context) {
              return ElevatedButton(
                onPressed: () {
                  // 向上查找父级最近的Scaffold对应的ScaffoldState对象
                  ScaffoldState _state = context.findAncestorStateOfType<ScaffoldState>()!;
                  // 打开抽屉菜单
                  _state.openDrawer();
                },
                child: Text('打开抽屉菜单1'),
              );
            }),
          ],
        ),
      ),
      drawer: Drawer(),
    );
```

方式二：通过GlobalKey，给与目标对象设定GlobalKey，然后根据GlobalKey.currentState;

GlobalKey，是flutter 提供的一种在整个APP中引用Element的机制，可以快速获取globalKey.widget he  element ,但是GlobalKey开销较大，且保证期唯一性，故应该尽量使用其他方式实现

##### Element

###### widget 和Element之间的关系：

Widget是为Element提供配置清单的存在，WIdget的引用被Element持有，最终Widget的生命周期都是被Element在不同时期调用的

生命周期：

1、framework 层调用Widget.createElement，创建Element示例，记为Element

2、



##### Flutter 四棵树

flutter框架的处理流程是如下：

1. 根据 Widget 树生成一个 Element 树，Element 树中的节点都继承自 `Element` 类。
2. 根据 Element 树生成 Render 树（渲染树），渲染树中的节点都继承自`RenderObject` 类。
3. 根据渲染树生成 Layer 树，然后上屏显示，Layer 树中的节点都继承自 `Layer` 类。

真正的布局和渲染逻辑在 Render 树中，Element 是 Widget 和 RenderObject 的粘合剂，可以理解为一个中间代理。



```

```



