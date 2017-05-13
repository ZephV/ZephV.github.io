---
title: '清单文件Manifest中的android:name'
date: 2017-05-14 00:24:46
tags: [AndroidNote,  Manifest]
---
这里` android:name `是指Manifest中Application的属性，而不是Application里Activity属性里的.

官方连接：https://developer.android.google.cn/guide/topics/manifest/activity-element.html

## 官方原文：
The fully qualified name of an Application subclass implemented for the application. When the application process is started, this class is instantiated before any of the application's components.
The subclass is optional; most applications won't need one. In the absence of a subclass, Android uses an instance of the base Application class.

## 根据我自己理解翻译（如有错误望指出，谢谢）：

android:name="指定一个子类应用充分正确的名字（完整的包名应用名） " 

 **作用 :**
- 当应用程序进程启动时，这个被指定的子类在任何应用的组件之前被实例化。
- 一般情况下并不需要指定子类（也就是说` android:name `这个属性是可选的）。
- 在你没有指定子类的情况下，系统（Android）会自动实例化应用程序基础类。