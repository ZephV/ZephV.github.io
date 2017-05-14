---
title: Android应用组件 Service
date: 2017-03-30 16:18:09
tags: [AndroidNote,Service]
---
官方原文：[地址](https://developer.android.google.cn/guide/components/services.html#Lifecycle)
本文摘录自官方原文，方便自己观看。

service 是一个可以在后台长时间运行的操作而不提供用户界面的应用组件。服务可以由其他应用组件启动，而且即使用户切换到其他应用程序，服务仍将在后台继续运行。此外，组件可以绑定到服务，以与之进行交互，甚至执行进程间的通信（IPC）

**服务基本分为两种形式：**

**启动**
-  当应用组件（如Activity）通过调用` startService() `启动服务时，服务即处于"启动"状态。一旦启动，服务即可在后台无限期运行，即使启动服务的组件已被销毁也不受影响。已启动的服务通常是执行单一操作，而且不会将结果返回给调用方。

**绑定**
- 当应用组件通过调用` BindService() `绑定到服务时，服务即处于"绑定"状态。绑定服务提供了一个客户端-服务器接口，允许组件与服务进行交互、发送请求、获取结果，甚至是利用进程间通信（IPC）跨进程执行这些操作。仅当与另一个应用组件绑定时，绑定服务才会运行。多个组件可以同时绑定到此服务，但全部取消绑定后，该服务即会被销毁。

上述虽然分开概括这两种服务，但是服务可以同时以这两种方式运行，也就是说，他既可以是启动服务（以无限期运行），也允许绑定。问题在于是否实现了一组回调方法：` onStartCommand() `（允许组件启动服务）和` onBing() `（允许绑定服务）。

无论应用是出于启动状态还是绑定状态，亦或处于启动并且绑定状态，任何应用组件均可以像使用Activity那么调用Itent来使用服务（即使此服务来自另一应用）。 不过，您可以通过清单文件将服务声明为私有服务，并阻止其他应用访问。 使用清单文件声明服务部分将对此做更详尽的阐述。

**注意：**
服务在其托管进程的主线程中运行，它既不创建自己的线程，也不在单独的进程中运行（除非另行指定）。这意味着，如果服务将执行任何CPU密集型工作或者阻止性操作（我理解为耗时操作，例如 MP3 播放或联网），则应在服务内创建新线程来完成这项工作。通过使用单独的线程，可以降低发生“应用无响应”(ANR) 错误的风险，而应用的主线程仍可继续专注于运行用户与 Activity 之间的交互。

## 基础知识
要创建服务，您必须创建 Service 的子类（或使用它的一个现有子类）。在实现中，您需要重写一些回调方法，以处理服务生命周期的某些关键方面并提供一种机制将组件绑定到服务（如适用）。 应重写的最重要的回调方法包括：

 **`onStartCommand()`**
- 当另一个组件（如 Activity）通过调用 `startService()` 请求启动服务时，系统将调用此方法。一旦执行此方法，服务即会启动并可在后台无限期运行。 如果您实现此方法，则在服务工作完成后，需要由您通过调用 `stopSelf()` 或 `stopService()` 来停止服务。（如果您只想提供绑定，则无需实现此方法。）

**`onBind()`**  
- 当另一个组件想通过调用 `bindService()` 与服务绑定（例如执行 RPC）时，系统将调用此方法。在此方法的实现中，您必须通过返回 IBinder 提供一个接口，供客户端用来与服务进行通信。请务必实现此方法，但如果您并不希望允许绑定，则应返回 null。
    
**`onCreate()`**
- 首次创建服务时，系统将调用此方法来执行一次性设置程序（在调用 `onStartCommand()` 或 `onBind()` 之前）。如果服务已在运行，则不会调用此方法.

**`onDestroy()`** 
- 当服务不再使用且将被销毁时，系统将调用此方法。服务应该实现此方法来清理所有资源，如线程、注册的侦听器、接收器等。 这是服务接收的最后一个调用。

如果组件通过调用 `startService()` 启动服务（这会导致对 `onStartCommand()` 的调用），则服务将一直运行，直到服务使用 `stopSelf()` 自行停止运行，或由其他组件通过调用 `stopService()` 停止它为止。

如果组件是通过调用 `bindService()` 来创建服务（且未调用 `onStartCommand()`，则服务只会在该组件与其绑定时运行。一旦该服务与所有客户端之间的绑定全部取消，系统便会销毁它。

## 使用清单文件声明服务
如同 Activity（以及其他组件）一样，您必须在应用的清单文件中声明所有服务。

要声明服务，请添加 `<service>` 元素作为 `<application>` 元素的子元素。例如：

    <manifest ... >
      ...
      <application ... >
          <service android:name=".ExampleService" />
          ...
      </application>
    </manifest>
为了确保应用的安全性，**请始终使用显式 Intent 启动或绑定** Service，且不要为服务声明 Intent 过滤器。 启动哪个服务存在一定的不确定性，而如果对这种不确定性的考量非常有必要，则可为服务提供 Intent 过滤器并从 Intent 中排除相应的组件名称，但随后必须使用 `setPackage()` 方法设置 Intent 的软件包，这样可以充分消除目标服务的不确定性。

此外，还可以通过添加 `android:exported` 属性并将其设置为 `"false"`，确保服务仅适用于您的应用。这可以有效阻止其他应用启动您的服务，即便在使用显式 Intent 时也如此

## 创建启动服务
**Service**
- 这是适用于所有服务的基类。扩展此类时，必须创建一个用于执行所有服务工作的新线程，因为默认情况下，服务将使用应用的主线程，这会降低应用正在运行的所有 Activity 的性能。

**IntentService**
- 这是 Service 的子类，它使用工作线程逐一处理所有启动请求。如果您不要求服务同时处理多个请求，这是最好的选择。 您只需实现 `onHandleIntent()` 方法即可，该方法会接收每个启动请求的 Intent，使您能够执行后台工作。

## 您应使用服务还是线程？
简单地说，服务是一种即使用户未与应用交互也可在后台运行的组件。 因此，您应仅在必要时才创建服务。

如需在主线程外部执行工作，不过只是在用户正在与应用交互时才有此需要，则应创建新线程而非服务。 例如，如果您只是想在 Activity 运行的同时播放一些音乐，则可在 onCreate() 中创建线程，在 onStart() 中启动线程，然后在 onStop() 中停止线程。您还可以考虑使用 AsyncTask 或 HandlerThread，而非传统的 Thread 类。

## 在前台运行服务
前台服务被认为是用户主动意识到的一种服务，因此在内存不足时，系统也不会考虑将其终止。 前台服务必须为状态栏提供通知，放在“正在进行”标题下方，这意味着除非服务停止或从前台移除，否则不能清除通知。

要请求让服务运行于前台，请调用 `startForeground()`。此方法采用两个参数：唯一标识通知的整型数和状态栏的 `Notification`。例如：

```
Notification notification = new Notification(R.drawable.icon, getText(R.string.ticker_text),
        System.currentTimeMillis());
Intent notificationIntent = new Intent(this, ExampleActivity.class);
PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, notificationIntent, 0);
notification.setLatestEventInfo(this, getText(R.string.notification_title),
        getText(R.string.notification_message), pendingIntent);
startForeground(ONGOING_NOTIFICATION_ID, notification);
```

**注意**：提供给 `startForeground()` 的整型 ID 不得为 0。

要从前台移除服务，请调用 `stopForeground()`。此方法采用一个布尔值，指示是否也移除状态栏通知。 此方法不会停止服务。 但是，如果您在服务正在前台运行时将其停止，则通知也会被移除。

## 实现生命周期回调
与 Activity 类似，服务也拥有生命周期回调方法，您可以实现这些方法来监控服务状态的变化并适时执行工作。 以下框架服务展示了每种生命周期方法：

```
    public class ExampleService extends Service {
    int mStartMode;       // indicates how to behave if the service is killed
    IBinder mBinder;      // interface for clients that bind
    boolean mAllowRebind; // indicates whether onRebind should be used

    @Override
    public void onCreate() {
        // The service is being created
    }
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        // The service is starting, due to a call to startService()
        return mStartMode;
    }
    @Override
    public IBinder onBind(Intent intent) {
        // A client is binding to the service with bindService()
        return mBinder;
    }
    @Override
    public boolean onUnbind(Intent intent) {
        // All clients have unbound with unbindService()
        return mAllowRebind;
    }
    @Override
    public void onRebind(Intent intent) {
        // A client is binding to the service with bindService(),
        // after onUnbind() has already been called
    }
    @Override
    public void onDestroy() {
        // The service is no longer used and is being destroyed
    }
}
```
**注**：与 Activity 生命周期回调方法不同，您**不**需要调用这些回调方法的超类实现。
![Service的生命周期图](https://developer.android.google.cn/images/service_lifecycle.png)

**注**：尽管启动服务是通过调用 `stopSelf()` 或 `stopService()` 来停止，但是该服务并无相应的回调（没有 `onStop()` 回调）。因此，除非服务绑定到客户端，否则在服务停止时，系统会将其销毁 — `onDestroy()` 是接收到的唯一回调。