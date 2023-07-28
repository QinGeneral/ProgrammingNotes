# Android 编译优化

[TOC]

## Android Build Profile

通过 Profile 分析各部分编译时间。

## 编译优化

- 使用最新的工具；
  - AndroidStudio
  - Gradle 插件
  - Kotlin 插件
- 编译配置过程支持缓存；
- 禁用 jetifier；
- 将稳定模块改为 aar 引入；
- 图片转为 WebP 格式；
- 停用 PNG 处理，Gradle 3.0.0 以上默认对调试编译类型停用 PNG 处理；
- 增加 JVM 堆大小。

## 参考资料

* [Android 官方文档：优化构建速度](https://developer.android.com/studio/build/optimize-your-build?hl=zh-cn)