# Kotlin 面向对象编程

[TOC]

在“面向对象”的概念上，虽然 Kotlin 和 Java 之间有一定的语法差异，但底层的思想是没有变的。比如 Java 和 Kotlin 当中，都有类、接口、继承、嵌套、枚举的概念，唯一区别就在于这些概念在两种语言中的具体语法不同。我们需要做的，仅仅只是为我们脑海里已经熟知的概念，再增加一种语法规则而已。

## 类

```kotlin
class Person(val name: String, var age: Int)
```

Kotlin 定义的类，在默认情况下是 public 的，编译器会帮我们生成“构造函数”，对于类当中的属性，Kotlin 编译器也会根据实际情况，自动生成 getter 和 setter。对于 val 变量，只有 getter 方法；对于 var 变量，既有 getter 方法，也有 setter 方法。

Kotlin 自定义属性 getter，虽然这里是自定义属性，但是编译期会自动判断将其编译为一个 JVM 方法：

```kotlin
class Person(val name: String, var age: Int) {
    val isAdult
        get() = age >= 18
//        ↑
//    这就是isAdult属性的getter方法
}
```

Kotlin 自定义属性 setter：

```kotlin
class Person(val name: String) {
    var age: Int = 0
//  这就是age属性的setter
//       ↓
        set(value: Int) {
            log(value)
            field = value
        }
    // 省略
}
```

## 抽象类和继承

一个 Kotlin 抽象类如下：

```kotlin
abstract class Person(val name: String) {
    abstract fun walk()
    // 省略
}
```

Kotlin 的继承直接使用 `:`。

对于重写，Java 当中是使用 @Override 注解，而 Kotlin 当中直接将其定义为了 override 关键字。而除了抽象类以外，正常的类其实也是可以被继承的。不过，我们必须对这个类标记为 open。如果一个类不是抽象类，并且没有用 open 修饰的话，它是无法被继承的。

所以，Kotlin 的类，默认是不允许继承的，除非这个类明确被 open 关键字修饰了。另外，对于被 open 修饰的普通类，它内部的方法和属性，默认也是不允许重写的，除非它们也被 open 修饰了。

```kotlin
open class Person() {
    val canWalk: Boolean = false
    fun walk()
}

class Boy: Person() {
    // 报错
    override val canWalk: Boolean = true
    // 报错
    override fun walk() {
    }
}
```

在继承的行为上面，Kotlin 和 Java 完全相反。Java 当中，一个类如果没有被 final 明确修饰的话，它默认就是可以被继承的。而这同时也就导致了，在 Java 当中“继承”被过度使用。对于这一点，经典书籍《Effective Java》也有提到过。所以，Java 的继承是默认开放的，Kotlin 的继承是默认封闭的。Kotlin 的这个设计非常好，这样就不会出现 Java 中“继承被滥用”的情况。

## 接口和实现

一个 Kotlin 的接口如下：

```kotlin
interface Behavior {
    fun walk()
}

class Person(val name: String): Behavior {
    override fun walk() {
        // walk
    }
    // ...
}
```

Kotlin 接口实现和继承的语法是一样的。

Kotlin 的接口，跟 Java 最大的差异就在于，接口的方法可以有默认实现，同时，它也可以有属性。这一特性是在 Java 1.8 中支持的。比如，我们来看看下面这段代码：

```kotlin
interface Behavior {
    // 接口内的可以有属性
    val canWalk: Boolean

    // 接口方法的默认实现
    fun walk() {
        if (canWalk) {
            // do something
        }
    }
}

class Person(val name: String): Behavior {
    // 重写接口的属性
    override val canWalk: Boolean
        get() = true
}
```

## 嵌套

Java 当中，最常见的嵌套类分为两种：非静态内部类、静态内部类。Kotlin 当中也有一样的概念。

```kotlin
class A {
    val name: String = ""
    fun foo() = 1


    class B {
        val a = name   // 报错
        val b = foo()  // 报错
    }
}
```

以上代码中，B 类，就是 A 类里面的嵌套类，这非常容易理解。不过我们需要注意的是，这种写法的嵌套类，我们无法在 B 类当中访问 A 类的属性和成员方法。

这种写法就对应了 Java 当中的静态内部类。如果我们想要在 B 类当中访问 A 类的属性和成员方法，我们需要使用 inner 关键字，将 B 类标记为内部类。

```kotlin
class A {
    val name: String = ""
    fun foo() = 1

    inner class B {
        val a = name   // 正确
        val b = foo()  // 正确
    }
}
```

Java 的内部类默认是非静态的。Kotlin 则反其道而行之，在默认情况下，嵌套类变成了静态内部类，而这种情况下的嵌套类是不会持有外部类引用的。只有当我们真正需要访问外部类成员的时候，我们才会加上 inner 关键字。这样一来，默认情况下，开发者是不会犯错的，只有手动加上 inner 关键字之后，才可能会出现内存泄漏，而当我们加上 inner 之后，其实往往也就能够意识到内存泄漏的风险了。也就是说，Kotlin 这样的设计，就将默认犯错的风险完全抹掉了。

## Kotlin 中的特殊类

### 数据类

数据类（Data Class），顾名思义，就是用于存放数据的类。要定义一个数据类，我们只需要在普通的类前面加上一个关键字“data”即可。

数据类是为了解决冗余的 Java Bean 而设计的。

```kotlin
    // 数据类当中，最少要有一个属性
                   ↓
data class Person(val name: String, val age: Int)
```

在 Kotlin 当中，编译器会为数据类自动生成一些有用的方法。它们分别是：

- equals()；
- hashCode()；
- toString()；
- componentN() 函数；
- copy()。

“val (name, age) = tom”这行代码，其实是使用了数据类的解构声明。这种方式，可以让我们快速通过数据类来创建一连串的变量。另外，就是 copy 方法。数据类为我们默认实现了 copy 方法，可以让我们非常方便地在创建一份拷贝的同时，修改某个属性。

### 密封类

Kotlin 当中的密封类，常常用来表示某种受到限制的继承结构。这样说起来可能有点抽象，让我们换个说法：密封类，是更强大的枚举类。

当我们尝试去判断枚举的“结构相等”和“引用相等”时，结果始终都是 true。而这就代表了，每一个枚举的值，它在内存当中始终都是同一个对象引用。

如果我们想要枚举的值拥有不一样的对象引用，这时候就需要“密封类”。

想要定义密封类，我们需要使用 sealed 关键字，它的中文含义也代表着“密封”：

```kotlin
sealed class Result<out R> {
    data class Success<out T>(val data: T, val message: String = "") : Result<T>()

    data class Error(val exception: Exception) : Result<Nothing>()

    data class Loading(val time: Long = System.currentTimeMillis()) : Result<Nothing>()
}

fun display(data: Result) = when(data) {
    is Result.Success -> displaySuccessUI(data)
    is Result.Error -> showErrorMsg(data)
    is Result.Loading -> showLoading()
}
```

由于我们的密封类只有这三种情况，所以我们的 when 表达式不需要 else 分支。可以看到，这样的代码风格，既实现了类似枚举类的逻辑完备性，还完美实现了数据结构的封装。

在最新的 Kotlin 1.5 版本当中，sealed 不仅仅可以用于修饰类，还可以用于修饰接口。

## object 关键字

Kotlin 中 object 关键字，有三种迥然不同的语义，分别可以定义：

- 匿名内部类；
- 单例模式；
- 伴生对象。

之所以会出现这样的情况，是因为 Kotlin 的设计者认为，这三种语义本质上都是在定义一个类的同时还创建了对象。

### 匿名内部类

我们在使用 object 定义匿名内部类的时候，其实还可以在继承一个抽象类的同时，来实现多个接口。

```kotlin
interface A {
    fun funA()
}

interface B {
    fun funB()
}

abstract class Man {
    abstract fun findMan()
}

fun main() {
    // 这个匿名内部类，在继承了Man类的同时，还实现了A、B两个接口
    val item = object : Man(), A, B{
        override fun funA() {
            // do something
        }
        override fun funB() {
            // do something
        }
        override fun findMan() {
            // do something
        }
    }
}
```

### 单例模式

在 Kotlin 当中，要实现单例模式其实非常简单，我们直接用 object 修饰类即可：

```kotlin
object UserManager {
    fun login() {}
}
```

我们只需要关注业务逻辑，至于这个单例模式到底是如何实现的，我们交给 Kotlin 编译器就行了。

这种方式定义的单例模式，虽然具有简洁的优点，但同时也存在两个缺点。

- 不支持懒加载；
- 不支持传参构造单例。举个例子，在 Android 开发当中，很多情况下我们都需要用到 Context 作为上下文。另外有的时候，在单例创建时可能也需要 Context 才可以创建，那么如果这时候单纯只有 object 创建的单例，就无法满足需求了。

### 伴生对象

Kotlin 当中没有 static 关键字，所以我们没有办法直接定义静态方法和静态变量。不过，Kotlin 还是为我们提供了伴生对象，来帮助实现静态方法和变量。

```kotlin
class Person {
    object InnerSingleton {
        fun foo() {}
    }
}
```

此类的调用如下：

```kotlin
// Kotlin当中这样调用
Person.InnerSingleton.foo()
//      等价
//       ↓  java 当中这样调用
Person.InnerSingleton.INSTANCE.foo()
```

如果要实现 Java 中同样的调用，可以使用 @JvmStatic 注解。

```kotlin
class Person {
//  改动在这里
//     ↓
    companion object InnerSingleton {
        @JvmStatic
        fun foo() {}
    }
}
```

companion object，在 Kotlin 当中就被称作伴生对象，它其实是我们嵌套单例的一种特殊情况。也就是，在伴生对象的内部，如果存在“@JvmStatic”修饰的方法或属性，它会被挪到伴生对象外部的类当中，变成静态成员。

嵌套单例，是 object 单例的一种特殊情况；伴生对象，是嵌套单例的一种特殊情况。