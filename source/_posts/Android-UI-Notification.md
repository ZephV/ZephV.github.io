---
title: 'Android UI — Notification通知'
date: 2017-05-14 00:24:46
tags: [AndroidNote, UI, Notification]
---
摘录自[官方文档](https://developer.android.google.cn/guide/topics/ui/notifiers/notifications.html)



通知是您可以在应用的常规 UI 外部向用户显示的消息。当您告知系统发出通知时，它将先以图标的形式显示在**通知区域**中。用户可以打开**抽屉式通知栏**查看通知的详细信息。 通知区域和抽屉式通知栏均是由系统控制的区域，用户可以随时查看。


## 创建通知

您可以在`NotificationCompat.Builder`对象中为通知指定UI信息和操作。要创建通知，请调用` NotificationCompat.Buider.build() `它将返回包含您的具体规范的Notification对象。要发出通知，请通过调用`NotificationManager.notify()`将Notification对象传递给系统。

### 必须的通知内容

Notification对象必须包含一下内容：

- 小图标 , 由` setSmallIcon()`设置
- 标题 , 由` setContentTitle()`设置
- 详细文本, 由` setContentText()`设置

### 可选通知内容设置

[NotificationCompat.Builder](https://developer.android.google.cn/reference/android/support/v4/app/NotificationCompat.Builder.html)参考

### 通知操作

尽管通知操作都是可选的，但是您至少应向通知添加一个操作。 操作允许用户直接从通知转到应用中的 `Activity`，他们可在其中查看一个或多个事件或执行进一步的操作。

一个通知可以提供多个操作。您应该始终定义一个当用户点击通知时会触发的操作；通常，此操作会在应用中打开 `Activity`。 您也可以向通知添加按钮来执行其他操作，例如，暂停闹铃或立即答复短信；

在 `Notification` 内部，操作本身由 `PendingIntent` 定义，后者包含在应用中启动 `Activity` 的 `Intent`。要将 `PendingIntent` 与手势相关联，请调用 `NotificationCompat.Builder` 的适当方法。例如，如果您要在用户点击抽屉式通知栏中的通知文本时启动`Activity`，则可通过调用 `setContentIntent()` 来添加 `PendingIntent`。

在用户点击通知时启动 `Activity` 是最常见的操作场景。此外，您还可以在用户清除通知时启动 `Activity`。在 Android 4.1 及更高版本中，您可以通过操作按钮启动 `Activity`。

### 通知优先级

可以根据需要设置通知的优先级。优先级充当一个提示，提醒设备 UI 应该如何显示通知。 要设置通知的优先级，请调用 `NotificationCompat.Builder.setPriority()` 并传入一个 `NotificationCompat` 优先级常量。有五个优先级别，范围从 `PRIORITY_MIN` (-2) 到 `PRIORITY_MAX` (2)；如果未设置，则优先级默认为 `PRIORITY_DEFAULT` (0)。

### 创建简单通知

以下代码段说明了一个指定某项 Activity 在用户点击通知时打开的简单通知。 请注意，该代码将创建 `TaskStackBuilder` 对象并使用它来为操作创建 `PendingIntent`。[启动 Activity 时保留导航](https://developer.android.google.cn/guide/topics/ui/notifiers/notifications.html#NotificationResponse)部分对此模式做了更详尽的阐述：

[启动 Activity 时保留导航](https://developer.android.google.cn/guide/topics/ui/notifiers/notifications.html#NotificationResponse)部分对此模式做了更详尽的阐述：

```
NotificationCompat.Builder mBuilder =
        new NotificationCompat.Builder(this)
        .setSmallIcon(R.drawable.notification_icon)
        .setContentTitle("My notification")
        .setContentText("Hello World!");
// Creates an explicit intent for an Activity in your app
Intent resultIntent = new Intent(this, ResultActivity.class);

// The stack builder object will contain an artificial back stack for the
// started Activity.
// This ensures that navigating backward from the Activity leads out of
// your application to the Home screen.
TaskStackBuilder stackBuilder = TaskStackBuilder.create(this);
// Adds the back stack for the Intent (but not the Intent itself)
stackBuilder.addParentStack(ResultActivity.class);
// Adds the Intent that starts the Activity to the top of the stack
stackBuilder.addNextIntent(resultIntent);
PendingIntent resultPendingIntent =
        stackBuilder.getPendingIntent(
            0,
            PendingIntent.FLAG_UPDATE_CURRENT
        );
mBuilder.setContentIntent(resultPendingIntent);
NotificationManager mNotificationManager =
    (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
// mId allows you to update the notification later on.
mNotificationManager.notify(mId, mBuilder.build());
```



##  管理通知

当您需要为同一类型的事件多次发出同一通知时，应避免创建全新的通知， 而是应考虑通过更改之前通知的某些值和/或为其添加某些值来更新通知。

### 更新通知

