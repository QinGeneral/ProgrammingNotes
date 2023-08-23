# Android View 绘制机制

每个 Activity 的 contentView 外层都嵌套了 DecorView，DecorView 最终要附着在 WindowManager 上。ViewRoot 的实现类 ViewRootImpl 是两者之间的纽带。

View 的绘制会触发到 ViewRoot 的 performTraversals 方法，然后调用 ViewGroup 的 performMeasure、performLayout、performDraw 方法。ViewGroup 的 onMeasure 方法会调用子 View 的 measure 过程。

measure 阶段确定了 View 的大小，layout 阶段确定了 View 的位置，draw 阶段将 View 绘制到屏幕上。

![](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1692801455.944886SmartPic.png)