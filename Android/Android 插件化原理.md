# Android 插件化原理

[TOC]

热修复和插件化过程中都涉及到动态加载类、资源，两者也都涉及到使用 Gradle 插件自动化打包过程，两者也都是由动态加载技术派生出来的。但是热修复的目标是修复已有问题，插件化目标是动态下发组件功能和复杂功能解耦，涉及到解决四大组件生命周期的问题。

热修复原理涉及到 ClassLoader，可以参考 [Java ClassLoader](../编程语言/Java/Java%20ClassLoader.md) 和 [Android ClassLoader](./Android%20ClassLoader.md)。

插件化涉及到两个概念：
- 宿主
- 插件

插件运行在宿主之上，由于 Android 四大组件必须在 AndroidManifest 中注册，所以插件的动态下发运行，需要提前预埋占位 Activity，称之为 StubActivity。

关于代码和资源的动态加载可以参考对比 [Android 热修复原理](./Android%20热修复原理.md)，本文主要介绍四大组件生命周期问题的解决。

## Activity 生命周期问题和资源问题

### 方案 1 手动调用

启动时，直接启动 StubActivity，然后在 StubActivity 中通过反射的方式加载 TargetActivity，然后在对应生命周期调用 TargetActivity 的生命周期和资源。

本方案采用反射的方式，效率较低，但实现相对简单。

### 方案 2 Hook IActivityManager 和 H

#### 1. 绕过 AMS 校验

1. AMS 会对 Activity 做校验，如果没有在 AndroidManifest 中注册，则会抛出异常，所以需要在宿主中预埋占位 StubActivity；
2. 需要找时机，对 AMS 启动 Activity 的逻辑进行 Hook；
3. Android 启动 Activity 时，通过 IActivityManager 获取到 AMS 对象；
4. 通过反射机制，将 Hook 类替换 IActivityManager 中的 mInstance 字段；
5. 在 Hook 中会判断是否是 startActivity 方法，是的话会反射调用 StubActivity 的 startActivity 方法。

#### 2. 真正的 Activity 生命周期问题

在步骤 1 中，为了绕过 AMS 的校验，最终启动了 StubActivity，但是我们的目标是真正的 TargetActivity 生命周期能够正常调用。

对 ActivityThread 的 Handler 回调中做处理：
1. AMS 在处理完 Activity 的栈，会回调 ActivityThread 中 H 类的 handleMessage 方法，在此处理 Activity 的生命周期；
2. 可以 Hook 此 Handler，在对应生命周期回调中，替换 Intent 为 TargetActivity 的 Intent，然后调用原方法。

### 方案 2 Hook Instrumentation

![](images/Activity%20Hook.png)

1. 实现 Instrument 代理类：
   1. 在 execStartActivity 方法中，如果原 Intent 中的 Activity 没有注册，则替换为 StubActivity，然后反射调用原 Instrumentation.execStartActivity 方法，以此跳过 AMS 校验；
   2. 在 newActivity 方法中，将 Intent 替换为 TargetActivity 的 Intent；
2. 通过反射，将 Instrumentation 替换为代理类。

本方案需要了解系统机制，实现相对复杂，需要适配不同版本的系统。但是处理比较完善。

##  Service 生命周期问题

启动时，直接启动 StubService，然后在 StubService 中通过反射的方式加载 TargetService，然后在对应生命周期调用 TargetService 的生命周期和资源。

## BroadcastReceiver

动态广播不需要在 AndroidManifest 中注册，方案类似 Service。

静态广播稍微麻烦一点，可以解析 Manifest 文件找到其中静态注册的 Broadcast 并进行动态注册，这里就不对 Manifest 进行解析了，知道其原理即可。

## ContentProvider

ContentProvider 方案类似 Service。