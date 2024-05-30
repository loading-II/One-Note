###路由管理

flutter 中的路由(route) 等同于 Android 中的Activity，等同于WEB单页面中的route概念，等同于IOS中的 ViewController的概念，所谓的路由管理实际上就是页面之间的跳转管理，也称之为导航管理，和移动端的导航管理类似，都为维护一个路由栈，入栈，出栈的操作。

Navigator: 路由管理组件，通过一个栈管理活动的路由集合，常用方法如下：

#####1、动态路由：无需注册直接跳转

```dart
1、将给定的路由入栈，打开新界面：
  Future push(BuildContext context, Route route)
  示例：
  Navigator.push(context, MaterialPageRoute(builder: (BuildContext context) {
      return LoginPage(title: 'LoginPage');
  }))
2、将给定的路由出栈，关闭界面，返回到上一个界面：
  bool pop(BuildContext context, [ result ])  ///result 是可选参数，即返回给上一个界面的值
  示例：
  Navigator.pop(context, {'result': 'Back Value = 1'});
3、替换当前界面，并且动画完成之后，dispose释放当前界面，一般此种push方式经常用于  闪屏页 打开首页的情况
  Future<T?> pushReplacement(BuildContext context, Route<T> newRoute, { TO? result })
  注意这里有三个参数：context 上下文，newRoute 即将打开的路由，result 返回给被替换的界面的上一个界面的值
  例如：
  1、page1  push 进入 page2, page1 监听push结果；
  2、page2 通过 pushReplacement 进入 page3, 
	3、按照如下示例，page1 便能监听到返回的数据
  示例：
	Navigator.pushReplacement(context, MaterialPageRoute(builder: (BuildContext context) {
      return LoginPage(title: 'LoginPage');
  }),result: "back-pre-page-data")
4、pushAndRemoveUntil 入栈指定界面，然后依次退出栈内路由，直到 判定条件为true
  例如：
  1、page1、page2、page3、page4 依次push打开，然后通过 pushAndRemoveUntil 打开 page5
    如下代码：
    Navigator.pushAndRemoveUntil(context, MaterialPageRoute(builder: (BuildContext context) {
            return Page5(title: '');
    }), (route) => route.settings.name == '/')
  那么将打开Page5 后，依次释放page4、page3、page2、直到 page1 因为page1 默认路由Name是 '/'
  如果，将 (route) => route.settings.name == '/' 更换成 (route) => false ，将不会释放任何 Page页面
```

#####2、静态路由：需要注册的路由

2.1、路由注册

```
void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: NavigatorPage(),
      routes: <String, WidgetBuilder>{
        '/router/first': (BuildContext context) => FirstPage(),
        '/router/second': (BuildContext context) => SecondPage()
      },
    );
  }
}

其上的routes 对应的 map 便是注册了静态路由
```

2.2、静态路由跳转

```
类似动态路由的跳转: pushNamed、pushReplacementNamed、pushNamedAndRemoveUntil
分别对应动态路由中的 push、pushReplacement、pushAndRemoveUntil
稍有不同的是多一个可选参数：Object? arguments 用于路由之间的传递参数 
因为动态路由详细介绍，这里不再具体介绍静态路由的细节了
```

#####3、路由间传参

```dart
1、动态路由传参，直接使用 route 的构造函数即可
Navigator.push(context, MaterialPageRoute(builder: (BuildContext context) {
      return LoginPage(title: 'LoginPage');
  }))
代码中的 title 便是传递了一个参数，如果想传递更多参数，可以增加更多，或者设定map类型参数: 
final Map<dynamic,dynamic>? argums;

2、静态路由传参，新版本中增加了传参：Object? arguments
示例：
入栈：
Navigator.pushReplacementNamed(context, RouteConfiguration.homePage, arguments: {'name': 'Guoql', 'age': 123})
接收解析参数：
final args = ModalRoute.of(context)?.settings.arguments;
```

#####4、路由钩子

```
路由钩子，适合的场景是 登录拦截又或者其他XX拦截，看下面代码
MaterialApp(
        ///1、注册路由表
        routes: RouteConfiguration.routeMap,
        ///2、当未找到指定name的路由触发
        onUnknownRoute: (RouteSettings setting) {
          if (setting != null) {
            String? name = setting.name;
            return MaterialPageRoute(builder: (context) {
              return const ErrorPage(title: "没有匹配的页面");
            });
          }
        },
        ///3、当未注册路由表的时候，触发路由钩子
        onGenerateRoute: (RouteSettings settings) {
          return CupertinoPageRoute(
            settings: settings,
            builder: (context) {
              String? name = settings.name;
              if (name == null) {
                name = RouteConfiguration.splashPage;
              } else if (name == RouteConfiguration.rootPage) {
                name = RouteConfiguration.splashPage;
              }
              Widget widget = RouteConfiguration.routeMap[name]!(context);
              return widget;
            },
          );
        },
)
MaterialApp 有onGenerateRoute 属性，当通过静态路由打开指定Name的路由时，如果指定的Name未在路由表中注册，此时变会触发 onGenerateRoute 那么变给与我们一个统一管理路由的钩子
故建议使用路由钩子代替路由表，进行统一管理
```

#####5、路由切换动画

```
切换动画，可以代来用户体验的提升
动态路由切换动画实现：
1、MaterialPageRoute：实现与'平台'页面切换动画风格一致的路由切换动画，对于安卓是底部向上入场，从顶部向下退出的效果
2、CupertinoPageRoute：是Cupertino组件库提供的iOS风格的路由切换组件，即在Android上也能实现左右切换风格
3、自定义路由切换动画
   仔细查看 MaterialPageRoute 和 CupertinoPageRoute 都是继承自 PageRoute 实现的既定动画效果，
   flutter 提供了 PageRouteBuilder 同样继承自 PageRoute，提供了对应动画时长和动画效果的入口，如下：
   PageRouteBuilder(
    transitionDuration: Duration(milliseconds: 500), //动画时间为500毫秒
    pageBuilder: (BuildContext context, Animation animation,
        Animation secondaryAnimation) {
      return FadeTransition(
        //使用渐隐渐入过渡,
        opacity: animation,
        child: PageB(), //路由B
      );
    },
  ),
  
  PageRouteBuilder(
      pageBuilder: (context, animation, secondaryAnimation) => (nextPage),
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        var begin = const Offset(1.0, 0.0);//左进右出的动画效果
        const end = Offset.zero;
        const curve = Curves.ease;
        var tween = Tween(begin: begin, end: end).chain(CurveTween(curve: curve));
        return SlideTransition(
          position: animation.drive(tween),
          child: child,
        );
      },
    )
PageRouteBuilder 是 PageRoute 的一个包装，我们还可以直接继承 PageRoute 进行定义你需要的动画效果

```

