# Kotlin 新特性是如何实现的

在 Java 中，一个对象类型可能为 null。为了保证代码的正常执行，我们经常需要加上判空操作 `if (object != null)`。

在 Kotlin 中，字将变量分为可空类型和不可空类型，以 String 类型为例，`var a: String` 声明不可空类型变量，不可赋值为 null；`var a: String?` 声明可控类型变量，可赋值为 null。

对于可空类型 `var a: String?` 变量访问时，可以加上安全调用符 `?.`，如 `a?.length`，如果 a 为 null，则返回 null，否则返回 a 的长度。

对于这个 Kotlin 语法糖，我们看看 Kotlin 是如何实现的。

Kotlin 代码最终会编译为 Java 字节码，在 JVM 中运行。这就限制了 Kotlin 最终能做的事情是无法超出 JVM 能力边界，Kotlin 在语法层面提供不同于 Java、便捷的语法糖最终也是经过编译期的处理，由相同的 Java 字节码表示，由相同的 JVM 执行。Kotlin 新增的协程机制，则是通过新增了代码库，在语言层面而非操作系统层面，实现了“轻量级线程”的调度功能。

本文以 Kotlin 可空对象的安全访问机制 `a?.` 为例，对比相同功能的 Java 代码，探索验证一下 Kotlin 语法糖是在编译期的处理后，在 Java 字节码层面的实现。我对以下内容中的字节码做了详细注释，不要了解字节码即可阅读。如果读者想对 Java 字节码有一定的了解，可以参考 JVM 系列文章。

我们可以通过 `kotlinc` 命令将 Kotlin 代码编译为 Java 字节码，再通过 `javap` 命令查看 Java 字节码，也可以使用 `jclasslib` 软件来查看 Java 字节码，该软件提供了 `IDEA` 的插件。

以下为示例 Kotlin 和 Java 代码：

```java
// Java 代码
public class Main {
    public static void main(String[] args) {
        String a = null;
        if (args.length > 0) {
            a = "aaa";
        }
        if (a != null) {
            System.out.println(a.length());
        }
    }
}

// Kotlin 代码
fun main(args: Array<String>) {
    var a: String? = null
    if (args.size > 0) {
        a = "aaa"
    }
    print(a?.length)
}
```

上述 Java 代码编译后的字节码如下：

```java
// String a = null; 赋值 a 变量为 null
 0 aconst_null
 1 astore_1
// args.length 加载方法参数 args，获取长度
 2 aload_0
 3 arraylength
// if (args.length > 0) 如果 args 长度小于等于 0，则跳转到 10 行执行
 4 ifle 10 (+6)
// a = "aaa";
 7 ldc #2 <aaa>
 9 astore_1
// if (a != null) 加载 a 变量，判断是否为 null，如果为 null，跳转 24 行结束执行
10 aload_1
11 ifnull 24 (+13)
// System.out 否则，获取 a 的长度并打印
14 getstatic #3 <java/lang/System.out : Ljava/io/PrintStream;>
// a.length()
17 aload_1
18 invokevirtual #4 <java/lang/String.length : ()I>
// out.println
21 invokevirtual #5 <java/io/PrintStream.println : (I)V>
24 return
```

我对 Java 代码和其字节码的对应关系做了注释。可以看出，`if (a != null)` 这行代码被编译为字节码行号为 11 的 `ifnull 24 (+13)`，通过 `ifnull` 指令判断 a 是否为 null，如果为 null，则跳转到 24 行执行，否则继续执行。

Kotlin 代码编译后的字节码如下：

```java
// 无对应 Kotlin 代码，由编译器生成。调用 checkNotNullParameter 方法，检测 main 方法入参是否为 null，为 null 方法内部会抛异常
 0 aload_0
 1 ldc #9 <args>
 3 invokestatic #15 <kotlin/jvm/internal/Intrinsics.checkNotNullParameter : (Ljava/lang/Object;Ljava/lang/String;)V>
// var a: String? = null 赋值 a 变量为 null
 6 aconst_null
 7 astore_1
// args.size 加载 args，判断长度
 8 aload_0
 9 arraylength
// args.size > 0 如果 args 长度小于等于 0，则跳转到 16 行执行
10 ifle 16 (+6)
// a = "aaa" 否则，声明 aaa 字符串，存储到局部变量表 a 变量中
13 ldc #17 <aaa>
15 astore_1
16 aload_1
17 dup
// a?.length 如果变量 a 为空，则跳转到 30 行执行，不为空则获取 a 的长度
18 ifnull 30 (+12)
21 invokevirtual #23 <java/lang/String.length : ()I>
24 invokestatic #29 <java/lang/Integer.valueOf : (I)Ljava/lang/Integer;>
27 goto 32 (+5)
30 pop
31 aconst_null
// print 调用 print 方法，打印 null 或 a 的长度
32 getstatic #35 <java/lang/System.out : Ljava/io/PrintStream;>
35 swap
36 invokevirtual #41 <java/io/PrintStream.print : (Ljava/lang/Object;)V>
39 return
```

可以看出，`a?.length` 这行代码同样被编译为字节码行号为 18 的 `ifnull 30 (+12)`，通过 `ifnull` 指令判断 a 是否为 null，如果为 null，则跳转到 30 行执行，否则继续执行。

对比两者可以看到，Kotlin 可空对象的安全访问 `a?.` 代码最终也编译为类似 Java 的 `ifnull` 指令，来保证代码的正常执行，而不抛出空异常。

Kotlin 的很多其他新特性，也是同样的机制，在编译期做了处理，而在字节码产物和实际运行时中并没有什么不同。感兴趣可以自己实践一下，编译对比一下 Java 字节码。