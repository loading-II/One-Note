基建->broadcast

[toc]

简介：Broadcast 属于四大组件之一，是属于全局的监听器，broadcast是基于观察者模式的实现机制，他有三个角色部分：消息订阅者(broadcast receiver)、消息发布者(broadcast)、消息管理者(activity manager service);

###### 1、broadcast receiver

```
通过Binder机制在AMS中注册，注册方式分两种（静态注册 和 动态注册）

静态注册：在配置清单中进行注册，静态注册又称之为常驻广播
优点：全局监听广播，适合长时间监听的场景
缺点很明显，因为需要时刻监听广播，所以导致耗电耗内存
注册示例：
<receiver android:name=".MBroadastReceiver"
          android:exported="true">
            <intent-filter>
                <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
                <action android:name="com.android.offcn.demo" />
            </intent-filter>
</receiver>

动态注册：在组件内按需代码注册，切记动态注册的广播接收者应有对应的‘注销’实现，否则会造成内存泄露

注册示例：
MBroadastReceiver mBroadcastReceiver = new MBroadastReceiver();
IntentFilter intentFilter = new IntentFilter();
intentFilter.addAction("com.android.offcn.demo");
intentFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
registerReceiver(mBroadcastReceiver, intentFilter);

动态和静态更有优缺点，场景不同故各有所需

下面来一段接收者的代码示例：
public class MBroadastReceiver extends BroadcastReceiver {
    private String TAG = "MBroadastReceiver";

    @Override
    public void onReceive(Context context, Intent intent) {
        Log.d(TAG, "onReceive: ");
    }
}
```

###### 2、发送者

```
广播是用 Intent 标识进行发送的，比如：
Intent intent = new Intent();
intent.setAction("com.android.offcn.demo");
sendBroadcast(intent);
便是发送一条消息至广播，
```

###### 3、广播的分类

```
1.普通广播：即开发者自定义的广播，一般用于APP内通信，也是最常用的一种广播；
2.系统广播：系统提供很多的广播，比如WiFi状态改变，屏幕亮暗，关闭或打开飞行模式等等；
3.有序广播：通过sendOrderedBroadcast(intent)发送的广播，接收者按照一定顺序接收，先接收者可截断修改广播
4.本地广播：又称之为APP内广播，注册广播时exported=false，通过LocalBroadcastManager进行注册和注销
```



