---
title: Android Service(服务)
date: 2017-03-30 16:18:09
tags: [AndroidNote,Service,AsyncTask]
---



## What is a Service

[**Service官方文档**](https://developer.android.google.cn/reference/android/app/Service.html "官方文档")

**服务(Service)**是一种应用程序组件代表应用程序的意图，而不是与用户或其他应用程序使用电源的功能执行更长的运行操作。每个服务类必须有一个相应的服务声明在其`AndroidManifest.xml`。启动服务的方法有`Context.startService()`, `Context.bindService()`。

**服务** 是运行在他们的主进程的主线程。这意味着，如果你的服务是要做任何CPU消耗大的操作（如MP3播放）或阻塞（如网络）的操作，它会以它自己的线程去做这项工作。

**Most confusion about the Service class actually revolves around what it is not :**

- 服务不是一个单独的进程。服务对象本身并不意味着它是在它自己的进程中运行；除非另有规定，它运行在同一进程中的应用就是它的一部分。
​
- 服务不是一个线程。这并不意味着主线程去完成相应工作（避免应用程序无响应的错误）。

**Thus a Service itself is actually very simple, providing two main features :**

- 用于告诉系统什么要在后台运行（即用户不直接与应用程序交互）,调用`Context.startService()` 启动服务，这要求系统来管理该服务，该服务会一直运行到明确告诉它停止为止。
 ​
- 一个应用程序提供它的一些功能给其他的应用，这对应`Context.bindService()`的调用。通过`Context.bindService()`来准许一个长期的连接，从而让**Service**与其他应用程序交互。



## ServiceLifeCycle 服务的生命周期

有两个理由可以说明**服务**可以被系统运行。如果有人`Context.startService()`来启动服务,然后系统会检索服务（创造它，如果需要调用它的`oncreate()`方法）然后调用`onStartCommand(Intent, int, int)` 方法。之后该服务将一直运行，直到`stopservice()`或`stopself()`被调用。请注意，当服务启动后，再调用`Context.startService()` 方法时，（`onstartcommand()`会被调用`oncreate()`方法不会再次被调用，也就是说服务只会被创建一次，然调用`stopservice()`或`stopself()`会停止服务；然而，服务可以用`stopSelf（int）`来保证服务直到成功打开意图之后才停止。

客户端（Clients）也还可以使用`Context.bindService（）`来获取与服务的持久连接。 如果它还没有运行，则同样创建该服务（在执行此操作时调用`onCreate()`），但不调用`onStartCommand()`。 客户端将收到该服务从其`onBind(Intent)`方法返回的`IBinder`对象，让客户端（绑定者）可以调用相关服务。


对于已经启动的服务，还有两种额外的主要操作模式可以决定运行。这取决于`onStartCommand()`的返回值。`START_STICKY`用于根据需要明确启动和停止的服务，而`START_NOT_STICKY`或`START_REDELIVER_INTENT`用于在处理发送给他们的任何命令时应保持运行的服务。该服务被创建后将和绑定者保持连接（不管客户端是否仍保留服务的`IBinder`）。通常`IBinder`返回的是一个复杂的接口，这个已经在AIDL中实现。

A service can be both started and have connections bound to it. In such a case, the system will keep the service running as long as either it is started or there are one or more connections to it with the `Context.BIND_AUTO_CREATE` flag.
 一旦这些情况都不成立，服务的`onDestroy()`就被调用，服务被有效地终止。 从`onDestroy()`返回后，所有清理（停止线程，注销接收器）都应该完成。
 反之：只要存在一种以上情况,`onDestroy()`不会被调用。



## 前台服务

前台服务是那些被认为用户知道（用户认可所认可）且在系统内存不足的时候不允许系统杀死的服务。前台服务必须给状态栏提供一个通知，它被放到正在运行(Ongoing)标题之下——这就意味着通知只有在这个服务被终止或从前台主动移除通知后才能被解除。

```
public class MyService extends Service {
	
	public void onCreate() {
        super.onCreate();
        Intent intent = new Intent(this,MainActivity.class);
        PendingIntent pi = PendingIntent.getActivity(this, 0, intent, 0);
        Notification notification = new NotificationCompat.Builder(this)
                .setContentTitle("This is content title")
                .setContentText("This is content text")
                .setWhen(System.currentTimeMillis())
                .setSmallIcon(R.mipmap.ic_launcher)
                .setLargeIcon(BitmapFactory.decodeResource(getResources(),R.mipmap.ic_launcher))
                .setContentIntent(pi)
                .build();
        startForeground(1, notification);
        Log.d("MyService","onCreate executed");
    }
} 
```

## 线程的基本用法

## AsyncTask
**AsyncTask**是Android异步消息处理工具。
### 自定义AsyncTask
**AsyncTask**是一个抽象类，我们在用子类进程它的时候能指定三个泛型参数: 
***Params ,Progress ,Result*** 
- ***Params*** 在后台执行AysnTest时需要传入的参数，用于后台任务使用
- ***Progress*** 后台任务执行时，如果需要在界面上显示当前进度，则使用这里指定的泛型作为单位
- ***Result*** 任务执行完毕时，需要返回结果的返回值类型

```
public class DownloadTask extends AsyncTask<Void, Integer, Boolean>{

    /**
     * 这个方法在后台任务开始执行前调用，用于进行一些界面上的初始化，比如显示一个进度条对话框等。
     */
    @Override
    protected void onPreExecute() {
        super.onPreExecute();
    }

    /**
     * 这个方法中的所有代码都会在子线程中运行,我们应该在这里去处理所有的耗时任务。
     * 任务完成会return结果,如果AsyncTask的第三次泛型为Void则不返回结果。
     * 这个方法内不能进行UI操作，比如反馈当前任务进度，可以调用publishProgress(Progress..)方法来
     * 完成
     */
    @Override
    protected Boolean doInBackground(Void... params) {
        return null;
    }

    /**
     * 当任务中调用了publishProgress(Progress..)方法后,onProgeressUpdate(Progress...)方法很快就
     * 会被调用，此方法携带的参数就是后台任务中传递过来的。在这个方法中可以对UI进行操作，利用
     * 参数中的数值可以对界面元素进行相应的更新
     */
    @Override
    protected void onProgressUpdate(Integer... values) {
        super.onProgressUpdate(values);
    }

    /**
     * 当后台任务执行完毕并通过return语句进行返回时，这个方法就会很快被调用。返回的数据会作为参数
     * 传递到此方法中，可以利用返回的数据来进行一些UI操作，如提醒任务执行结果，一级进度条对话框等
     */
    @Override
    protected void onPostExecute(Boolean aBoolean) {
        super.onPostExecute(aBoolean);
    }
}
```
### 启动AsyncTask
如果要启动上面这个任务可以编写：

```
new DownloadTask().execute();
```



