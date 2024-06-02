### flutter基建--动画

Flutter中也对动画进行了抽象，主要涉及 Animation、Curve、Controller、Tween这四个角色，它们一起配合来完成一个完整动画;

##### Controller

```
AnimationController,用于控制动画播放，停止，还原，重置等操作，如下代码：
_controller.forward();//播放
_controller.stop();//停止
_controller.reverse();//还原动画--恢复成原始状态，此阶段有动画
_controller.reset();//重置--恢复成原始状态，此阶段无动画

AnimationController构造器中的几种参数注意：
AnimationController({
      double? value,
      this.duration,//动画时长
      this.reverseDuration,//还原动画的时长
      this.debugLabel,
      this.lowerBound = 0.0,//生成数字区间的最小值
      this.upperBound = 1.0,//生成数字区间的最大值
      this.animationBehavior = AnimationBehavior.normal,
      required TickerProvider vsync,
    })
AnimationController 默认的生成的数字区间是 0.0---1.0之间
```

##### Tween

```dart
默认情况下，AnimationController对象值的范围是[0.0，1.0]。如果我们需要构建UI的动画值在不同的范围或不同的数据类型，则可以使用Tween来添加映射以生成不同的范围或数据类型的值。如下
Tween<double>(begin: 50, end: 100)//定义了50--100区间的值范围

Tween 继承自抽象类 Animatable<T> 主要定义动画值的映射规则
要使用Tween,需要调用其 animate()函数，然后传入一个控制对象，如下：
AnimationController _controller = AnimationController(duration: const Duration(milliseconds: 2000), vsync: this);  
Animation<double> _animation = Tween<double>(begin: 50, end: 100).animate(_controller)
代码中，Tween 便是在 2 秒内生成了 50--100的数值
```

##### Animation

```
接Tween的使用返回了一个Animation，
So, Animation 是一个抽象类，本身和UI渲染无关，它主要的作用是保存动画的插值和状态，其中常用的有Animation<Double>
故，Animation是一段时间内依次生成一段区间(Tween)之间的类
通过Animation 可以得到：Animation.value 变化的值，Animation.status,还可以添加值变化的监听和状态变化的监听如下代码：
AnimationController _controller = AnimationController(duration: const Duration(milliseconds: 2000), vsync: this);  
Animation<double> _animation = Tween<double>(begin: 50, end: 100).animate(_controller);
_animation
..addListener(() {//数值变化的监听
        setState(() {}); // 更新UI
  })
..addStatusListener((status) {//状态变化的监听
        debugPrint('status：$status'); // 监听动画执行状态
  });
```

##### Curve

```
Curve翻译成中文是：曲线，
前面讲到Tween 映射不同范围区间的数据值的类, AnimationController定义一定的时间，那么组合起来就是定义了一定时间内映射不同范围区间的数值，这个数值的变化可匀速，加速度，先加速后减速等等，Curve便是定义这个曲线的，如下
```

| Curves曲线 | 动画过程                     |
| ---------- | ---------------------------- |
| linear     | 匀速的                       |
| decelerate | 匀减速                       |
| ease       | 开始加速，后面减速           |
| easeIn     | 开始慢，后面快               |
| easeOut    | 开始快，后面慢               |
| easeInOut  | 开始慢，然后加速，最后再减速 |

```
类似Tween的使用，代码如下：
AnimationController _controller = AnimationController(duration: const Duration(milliseconds: 2000), vsync: this);  
Animation<double> _animation = CurvedAnimation(parent: _controller, curve: Curves.linear)；
_animation
..addListener(() {//数值变化的监听
        setState(() {}); // 更新UI
  })
..addStatusListener((status) {//状态变化的监听
        debugPrint('status：$status'); // 监听动画执行状态
  });
这里定义了在2秒内匀速变化的0--1之间的数值，那么使用时候，便可根据此区间的数据进行自定义使用
```

##### 实现动画的不同方式

方式一: 通过 Animation添加监听每一帧变化，进行setState刷新UI

```
_controller = AnimationController(duration: const Duration(milliseconds: 2000), vsync: this);
_animation = Tween<double>(begin: 50, end: 100).animate(_controller)
..addListener(() {
   print("value=======>${_animation.value}");
     setState(() {}); // 更新UI
})
..addStatusListener((status) {
     debugPrint('status：$status'); // 监听动画执行状态
});

widget：
Icon(Icons.ac_unit,size: _animation.value)
///缺点：每一个动画实现起来比较繁琐，假设UI内有N多动画，那么代码会很不优雅，且会有多次setState刷新
```

方式二：通过AnimatedWidget进行简化

```
class AnimatedImage extends AnimatedWidget {
  const AnimatedImage({
    super.key,
    required Animation<double> animation,
  }) : super(listenable: animation);

  @override
  Widget build(BuildContext context) {
    final animation = listenable as Animation<double>;
    return Center(
      child: Icon(Icons.ac_unit, size: animation.value),
    );
  }
}

widget:
AnimatedImage(animation: _animation),
//上述代码，将路由中的widget，迁移出独立封装，避免了在路由中进行监听并进行不断setState地刷新操作
//查看 AnimatedWidget 源码，其实是在内部实现了监听+setState刷新机制，和方式一实现是一毛一样的
//缺点，如果每一个动画，都要独立封装的话，其实还可以再次优化
```

##### 方式三：通过 AnimatedBuilder

```dart
AnimatedBuilder(animation: _animation,
   builder: (context, child) {
      return SizedBox(
        width: _animation.value,
        height: _animation.value,
        child: Icon(Icons.ac_unit, size: _animation.value));
})
//如此算是比较优化的解决了动画的视线问题
```

##### 扩展

Flutter中正是通过这种方式封装了很多动画，如：FadeTransition、ScaleTransition、SizeTransition等，很多时候都可以复用这些预置的过渡类。





[Flutter 动画（显式动画、隐式动画、Hero动画、页面转场动画、交错动画） - 掘金 (juejin.cn)](https://juejin.cn/post/7335653891943858212?searchId=20240530142030B6CFC2C13B8D1E33081E#heading-2)
