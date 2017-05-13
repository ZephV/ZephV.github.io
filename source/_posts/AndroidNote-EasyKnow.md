---
title: EasyKnow
date: 2017-04-18 01:31:11
tags: [AndroidNote, EasyKnow]
---
# CardView

- Layout 
 app:elevation // 指定卡片高度
 app:cardCornerRadius // 指定卡片圆角的弧度
	

# Glide

一个超级强大的图片加载库


# RecyclerView

- 映射类
- 子项布局
- 适配器
 继承RecycleView.Adapter<类名.ViewHolder>
 定位器ViewHolder继承RecyclerView.ViewHolder
 构造方法
 重写onCreatViewHolder(),onBindViewHolder(), getItemCount()三个方法
- 对要适配的内容进行适配
 

# Java构造方法

构造方法用于对类的成员变量进行初始化。

是类就有构造方法，如果我们没有给出构造方法，系统将自动提供一个无参构造方法，如果我们给出了构造方法，系统将不会提供。

方法重载：
- 普通方法重载：主要是当两个方法的功能相似而参数列表（参数的类型或个数）不同时使用。
- 构造方法重载：使成员变量具有不同的初值，重载时也要求参数列表不同。 

