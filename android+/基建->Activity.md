Android基建--四大组件

[toc]

简介：生命周期、启动模式、瞬时数据保存复现、横竖屏销毁重建、TaskAffinity亲合度、启动Flag

###### 1、首先离不开生命周期：onCreate、onStart、onResume、onPause、onStop、onDestory

```
onPause：可见但不可交互<------>onStop：不可见，不可交互；
onStart：不可见不可交互<------>onResume：可见可交互；
```



###### 2、有activity：A、B、C 三个，依次跳转打开，然后返回到A的生命周期变化

 ```java
 1、打开A执行：a.onCreate-->a.onStart-->a.onResume
 2、A 打开 B 执行：a.onPause-->b.oncreate-->b.onstart-->b.onResume-->a.onStop
 3、B 打开 C 执行：重复步骤二的操作
 4、C 界面通过物理返回按键回退的操作：c.onPause->b.onRestart->b.onStart->b.onResume->c.onStop->c.onDestory
 5、B 界面通过物理返回按键返回到 A 界面，重复步骤四的操作
 6、A 界面通过物理返回按键返回到 桌面：a.onpause->a.onStop->a.onDestory
 ```

###### 3、在activity 中，点击Home按键的生命周期变化

```
点击Home按键操作：a.onPause-->D.on……-->a.onstop
通过桌面 Icon 重新打开APP：D.onPause-->a.onReStart-->a.onStart-->a.onResume
很好理解，相当于APP 和 桌面应用之间的生命周期切换的变化
```

###### 4、onSaveInstanceState + onRestoreInstanceState

```
首先说明，此状态的调用，一般发生在特别情况的生命周期变化，比如横竖屏的切换：
1、activity默认是支持横竖屏切换的，其生命周期变化是，先销毁后重建：
t.onPause-->t.onStop-->t.onDestory----->onCreate、onStart、onResume
2、类似于横竖屏切换的这种特殊过程中，系统会调用onSaveInstanceState用于轻量数据的保存，便于界面恢复的时候，数据进行恢复，onSaveInstanceState发生在t.onStop之前，和 onPause之间如具体时序定义，界面恢复后使用 onResoreInstanceState进行数据恢复
```

###### 5、onRestoreInstanceState 和 onCreate(Bundle savedInstanceState) 恢复数据的区别

```
1、首先这个两个函数都属于生命周期的一部分，执行的时机不同
2、onRestoreInstanceState不一定会执行，只有在特别的生命周期变化的时候，才会执行
3、onCreate中的Bundle参数需要判断是否为null，而onRestoreInstanceState则不需要判断，只需要执行必定瞬时数据
```

###### 6、如何避免activity销毁重建

```
1、通过配置清单内对activity进行配置
2、APP的开发和游戏不同，可以不支持横屏切换，比如固定竖屏： android:screenOrientation="portrait"
3、支持屏幕切换，修改configChanges:
android:configChanges="orientation|screenSize"
在activity内重写 onConfigurationChanged函数，在触发屏幕切换的时候，进行监听
```

###### 7、activity的状态和优先级

```
1、状态-->onResume： running状态，正在和用户交互的状态，属于前台activity，优先级最高
2、状态-->onPause: 暂停状态，举例：有个透明非全屏的Activity，被覆盖的Activity可见但不可交互，此时便是暂停状态，优先级次之
3、状态-->onStop：后台Activity：A 打开 B，B处于前台状态，那么A便是后台Activity，优先级最低
系统内存不足，便会杀死优先级最低的进程
```

###### 8、启动模式：

```
standard: 标准模式，启动一次创建一个对象，放入Activity栈内
singleTop: 栈顶复用模式，一定是栈顶才会复用，届时会触发 onNewIntent函数，
					 此时的生命周期变化：a.onPause->a.onNewIntent->a.onResume函数
singleTask：栈内复用模式，且置为栈顶，把原有栈内其上的对象逐个销毁，最后目标Activity便存在于栈顶了
					 实例：a(singleTask)、b、c、A
					 此时的生命周期变化：
					 b.onDestory->c.onPause->a.onNewInsatnce->a.restart->a.start->a.resume->c.onStop->c.onDestory
					 具有clearTop的特性
singleInstance：单例模式，比较特殊，单开新的Activity任务栈，有区别于默认的任务栈，且单开的任务栈内唯一实例
					实例： a(默认启动模式)、b(singleInstance)
					情景一：打开a,回到桌面，通过桌面ICON打开APP，进入A 界面，没问题
					情景二：打开a,打开b,回到桌面，通过桌面ICON打开APP，进入A界面，问题来了，为什么呢？
					因为打开APP默认的是APP的任务栈，而非singleInstance的大单独任务栈，这里涉及 taskAffinity
```

###### 9、taskAffinity

```
TaskAffinity：任务亲和性；
任何一个Activity都能指明希望加入的Task栈，如果Activity没有指明要加入的TaskAffinity名称，那么就默认使用application指明的名称，如果application也没指明，那么默认就是包名；
通过命令查看任务栈Name多是包名：adb shell dumpsys activity activities

```

###### 10、另一种启动模式的Flag

```
flag_activity_new_task: 类似于 singleTask，单开一个新的任务,但不完全相同；
flag_activity_single_top: 等同于 singleTop
flag_activity_clear_top 一般多和 flag_activity_new_task 组合使用，此时等同于 singleTask
```

###### 10、Activity、window、decoreview关系

```
Activity是基于模版模式+组合模式的实现，目的是摒弃后台机制，留给开发者一个简单干净清爽的能够快速上手实现界面开发的实现子类模版；
Activity的职责边界是：包括但不限于window，它的职责对内是生命周期管理，对外是界面间的管理
```

10.1: activity.attach中实例化window的实现类phonewindow，使Activity和phonewindow以及windowManager之间产生了关联

```
//TODO: 这里可以用于解答：Activity和window产生关联的位置
// 1.创建了PhoneWindow对象，在Activity中持有了Window。
mWindow = new PhoneWindow(this, window, activityConfigCallback);//跟进window的实例化去看一看
mWindow.setWindowControllerCallback(this);
// 2.将 Activity 作为参数传递给 Window，所以在 Window 中持有了 Activity 。
mWindow.setCallback(this);
……………………省略代码……………………
//Activity持有WindowManage
mWindowManager = mWindow.getWindowManager();
```

10.2：DecorView的初始化流程

```
Activity.onCreate生命周期内，通过setConetntView(R.layout.xx) 触发DecorView的初始化工作
@Override
    public void setContentView(int layoutResID) {
        if (mContentParent == null) {
            installDecor();
        } else
        …………………………省略部分代码
    }
跟进：installDecor
private void installDecor() {
        mForceDecorInstall = false;
        if (mDecor == null) {
            mDecor = generateDecor(-1);
        }
继续跟进：generateDecor ===>实质是 new DecorView(context, featureId, this, getAttributes());
根据代码显示：DecorView 的实质是一个FrameLayout，  第三个参数 this 便是 phonewindow 意味着和持有者产生了关联
```

10.3：显示

```
前面我们了解到 
1、Activity.attach创建了 phoneWindow
2、activity.onCreate->setContentView  创建DecorView(FrameLayout) 并将对应的布局文件转换成视图树加载到 DecorView中
3、下面看如何去显示出来，具体看 ActivityThread. handleResumeActivity() 也就是 Activity.onResume之后，才是可见的原因
```

10.4：延伸一个问题，那就是为什么在onCreate 和 onStart中，无法获取View的宽高，又应该如何获取View的宽高

```
根据上面描述，attach初始化了 phonewindow，onCreate（setContent）才创建window内的decorView，根据FrameWork层源码看到，等到onResume之后，decoreView 才加载通过WindowManage才加载到Window中，并设置了可见，

好的，那么如何在onCreate 和 onStart中获取View的宽高呢？
可以通过获取 ViewTreeObsever监听布局完成，获取对应View的宽高 这是监听View布局完成的监听操作
可以通过View.post(Runnable)的方式，进行获取View的宽高
推荐第二种方式，View.post方案，实际原理还是handler机制的原理
```

10.5：延伸介绍下 ViewRootImpl，因为 View.post(Runnable) 获取宽高

```
```





###### 11、activity 的任务栈和回退栈的设计

```
```

###### 12、ActivityRecorder、TaskRecorder………………



