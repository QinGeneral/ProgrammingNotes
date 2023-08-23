# Kotlin 基本语法

[TOC]

## 变量

```kotlin
关键词 变量名: 类型 = 初始值

var price: Int = 100;

var price = 100
```

关键词 var 用于定义可变变量，val 用于定义不可变变量，相当于 Java 的 final。要尽可能的使用 val。

## 基础类型

Kotlin 中一切都是对象。在 Kotlin 当中，Any 是所有类型的父类，我们可以称之为根类型。任何类型，当它被“?”修饰，变成可空类型以后，它就变成原本类型的父类了。所以，从某种程度上讲，我们可以认为“Any？”是所有 Kotlin 类型的根类型。

但为了性能考虑，对于非空原始类型，它通过编译期的操作，会将其编译为原始类型。

![](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1692787377.87658SmartPic.png)

### 空安全

Kotlin 将变量分为可空类型和不可空类型。强制要求开发者在定义变量的时候，指定这个变量是否可能为 null。对于可能为 null 的变量，我们需要在声明的时候，在变量类型后面加一个问号“?”。

```kotlin
var name: String? = null
```

### 数字类型

在数字类型上，Kotlin 和 Java 几乎是一致的，包括它们对数字“字面量”的定义方式。

- 整数默认会被推导为“Int”类型；
- Long 类型，我们则需要使用“L”后缀；
- 小数默认会被推导为“Double”，我们不需要使用“D”后缀；
- Float 类型，我们需要使用“F”后缀；
- 使用“0x”，来代表十六进制字面量；
- 使用“0b”，来代表二进制字面量。

但是，对于数字类型的转换，Kotlin 与 Java 的转换行为是不一样的。Java 可以隐式转换数字类型，而 Kotlin 更推崇显式转换。因为 int、long、float、double 这些类型之间的互相转换是存在精度问题的。尤其是当这样的代码掺杂在复杂的逻辑中时，在碰到一些边界条件的情况下，即使出现了 Bug 也不容易排查出来。

以下代码在 Java 中是可行的，但是在 Kotlin 中无法编译。

```kotlin
val i = 100
val j: Long = i // 编译器报错
```

Kotlin 中，我们需要使用“toXXX()”方法来显式转换数字类型。

```kotlin
val i = 100
val j: Long = i.toLong() // 编译通过
```

### 布尔类型

Kotlin 中布尔类型的变量只有两种值，分别是 true 和 false。布尔类型支持一些逻辑操作，比如说：

- “&”代表“与运算”；
- “|”代表“或运算”；
- “!”代表“非运算”；
- “&&”和“||”分别代表它们对应的“短路逻辑运算”。

### 字符：Char

Char 用于代表单个的字符，比如'A'、'B'、'C'，字符应该用单引号括起来。

同样的，Kotlin 中的 Char 类型也是不允许隐式转换的，我们需要使用“toXXX()”方法来显式转换。

```kotlin
val c: Char = 'A'
val i: Int = c.toInt() // 编译通过
```

### 字符串：String

字符串（String），顾名思义，就是一连串的字符。和 Java 一样，Kotlin 中的字符串也是不可变的。在大部分情况下，我们会使用双引号来表示字符串的字面量，这一点跟 Java 也是一样的。

Kotlin 还提供了简洁的字符串模版，在 Java 当中，我们必须使用两个“+”进行拼接，比如说("Hello" + name + "!")。这样一来，在字符串格式更复杂的情况下，代码就会很臃肿。

```kotlin
val name = "world"
println("Hello, $name!") // 输出 Hello, world!
```

如果我们需要在字符串当中引用更加复杂的变量，则需要使用花括号将变量括起来：

```kotlin
val array = arrayOf("Java", "Kotlin")
print("Hello ${array.get(1)}!")
```

Kotlin 还新增了一个原始字符串，是用三个引号来表示的。它可以用于存放复杂的多行文本，并且它定义的时候是什么格式，最终打印也会是对应的格式。所以当我们需要复杂文本的时候，就不需要像 Java 那样写一堆的加号和换行符了。

```kotlin
val s = """
       当我们的字符串有复杂的格式时
       原始字符串非常的方便
       因为它可以做到所见即所得。 """

print(s)
```

### 数组

在 Kotlin 当中，我们一般会使用 arrayOf() 来创建数组，括号当中可以用于传递数组元素进行初始化，同时，Kotlin 编译器也会根据传入的参数进行类型推导。

```kotlin
val array = arrayOf(1, 2, 3)
```

如果要获取数组的长度，Java 中应该使用“array.length”；但如果是获取 List 的大小，那么 Java 中则应该使用“list.size”。这主要是因为数组不属于 Java 集合。虽然 Kotlin 的数组仍然不属于集合，但它的一些操作是跟集合统一的，我们直接使用 array.size 就能拿到数组的长度。

## 函数

Kotlin 中的函数定义格式如下：

```kotlin
/*
关键字    函数名          参数类型   返回值类型
 ↓        ↓                ↓       ↓      */
fun helloFunction(name: String): String {
    return "Hello $name !"
}/*   ↑
   花括号内为：函数体
*/
```

对于一些比较简单的函数，我们可以简化为单一表达式函数。比如说，上面的函数可以简化为：

```kotlin
fun helloFunction(name: String): String = "Hello $name !"
```

函数调用则直接通过 `helloFunctioin("Kotlin")` 调用。Kotlin 提供了一些新的特性，那就是命名参数。简单理解，就是它允许我们在调用函数的时候传入“形参的名字”。这一特性增加了代码的可读性，在代码参数过多时使用。比如说：

```kotlin
helloFunction(name = "Kotlin")
```

而除了命名参数这个特性，Kotlin 还支持参数默认值，这个特性在参数较多的情况下同样有很大的优势：

```kotlin
fun helloFunction(name: String = "Kotlin"): String = "Hello $name !"
```

## 流程控制

在 Kotlin 当中，流程控制主要有 if、when、for、 while。

### if

Kotlin 中的 if 语句和 Java 中的 if 语句基本一致，但是 Kotlin 中的 if 语句是有返回值的，可以作为表达式使用，这一点和 Scala 语言类似。比如说：

```kotlin
val a = 1
val b = 2
val max = if (a > b) a else b
```

Kotlin 还提供了 Evis 语法：

```kotlin
fun getLength(text: String?): Int {
  return text?.length ?: 0
}
```

### when

when 语句，在程序当中主要也是用于逻辑判断的。当我们的代码逻辑只有两个分支的时候，我们一般会使用 if/else，而在大于两个逻辑分支的情况下，我们使用 when。when 语句有点像 Java 里的 switch case 语句，不过 Kotlin 的 when 更加强大，它同时也可以作为表达式，为变量赋值。比如说：

```kotlin
val i: Int = 1

val message = when(i) {
    1 -> "一"
    2 -> "二"
    else -> "i 不是一也不是二" // 如果去掉这行，会报错
}

print(message)
```

另外，与 switch 不一样的是，when 表达式要求它里面的逻辑分支必须是完整的。举个例子，以上的代码，如果去掉 else 分支，编译器将报错，原因是：i 的值不仅仅只有 1 和 2，这两个分支并没有覆盖所有的情况，所以会报错。

### 循环迭代：while 与 for

首先 while 循环，我们一般是用于重复执行某些代码，它在使用上和 Java 也没有什么区别。但是对于 for 语句，Kotlin 和 Java 的用法就明显不一样了。

Kotlin 的 for 语句更多的是用于“迭代”。

```kotlin
val oneToThree = 1..3 // 代表 [1, 3]

for (i in oneToThree) {
    println(i)
}

// 逆序迭代
for (i in 6 downTo 0 step 2) {
    println(i)
}
```
