# Kotlin 扩展

Kotlin 的扩展（Extension），主要分为两种语法：第一个是扩展函数，第二个是扩展属性。从语法上看，扩展看起来就像是我们从类的外部为它扩展了新的成员。

借助 Kotlin 的扩展函数，我们完全可以在语义层面，来为第三方 SDK 的类扩展新的成员方法和成员属性。不管是为 JDK 的 String 增加新的成员方法，还是为 Android SDK 的 View 增加新成员属性，我们都可以实现。

## 扩展函数

```kotlin
// Ext.kt
package com.boycoder.chapter06

/*
 ①    ②      ③            ④
 ↓     ↓       ↓            ↓      */
fun String.lastElement(): Char? {
    //    ⑤
    //    ↓
    if (this.isEmpty()) {
        return null
    }

    return this[length - 1]
}

// 使用扩展函数
fun main() {
    val msg = "Hello Wolrd"
    // lastElement就像String的成员方法一样可以直接调用
    val last = msg.lastElement() // last = d
}
```

- 注释 ①，fun 关键字，代表我们要定义一个函数。也就是说，不管是定义普通 Kotlin 函数，还是定义扩展函数，我们都需要 fun 关键字。
- 注释 ②，“String.”，代表我们的扩展函数是为 String 这个类定义的。在 Kotlin 当中，它有一个名字，叫做接收者（Receiver），也就是扩展函数的接收方。
- 注释 ③，lastElement()，是我们定义的扩展函数的名称。
- 注释 ④，“Char?”，代表扩展函数的返回值是可能为空的 Char 类型。
- 注释 ⑤，“this.”，代表“具体的 String 对象”，当我们调用 msg.lastElement() 的时候，this 就代表了 msg。

如果我们在普通函数的名称前面加上一个“接收者类型”，比如“String.”，Kotlin 的“普通函数”就变成了“扩展函数”。

上述代码编译后的 Java 代码如下：

```Java
public final class ExtKt {
   // ①
   public static final Character lastElement(String $this) {
      CharSequence var1 = (CharSequence)$this;
      if (var1.length() == 0) {
        return null
      }

      return  var1.charAt(var1.length() - 1);
   }
}

public static final void main() {
  String msg = "Hello Wolrd";
  //                        ②
  //                        ↓
  Character last = ExtKt.lastElement(msg);
}
```

可以看到，由于 JVM 不理解 Kotlin 的扩展语法，所以 Kotlin 编译器会将扩展函数转换成对应的静态方法，而扩展函数调用处的代码也会被转换成静态方法的调用。

## 扩展属性

类似的，扩展属性，也就是在普通属性定义的时候多加一个“接收者类型”即可。

```kotlin
// 接收者类型
//     ↓
val String.lastElement: Char?
    get() = if (isEmpty()) {
            null
        } else {
            get(length - 1)
        }

fun main() {
    val msg = "Hello Wolrd"
    // lastElement就像String的成员属性一样可以直接调用
    val last = msg.lastElement // last = d
}
```

以上的代码进行反编译，你会发现它反编译后的 Java 代码几乎和前面扩展函数的一模一样。

Kotlin 的扩展表面上看起来是为一个类扩展了新的成员，但是本质上，它还是静态方法。而且，不管是扩展函数还是扩展属性，它本质上都会变成一个静态的方法。那么，到底什么时候该用扩展函数，什么时候该用扩展属性呢？

其实，我们只需要看扩展在语义上更适合作为函数还是属性就够了。比如这里的 lastElement，它更适合作为一个扩展属性。这样设计的话，在语义上，lastElement 就像是 String 类当中的属性一样，它代表了字符串里的最后一个字符。

## 扩展的应用场景和限制

在 Kotlin 当中，几乎所有的类都可以被扩展，包括普通类、单例类、密封类、枚举类、伴生对象，甚至还包括第三方提供的 Java 类。唯有匿名内部类，由于它本身不存在名称，我们无法指定“接收者类型”，所以不能被扩展，当然了，它也没必要被扩展。

可以说，Kotlin 扩展的应用范围还是非常广的。它最主要的用途，就是用来取代 Java 当中的各种工具类，比如 StringUtils、DateUtils 等等。

Kotlin 扩展的第一个典型使用场景：关注点分离。所谓关注点分离，就是将我们程序的逻辑划分成不同的部分，每一个部分，都只关注自己那部分的职责。以 Kotlin 源码中的 String 类为例，String.kt 这个类，只关注 String 的核心逻辑；而 Strings.kt 则只关注 String 的操作符逻辑。

扩展第二个使用场景：借助扩展函数，我们不仅提升了代码的可读性，还提升了编码效率，而这种效率可以说是成倍的提升。

总的来说，Kotlin 扩展主要有两个核心使用场景：

- 主动使用扩展，通过它来优化软件架构。
  - 对复杂的类进行职责划分，关注点分离。让类的核心尽量简单易懂，而让类的功能性属性与方法以扩展的形式存在于类的外部。比如我们的 String.kt 与 Strings.kt。
- 被动使用扩展，提升可读性与开发效率。
  - 当我们无法修改外部的 SDK 时，对于重复的代码模式，我们将其以扩展的方式封装起来，提供给对应的接收者类型，比如 view.updateMargin()。

扩展的限制如下：

- 第一个限制，Kotlin 扩展不是真正的类成员，因此它无法被它的子类重写；
- 第二个限制，扩展属性无法存储状态。就如前面代码当中的 isAdult 属性一般，它的值是由 age 这个成员属性决定的，它本身没有状态，也无法存储状态。背后的根本原因，还是因为它们都是静态方法；
- 第三个限制，扩展的访问作用域仅限于两个地方。第一，定义处的成员；第二，接收者类型的公开成员。也就是说它是无法访问接受者的私有成员的。
  - 如果扩展是顶层的扩展，那么扩展的访问域仅限于该 Kotlin 文件当中的所有成员，以及被扩展类型的公开成员，这种方式定义的扩展是可以被全局使用的。
  - 如果扩展是被定义在某个类当中的，那么该扩展的访问域仅限于该类当中的所有成员，以及被扩展类型的公开成员，这种方式定义的扩展仅能在该类当中使用。

![](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1692802263.63904SmartPic.png)