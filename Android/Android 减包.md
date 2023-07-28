# Android 减包

[TOC]

## 安装包增量

对比两个安装包的增量部分。

## 减包

### 资源

* 移除未使用的资源，未使用资源可以使用 lint 工具检查；
* 仅支持特定密度
  * 至少包含 xxhdpi
* 资源、代码线上加载；
* 压缩 PNG 图片；
  * 通过 `buildTypes.all { isCrunchPngs = false }` 开启
* 使用 WebP 格式图片；
* 使用矢量图形；

### 代码

- 避免使用枚举，请考虑使用 @IntDef 注解和代码缩减移除枚举并将它们转换为整数；
* 缩减原生库大小，移除非必要的 so 库；

### 维护多个精简的 apk

## 参考资料

- [缩减应用大小](https://developer.android.com/topic/performance/reduce-apk-size?hl=zh-cn)