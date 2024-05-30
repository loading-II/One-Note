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

##### 扩展：

```
flutter 内置提供了一些显示动画组件，比如缩放动画组件，替换上面的实现如下，无需在进行setState刷新界面了
ScaleTransition(
              scale: _animation,
              child: const SizedBox(
                width: 50,
                height: 50,
                child: Icon(
                  Icons.ac_unit,
                  size: 50,
                ),
              ),
)
类似 ScaleTransition 还有FadeTransition、AlignTransition、SizeTransition 以及 AnimatedBuilder、
```



[Flutter 动画（显式动画、隐式动画、Hero动画、页面转场动画、交错动画） - 掘金 (juejin.cn)](https://juejin.cn/post/7335653891943858212?searchId=20240530142030B6CFC2C13B8D1E33081E#heading-2)
