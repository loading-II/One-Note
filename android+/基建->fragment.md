基建->broadcast

[toc]

简介：Fragment 是Activity内的碎片，是用于分担Activity试图管理的一个角色

###### 1、缘起

```
Android3.0时开始增加了Fragment的角色，是用于适配平板的横竖屏布局而增加的，到了Android4.0的时候Fragment适应于手机和平板的设备，故Fragment的缘起是为了适配横竖屏布局的设置产生的；
```

###### 2、生命周期

```
Fragment是依托于Activity存在的，故它的生命周期和Activity的生命周期有密切关系：
onCreate阶段可分为：onAttach、onCreate、onCreateView、onViewCreated、onActivityCreated
onStart--->onStart
onResume--->onResume
onPause--->onPause
onStop--->onStop
onDestory阶段：onDestoryView、onDestory、onDtach
可见Fragment的生命周期和Activity类似，但是在onCreate和onDestory阶段，有所不同，因为Fragment是专注于视图管理，所以对View的创建和销毁细节更加明确
```

###### 3、重叠现象

```
先说什么是重叠现象：
3.1、Activity内通过fragmentManager开启事务对Fragment进行：add、remove、replace、show、hide等操作以此实现对Fragment的管理效果
3.2、假设在Activity的onCreate中动态add显示一个Fragment，那么当Activity因为配置变化实现重建的时候，此次Fragment就会产生重叠现象
3.3、这是因为Activity在重建的时候，默认恢复了Fragment，但是呢，由于开发者对重建恢复的机制了解不全，在onCreate中没有进行判断却比，导致了add连个Fragment，因此就出现了重叠的现象
```

###### 3、Fragment的返回栈

```
Fragment是专注于视图管理，占用内存小，切换速度快，有一些APP采用的是单Activity的架构设计，因为Activity不只是视图管理还涉及视图间的跳转，涉及AMS，WMS等等相对于Fragment是一个很重的设计；
就是因为Fragment专注于视图部分，所以他的生命周也无非是视图的创建状态等等，之所以它的设计也只有返回栈无其他任务栈之类的
使用的时候通过FragmentManager开启事务，然后调用 addToBackStack 即可添加到返回栈里面
```

###### 4、为何推荐DialogFragment而非Dialog

```
原因是DialogFragment有重建机制，遇到类似横竖屏切换的情况，Activity销毁重建后，对应的DialogFragment也能跟随者重建
如果是单纯Dialog则无法做到此现象
```

###### 5、为何试图控制建议Fragment而非Activity

```
5.1、在混淆的情况，Fragment可被混淆，但是Activity 通过反编译软件后，一目了然
5.2、Fragment专注于视图管理，相对于Activity轻很多，且切换速度很快，很适合进行单Activity的架构设计
5.3、因为Fragment的设计专注于视图管理，非组件，职责少，占资源也小，切换速度是Activity的5--10倍，故非常适合进行视图管理
```

###### 6、Fragment和Activity之间的通信

```
Activity调用Fragment的方式：
1、如果Activity 持有Fragment的对象，那么可以直接通过Fragment对象调用其内部的所有public函数
2、通过FindFragmentByID或者byTag获取对应的Fragment，然后按照持方式一进行调用
3、Activity 在打开Fragment的时候，可以通过setArgument的方式，传入参数
Fragment调用Activity的方式：
1、通过getActivity 获取Fragment当前绑定的Activity实例，以此调用Activity内部的函数

上面都是通过直接的调用函数的形式，进行通信，下面还可以使用
1、通过设置监听进行接口回调的方式进行通信，
2、通过handler进行通信
3、通过EventBus、通过共享ViewModel内的LiveData等方式进行通信，均可
```

