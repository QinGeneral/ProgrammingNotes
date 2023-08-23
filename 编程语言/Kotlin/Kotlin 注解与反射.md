# Kotlin 注解与反射

注解与反射存在的意义是提高代码的灵活性。

## 注解

Kotlin 当中的注解，类似生活中的便利贴一样，其实就是“程序代码的一种补充”。

注解的定义上面，还有可能有其他的注解 @Target、@MustBeDocumented。这样的注解，我们叫做元注解，即它本身是注解的同时，还可以用来修饰其他注解。

Kotlin 常见的元注解有四个：

- @Target，这个注解是指定了被修饰的注解都可以用在什么地方，也就是目标；
- @Retention，这个注解是指定了被修饰的注解是不是编译后可见、是不是运行时可见，也就是保留位置；
- @Repeatable，这个注解是允许我们在同一个地方，多次使用相同的被修饰的注解，使用场景比较少；
- @MustBeDocumented，指定被修饰的注解应该包含在生成的 API 文档中显示，这个注解一般用于 SDK 当中。

## 反射

Kotlin 反射具备这三个特质：

- 感知程序的状态，包含程序的运行状态，还有源代码结构；
- 修改程序的状态；
- 根据程序的状态，调整自身的决策行为。

```kotlin
fun readMembers(obj: Any) {
    obj::class.memberProperties.forEach {
        println("${obj::class.simpleName}.${it.name}=${it.getter.call(obj)}")
    }
}
```

obj::class 是 Kotlin 反射的语法，我们叫做类引用，通过这样的语法，我们就可以读取一个变量的“类型信息”，并且就能拿到这个变量的类型，它的类型是 KClass。

这个 KClass 其实就代表了一个 Kotlin 类，通过 obj::class，我们就可以拿到这个类型的所有信息，比如说，类的名称“obj::class.simpleName”。而如果要获取类的所有成员属性，我们访问它的扩展属性 memberProperties 就可以了。

KClass 代表了一个 Kotlin 的类，下面是它的重要成员：

- simpleName，类的名称，对于匿名内部类，则为 null；
- qualifiedName，完整的类名；
- members，所有成员属性和方法，类型是 Collection<KCallable<\*>>；
- constructors，类的所有构造函数，类型是 Collection<KFunction<T>>>；
- nestedClasses，类的所有嵌套类，类型是 Collection<KClass<\*>>；
- visibility，类的可见性，类型是 KVisibility?，分别是这几种情况，PUBLIC、PROTECTED、INTERNAL、PRIVATE；
- isFinal，是不是 final；
- isOpen，是不是 open；
- isAbstract，是不是抽象的；
- isSealed，是不是密封的；
- isData，是不是数据类；
- isInner，是不是内部类；
- isCompanion，是不是伴生对象；
- isFun，是不是函数式接口；
- isValue，是不是 Value Class。

KCallable 代表了 Kotlin 当中的所有可调用的元素，比如函数、属性、甚至是构造函数。下面是 KCallable 的重要成员：

- name，名称，这个很好理解，属性和函数都有名称；
- parameters，所有的参数，类型是 List<KParameter>，指的是调用这个元素所需的所有参数；
- returnType，返回值类型，类型是 KType；
- typeParameters，所有的类型参数 (比如泛型)，类型是 List<KTypeParameter>；
- call()，KCallable 对应的调用方法，在前面的例子中，我们就调用过 setter、getter 的 call() 方法；
- visibility，可见性；
- isSuspend，是不是挂起函数。

KParameter，代表了 KCallable 当中的参数，它的重要成员如下：

- index，参数的位置，下标从 0 开始；
- name，参数的名称，源码当中参数的名称；
- type，参数的类型，类型是 KType；
- kind，参数的种类，对应三种情况：INSTANCE 是对象实例、EXTENSION_RECEIVER 是扩展接受者、VALUE 是实际的参数值。

KType，代表了 Kotlin 当中的类型，它重要的成员如下：

- classifier，类型对应的 Kotlin 类，即 KClass，我们前面的例子中，就是用的 classifier == String::class 来判断它是不是 String 类型的；
- arguments，类型的类型参数，看起来好像有点绕，其实它就是这个类型的泛型参数；
- isMarkedNullable，是否在源代码中标记为可空类型，即这个类型的后面有没有“?”修饰。

所以，归根结底，反射，其实就是 Kotlin 为我们开发者提供的一个工具，通过这个工具，我们可以让程序在运行的时候“自我反省”。这里的“自我反省”一共有三种情况，其实跟我们的现实生活类似。

- 第一种情况，程序在运行的时候，可以通过反射来查看自身的状态。
- 第二种情况，程序在运行的时候，可以修改自身的状态。
- 第三种情况，程序在运行的时候，可以根据自身的状态调整自身的行为。
