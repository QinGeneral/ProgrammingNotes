# Android 热修复原理

[TOC]

热修复和插件化过程中都涉及到动态加载类、资源，两者也都涉及到使用 Gradle 插件自动化打包过程，两者也都是由动态加载技术派生出来的。但是热修复的目标是修复已有问题，插件化目标是动态下发组件功能和复杂功能解耦，涉及到解决四大组件生命周期的问题。

热修复原理涉及到 ClassLoader，可以参考 [Java ClassLoader](../编程语言/Java/Java%20ClassLoader.md) 和 [Android ClassLoader](./Android%20ClassLoader.md)。

## 代码热修复方案

| 方案           | 生效时机 | 优缺点                         | 框架                |
| -------------- | -------- | ------------------------------ | ------------------- |
| Native Hook    | 实时生效 | 需要 native 修改，兼容性差     | Andfix              |
| Dex 插桩、替换 | 重启生效 | 实现简单，兼容性好             | Tinker、QFix、Amigo |
| InstantRun     | 实时生效 | 实现简单，兼容性好，增大包体积 | Robust              |

### Native Hook 替换 ArtMethod 内容

Java 中的类、方法、变量，对应到虚拟机 native 里的实现是 Class、ArtMethod、ArtField。Native Hook 就是替换 ArtMethod 内容，实现方法替换。

此方案无需重启，但是需要修改 native 代码，兼容性差。

### Dex 插桩、替换

前文讲到，BaseDexClassLoader 是通过遍历 Element 数组来查找类的，利用这个机制，可以在 Element 数组中插入新的 Element，从而实现类的插桩或全量替换。

由于类无法被卸载，出问题类已经加载了，所以必须重启才能首先加载修复的类，达到修复的效果。

### InstantRun

InstantRun 方案就是在编译期使用 ASM 为每个方法注入一段代码，这段代码会在方法执行前先判断是否有新的方法，如果有则执行新的方法，否则执行原方法。

此方案首先会根据补丁，拿到所有要热修复的类列表，然后将这些类列表中预置的 localIncrementalChange 变量通过反射的方式注入新的值。在方法执行时，判断 localIncrementalChange 是否为 null，如果不为 null，则执行新的方法，否则执行原方法。

## 资源热修复方案

### InstantRun

InstantRun 资源热修复的方案如下：
创建一个新的AssetManager，通过反射调用addAssetPath方法加载外部（SD卡）的资源。遍历Activity列表，得到每个Activity的Resources，通过反射得到Resources的AssetManager类型的mAssets字段，并改写mAssets字段的引用为新的AssetManager。采用同样的方式，将Resources.Theme的mAssets字段的引用替换为新创建的AssetManager。紧接着根据SDK 版本的不同，用不同的方式得到Resources 的弱引用集合，再遍历这个弱引用集合，将弱引用集合中的Resources的mAssets字段引用都替换成新创建的AssetManager。

可以简单地总结为两个步骤：
1. 创建新的AssetManager，通过反射调用addAssetPath方法加载外部的资源，这样新创建的AssetManager就含有了外部资源；
2. 将AssetManager类型的mAssets字段的引用全部替换为新创建的AssetManager。

在 Sophix 方案中，也可以只构造一个对应的资源包，然后将其添加到 AssetManager 中，将原来的 AssetManager 重新初始化，这样可以避免繁琐的反射操作实现热修复。

## so 热修复方案

### 替换 System.loadLibrary

Android 中加载 so 的方法有两个：
```java
System.loadLibrary(String libName)：用来加载已经安装的 apk 中的 so；
System.load(String pathName)：可以加载自定义路径下的 so。
```

我们可以增加一层代理，将 System.loadLibrary 替换为自己的实现，然后在自己的实现中，先加载修复包中的 so，然后再加载原来的 so。

### 反射注入补丁 so

DexPathList 中除了 Element 数组外，还包含一个 NativeLibraryElement 数组。Native so 的加载机制和 dex 文件加载机制类似，也是通过遍历 NativeLibraryElement 数组来查找 so 的，所以我们可以通过反射的方式，将补丁包中的 NativeLibraryElement 数组插入到 Element 数组中。