# 浅谈 JVM 6：Android 虚拟机对比

Android 开发最初使用 Java 作为主要开发语言，后来由于版权等原因，Google 官方目前推荐使用 Kotlin。但无论是 Java 还是 Kotlin，都是需要编译成字节码，然后经过 Android 特有的优化，运行在 Android 虚拟机中的。JVM 主要为电脑而设计，直接在手机设备上使用会有很多问题，比如耗电等。Android 虚拟机也针对手机设备做了一些优化。

本文来简单对比一下 JVM 和 Android 虚拟机。

虚拟机在类型上分为两种。
1. 系统虚拟机：一个完整的虚拟机，提供了真实硬件的模拟，比如 VirtualBox、VMware 属于此类；
2. 处理虚拟机：通过解析指令的方式来执行程序，主要目标是提供一个平台无关的环境。

## JVM vs DVM

Java 虚拟机属于第二类，主要提供了平台无关的 Java 代码执行环境，其中一种方式就是通过解析字节码的方式运行。

JVM 工作流程可以主要参考 “浅谈 JVM 4：类加载机制” 一文，在此不再详述。

Android 系统采用 VM 来运行上层程序有两个原因：
1. 安全：应用层代码运行在 VM 中，和系统其他代码完全隔离，能够保证应用和系统的安全；
2. 平台无关：相同的代码可以运行在不同的架构中（ARM，MIPs，x86）。

Android 的虚拟机叫做 Dalvik Virtual Machine（DVM），该虚拟机能够执行 Dalvik EXecutable 字节码，可以使用 dx/d8 工具从 Java 字节码转换而来。转换过程中做了内存、处理速度的优化。相当于从多个 .class 文件编译为 .dex 文件放在 APK 中。

![](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1646404106.86SmartPic.png)

## 栈 vs 寄存器

根据我们前几篇文章分析，JVM 是基于栈的虚拟机，而 DVM 基于寄存器。基于寄存器的虚拟机使用更少的指令集、更少的代码和更少的内存操作，所以在性能上也有更好的表现。

![](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1646404323.92SmartPic.png)

这两者的差异意味着 Android 虚拟机的指令集中更多的是类似于汇编语言中的 `move` 等指令，而非 JVM 的 `push`、`pop`。

## DVM vs ART

ART 用于替代早期的 DVM，ART 为了兼容以前的程序，同样使用 dex 字节码。但是在安装时会使用 dex2oat 将 dex 字节码转换为 oat 本地码。

除此之外，ART 优化了垃圾回收机制、更快的本地方法调用、更加省电、应用程序启动更快、更好的运行时性能等。

![](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1646404989.32SmartPic.png)

你可以通过下图查看不同虚拟机的 apk 在安装后文件结构的差异。

![](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1646405044.64SmartPic.png)

## JIT vs AOT

Just In Time（JIT）在 JVM 有对应机制，在 Android 2.2 添加到虚拟机中。这是一种运行时优化热点代码的机制，即当虚拟机发现某段代码运行次数够多时，会将该段代码编译为机器码，下次执行该该代码时，直接运行机器码即可。这优化了程序运行性能。

Ahead of Time（AOT）是在安装应用时，将字节码编译为本地机器码。这减少了运行时编译的时间，提高了运行性能。但增大了安装耗时和存储空间占用。

## 总结

我们这次简单对比了下两个虚拟机的异同。掌握了本系列文章中涉及的知识之后，其实就已经具备了自己写一个虚拟机的理论知识。

## 拓展资料

- [Android 运行时](https://source.android.com/devices/tech/dalvik)
- [dex 文件格式](https://source.android.com/devices/tech/dalvik/dex-format)
- [Dalvik 指令集](https://source.android.com/devices/tech/dalvik/dalvik-bytecode)
- [Virtual Machine in Android: Everything you need to know](https://medium.com/android-news/virtual-machine-in-android-everything-you-need-to-know-9ec695f7313b)