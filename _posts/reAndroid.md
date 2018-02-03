---
title: 重识 Android
date: 2017-09-19 11:27
tags:
	- Android
---

> Android 基础知识整理。

<!-- more -->

### 1. Activity

#### 1.1 Activity 生命周期

状态：`running` `paused` `stopped` `killed`

启动：onCreate() -> onStart() -> onResume()

点击 Home ：-> onPause() -> onStop()

重现：-> onRestart() -> onStart() -> onResume()

退出：-> onPause() -> onStop() -> onDestory()

进程优先级：`前台`(可交互) `可见`(失去焦点) `服务` `后台`(不可见) `空`(缓存用)

#### 1.2 Activity 任务栈

standard：标准模式(每次都创建)

singleTop：栈顶复用模式(栈顶检测)

singleTask：栈内复用模式(栈内检测)

singleInstance：单实例模式(独立的任务栈)

#### 1.3 Activity 启动模式

#### 1.4 Scheme 跳转协议

#### 1.5 参考

### 2. Fragment

#### 2.1 生命周期

Create: 准备视图

`onAttach`: Fragment 与 Activity 关联

`onCreate`: 创建 Fragment 对象

`onCreateView`: 创建视图

`onActivityCreated`: Activity 对象创建完成

Start: 加载视图

`onStart`: Fragment 可见

Resume: 获取焦点

`onResume`: Fragment 可交互 

Pause 失去焦点

`onPause`: Fragment 失去焦点

Stop 视图不可见

`onStop`: Fragment 不可见

Destory 销毁对象

`onDestoryView`: Fragment 视图销毁

`onDestory`: 对象销毁

`onDetach`: 解绑 Fragment 并销毁对象

> 以上为个人理解，不是完整的视图加载过程，只是属于一个理解分析的过程。

#### 2.2 Fragment 添加到 Activity 中

静态加载：XML

动态加载： FragmentManager

- FragmentPagerAdapter(detach)

  > 适用于页面较少的情况，不销毁 Fragment 只与 Activity 脱离。

- FragmentStatePagerAdapter(remove)

  > 适用于页面较多的情况，直接移除 Fragment。

#### 2.3 Fragment 通信

1. Fragment 中调用 Activity : 调用 getActivity()
2. Activity 中调用 Fragment : Fragment 回调函数
3. Fragment 中调用 Fragment : findFragmentById()

#### 2.4 典型方法

`replace`: 替换 Fragment 实例 

`add`: 添加 Fragment 实例 

`remove`: 移除 Fragment 实例


#### 2.5参考

1. [Android_Tutor : 两分钟彻底让你明白Android Activity生命周期(图文)!](http://blog.csdn.net/android_tutor/article/details/5772285)

### 3. Service

#### 3.1 Service 和 Thread

Service: 依托于所在的线程并且在后台运行，不可做耗时操作，否则会 ANR。
Thread: 主要是出处理耗时操作。

### 4. BroadCast Receiver


