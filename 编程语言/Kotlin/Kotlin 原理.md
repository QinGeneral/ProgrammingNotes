# Kotlin 原理

[TOC]

本文可以结合 [Kotlin 新特性是如何实现的](./Kotlin%20新特性是如何实现的.md) 一起阅读。

Kotlin 可以和 Java 互相调用，Java 可以识别 Kotlin 的独特语法，这是如何实现的？

这涉及到 Kotlin 的实现机制。本文从 Kotlin 的编译流程出发，探索这门语言的底层原理。

Kotlin 代码经过编译后，最终会变成 Java 字节码。Java 编译后也是编译成 Java 字节码。而 Kotlin 和 Java 能够兼容的原因也在于此，Java 和 Kotlin 本质上是在用同一种语言进行沟通。

![](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1692802361.005449SmartPic.png)

所以研究 Kotlin 可以从 Kotlin 代码编译后的 Java 字节码开始，或者将 Kotlin 转换成字节码后，再将字节码反编译成等价的 Java 代码。

可以说，Kotlin 的每一个语法，最终都会被翻译成对应的 Java 字节码。但如果你不去反编译，你甚至感觉不到它在幕后做的那些事情。而正是因为 Kotlin 编译器在背后做的这些翻译工作，才可以让我们写出的 Kotlin 代码更加简洁、更加安全。

- 类型推导，我们写 Kotlin 代码的时候省略的变量类型，最终被编译器补充回来了。
- 原始类型，虽然 Kotlin 没有原始类型，但编译器会根据每一个变量的可空性将它们转换成“原始类型”或者“包装类型”。
- 字符串模板，编译器最终会将它们转换成 Java 拼接的形式。
- when 表达式，编译器最终会将它们转换成类似 switch case 的语句。
- 类默认 public，Kotlin 当中被我们省略掉 public，最终会被编译器补充。
- 嵌套类默认 static，我们在 Kotlin 当中的嵌套类，默认会被添加 static 关键字，将其变成静态内部类，防止不必要的内存泄漏。
- 数据类，Kotlin 当中简单的一行代码“data class Person(val name: String, val age: Int)”，编译器帮我们自动生成很多方法：getter()、setter()、equals()、hashCode()、toString()、componentN()、copy()。
