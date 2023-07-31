# 浅谈 JVM 1：一次编写，到处运行

不同于 C、C++ 无需运行时可直接编译为机器码运行，Java 程序需要运行在 JRE 之上。而正因如此，Java 拥有了 C、C++ 所不具有的可移植性。而在 JRE 之中，JVM 又是一个重要的组成。

之所以要大费周章地编写 Linux、macOS、Windows 平台的 JVM，其中之一便是因为 Java “一次编写，到处运行” 的设计理念。

打开 [Java 官网](https://docs.oracle.com/javase/specs/index.html) 可以看到以下内容：

![Java 官网](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1641718576.98SmartPic.png)

Java SE 的每个版本都包含两部分：
- Java 语言规范；
- 虚拟机规范。

我们再结合 Java、JRuby 等语言的编译执行过程来看，如下图：

![Java 编译执行过程](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1641719366.23SmartPic.png)

可以看到 **Java 官方之所以要将 Java 语言规范和虚拟机规范分别定义，就是为了让多种语言如 Java、JRuby、Jython、Groovy 等都可以运行在 JVM 之上。Java 并不是 JVM 支持运行的唯一语言。**

> 2018 年推出的 GraalVM，更是有一统天下语言运行时之趋势。可以参见 [GraalVM 官网](https://www.graalvm.org/)。

这其中的关键之一就是 class 文件。任何操作系统上编写的 Java 等语言的代码都可以编译为 class 字节码，然后将其交给不同平台的 JVM 解析、加载、链接、编译、执行。

JVM 除了能够执行 class 文件，让语言能够一次编写多处运行之外。还有以下特性支持：
- 内存管理
- 垃圾回收
- 数组越界等安全保护机制

既然 JVM 执行的是 class 字节码，而非原生 Java 代码。同时又有多种语言可以编译为 class 字节码，而不同语言的特性是不同的。比如 Java 字段类型一旦确定就不许变化，而 JavaScript 字段类型则更加灵活。这就意味着 class 字节码和 JVM 层面必须对多种语言特性进行支持，JVM 执行层面各字段类型不完全等于 Java 代码中的类型，JVM 执行代码的顺序也不完全等于 Java 代码中的顺序。

要想了解 Java 代码真正运行期间发生了什么，我们需要了解 class 字节码格式和 JVM 执行流程。这两者均在 [JVM 规范](https://docs.oracle.com/javase/specs/jvms/se17/html/index.html)中有所定义。

我们日常开发中让人头疼的问题，在对字节码和 JVM 有一定了解后都会烟消云散。比如：
- `if (flag)` 和 `if (flag == true)` 在字节码层面是否有区别？有何区别？
- 如下代码在 try catch 不同代码块调用 return，函数最终返回返回值是什么？为什么会这样？
    ```java
    // 此代码参考《深入理解 Java 虚拟机》代码清单 6-5
    public int inc() {
        int x;
        try {
            x = 1;
            return x;
        } catch (Exception e) {
            x = 2;
            return x;
        } finally {
            x = 3;
        }
    }
    ```
- 对于重写和重载方法，JVM 调用时如何定位到具体方法？
- 对于多态，JVM 又是如何实现？

> 推荐资料
>
> 1. 《深入理解 Java 虚拟机》
> 正如《重构》一书是代码重构领域的圣经，此书也是国内理解 JVM 的必读之书。
> 2. [Java 语言和虚拟机规范官网](https://docs.oracle.com/javase/specs/index.html)
