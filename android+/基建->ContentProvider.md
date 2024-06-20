基建->broadcast

[toc]

简介：ContentProvider 属于四大组件之一，底层采用的Binder实现机制，用于进程间数据共享、数据通信；ContentProvider 包括以下几个角色：

###### 1、ContentProvider

```
以表格的形式组织数据，其中ContentProvider 包含多张表，每张表又包含多个行与列，
数据通信和数据共享的本质是数据的 增、删、改、查，故ContentProvider也提供了对应的操作
public Uri insert(Uri uri, ContentValues values)
public int delete(Uri uri, String selection, String[] selectionArgs)
public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)
public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)

Android 系统的通讯录、日程表本身就是采用了默认的 ContentProvider 进行多进程间的数据共享
ContentProvider 虽然提供数据的增删改查，但是它不会直接和外部进行交互，在ContentProvider之上还包裹着一层管理多个ContentProvider的管理者： ContentResolver
```

###### 2、ContentResolver

```
设计模式中六大原则中，有一项：单一职责；
ContentProvider 专注维护数据的增删改查
ContentResolver 专注管理不同的 ContentProvider
职责明确，对外提供更加简洁明了的API
ContentResolver提供了和ContentProvider 类似的增删改查的API
public Uri insert(Uri uri, ContentValues values)
public int delete(Uri uri, String selection, String[] selectionArgs)
public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)
public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)
```

###### 3、ContentProvider的辅助工具

```
ContentUris：作用是操作 uri,
Uri uri = Uri.parse("content://com.carson.provider/User/1")
uri是用于多进程间数据共享的路径
UriMatcher：在ContentProvider中注册Uri
ContentObserver：内容观察者，当ContentProvider内容发生改变时，通知外部
```

###### 4、扩展进程间数据共享的方法

```
1.通过ContentProvider 是一个绝好的数据通信和数据共享的方式，但是要做好权限校验，以便数据泄露
2.broadcast 同样适用于多进程间的数据通信，同样要做好注册和接收之间权限校验
3.AIDL 也是多进程间通信的一个不错选择
4.信使Messager 底层正式对AIDL的封装
5.文件共享，不过要注意文件的数据同步
6.socket通信，可以实现本地socket通信，一个APP为server端，一个为client端，以此进行通信
```

