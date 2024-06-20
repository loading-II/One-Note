基建->service

[toc]

简介：Service和Activity一样的重要，都是Context的子类，只不过Service没有界面而已，适合执行一些后台任务，但是Service默认运行主线程，所以还不能做耗时任务，除非在Service中开启异步现成处理

###### 1、生命周期

```
Service 和 Activity 一样都是Context 的子类，但Service没有界面，故生命周期也跟着简单了一些：
onCreate:	第一次创建会运行此方法，如果Service已经存在，将不再二次运行；
onStartCommand: 采用startService方式启动Service的时候，触发次方法，每startService一次，将触发一次；
onBind: 采用binderService方式启动Service的时候，触发此方法，多次binderService将不会多次触发，仅一次;
onUnBind: binderService的启动对应关闭，采用unbinderSrevice进行断开服务，便触发onUnBind操作；
onDestory: 停止服务有两个方式，stopService 或 unbinderservice 又或者stopself 都将触发onDestory;
```

###### 2、多种情景的生命周期变化

```
1.startService启动：onCreate->onStartCommand--->onDestory；
多次startService 将多次执行 onStartCommand，onCreate只会触发一次；
无论N多次startService，stop 只需要一次StopService即可；

2.binderService启动：onCreate->onBinder--->onUnbinder->onDestory;
多次binderService启动，仅会执行一次 onCreate、onBinder
无论N多次binderService启动，停止，只需要一次unbinderService即可

3.混合启动Service，也就是startService + binderService的启动方式，停止服务，需要 unbinder + stopService
```

###### 3、服务分类

```
1.本地服务：依托于主线程，一般用于APP内部后台服务，不能做耗时操作，
配置方式：<service android:name=".BackService"></service>
2.远程服务：运行在独立进程，用于不同APP之间或者多进程之间通信
配置方式：<service android:name=".RemoteService" android:process=":remote_process" />

前台服务：通过一定的方式将服务所在的进程提升优先级，会有一个正在运行的图标显示在状态栏，类似于通知，以此提升服务的优先级，后台服务的优先级比较，当内存不足的时候，容易内杀死回收，而前台服务正好弥补了这个缺陷；
通过 startForeground(id,notification) 提升服务优先级，注意不同版本的系统，创建notification 方式有些许不同
```

###### 4、Service 的通信

```
1.local_service: 
方式一： 采用binderService 启动，通过binder对象，创建连接，使用binder + handler 或者binder + callback 的形式进行通信
方式二：也可以使用广播的形式，进行发送消息，不过更加推荐方式一
2.remote_service：
方式一：推荐自定义AIDL 通过binderService 启动，连接成功后，获取Aidl.stub 对象，以此进行通信
方式二：也可以使用Messager,其原理和AIDL一致，实际上就是对AIDL的一种封装
```

###### 5、onStartCommand 返回值详解

```
这几个返回代表了，服务被系统杀死后，应该如何状态恢复；
START_STICKY：粘性，服务被系统kill掉后，会尝试重启服务，并且不携带 intent 信息，但不保证服务能重启成功；
START_NOT_STICKY：非粘性，服务被系统kill掉后，无任何动作；一死白死；
START_REDELIVER_INTENT：系统尝试重启服务，并把之前的intent进行传递；
```

###### 6、Service 、IntentService、JobIntentService、WorkManager的区别

```
IntentService： 通过源码可知：IntentService 是 Service 的子类，内部封装了 HandlerThread  + handler 进行通信，可以用于做异步任务，且任务完成后，无需手动关闭，一般可以进行下载任务之类的；
使用IntentService 的时候，只需要重写onHandlerIntent函数即可；
JobIntentService： 首先说，这个Service已经被弃用了，暂不关注
```

###### 7、Service保活的方式

```
1.如果使用startService ，则在startCommand中返回 start_sticky,
2.开启前台服务，因为前台服务优先级比较高
3.双进程守护，开启守护进程 + 心跳机制，确保服务kill之后，能有效重启
4.监听系统广播，比如WiFi变化，屏幕亮度，电池断连等等，如发现Service被kill，及时的唤起
5.覆盖Service的 onDestory 函数，及时的拉起服务
6.加入到系统白名单
7.置为系统应用
```





