基建->broadcast + contentProvider

[toc]

简介：broadcast 和 contentprovider 同属四大组件的部分

###### 1、broadcast

```
Service 和 Activity 一样都是Context 的子类，但Service没有界面，故生命周期也跟着简单了一些：
onCreate:	第一次创建会运行此方法，如果Service已经存在，将不再二次运行；
onStartCommand: 采用startService方式启动Service的时候，触发次方法，每startService一次，将触发一次；
onBind: 采用binderService方式启动Service的时候，触发此方法，多次binderService将不会多次触发，仅一次;
onUnBind: binderService的启动对应关闭，采用unbinderSrevice进行断开服务，便触发onUnBind操作；
onDestory: 停止服务有两个方式，stopService 或 unbinderservice 又或者stopself 都将触发onDestory;
```

