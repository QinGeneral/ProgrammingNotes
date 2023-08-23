# Kotlin 高阶函数

## 函数类型

将函数的“参数类型”和“返回值类型”抽象出来后，就得到了“函数类型”。(Int, Int) ->Float 就代表了参数类型是两个 Int，返回值类型为 Float 的函数类型。

## 高阶函数

高阶函数是将函数用作参数或返回值的函数。

```kotlin
//                      函数作为参数的高阶函数
//                              ↓
fun setOnClickListener(l: (View) -> Unit) { ... }
```

## 高阶函数和 Lambda 表达式

高阶函数可以大大简化 setOnClickListener 的使用，比如说：

在 Android 中，Java 代码可以通过 Lambda 表达式简化 setOnClickListener 写法。原因是这样的，由于 OnClickListener 符合 SAM 转换的要求，因此编译器自动帮我们做了一层转换，让我们可以用 Lambda 表达式来简化我们的函数调用。

SAM 是 Single Abstract Method 的缩写，意思就是只有一个抽象方法的类或者接口。但在 Kotlin 和 Java 8 里，SAM 代表着只有一个抽象方法的接口。只要是符合 SAM 要求的接口，编译器就能进行 SAM 转换，也就是我们可以使用 Lambda 表达式，来简写接口类的参数。

Java 8 中的 SAM 有明确的名称，叫做函数式接口（FunctionalInterface）。FunctionalInterface 的限制如下，缺一不可：

- 必须是接口，抽象类不行；
- 该接口有且仅有一个抽象的方法，抽象方法个数必须是 1，默认实现的方法可以有多个。

## 高阶函数的本质

Kotlin 引入全新的高阶函数，最终变成 JVM 字节码后是通过匿名内部类来实现的。

## 带接受者的函数类型

```kotlin
User.()->Unit
```

![](https://static001.geekbang.org/resource/image/9a/04/9acd54ac08d88c94ca52336a576b3304.png?wh=1890x1156)

从外表上看，带接收者的函数类型，就等价于成员方法。但从本质上讲，它仍是通过编译器注入 this 来实现的。

## 总结

总的来说：

- 将函数的参数类型和返回值类型抽象出来后，我们就得到了函数类型。比如(View) -> Unit 就代表了参数类型是 View，返回值类型为 Unit 的函数类型。
- 如果一个函数的“参数”或者“返回值”的类型是函数类型，那这个函数就是高阶函数。
- Lambda 就是函数的一种简写。
