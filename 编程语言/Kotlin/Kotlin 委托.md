# Kotlin 委托

## 委托类

UniversalDB 实现了 DB 接口，同时通过 by 这个关键字，将接口的实现委托给了它的参数 db。

```kotlin
interface DB {
    fun save()
}

class SqlDB() : DB {
    override fun save() { println("save to sql") }
}

class GreenDaoDB() : DB {
    override fun save() { println("save to GreenDao") }
}
//               参数  通过 by 将接口实现委托给 db
//                ↓            ↓
class UniversalDB(db: DB) : DB by db

fun main() {
    UniversalDB(SqlDB()).save()
    UniversalDB(GreenDaoDB()).save()
}

/*
输出：
save to sql
save to GreenDao
*/
```

上述代码等价于以下 Java 代码：

```Java
class UniversalDB implements DB {
    DB db;
    public UniversalDB(DB db) { this.db = db; }
             //  手动重写接口，将 save 委托给 db.save()
    @Override//            ↓
    public void save() { db.save(); }
}
```

Kotlin 的委托类提供了语法层面的委托模式。通过这个 by 关键字，就可以自动将接口里的方法委托给一个对象，从而可以帮我们省略很多接口方法适配的模板代码。

## 委托属性

Kotlin“委托类”委托的是接口方法，而“委托属性”委托的，则是属性的 getter、setter。

### 直接委托

```kotlin
class Item {
    var count: Int = 0
    //              ①  ②
    //              ↓   ↓
    var total: Int by ::count
}
```

以上代码定义了两个变量，count 和 total，其中 total 的值与 count 完全一致，因为我们把 total 这个属性的 getter 和 setter 都委托给了 count。

以上代码等价于：

```kotlin
// 近似逻辑，实际上，底层会生成一个Item$total$2类型的delegate来实现

class Item {
    var count: Int = 0

    var total: Int
        get() = count

        set(value: Int) {
            count = value
        }
}
```

这个特性，其实对我们软件版本之间的兼容很有帮助。假设 Item 是服务端接口的返回数据，1.0 版本的时候，我们的 Item 当中只 count 这一个变量：

```kotlin
// 1.0 版本
class Item {
    var count: Int = 0
}
```

而到了 2.0 版本的时候，我们需要将 count 修改成 total，这时候问题就出现了，如果我们直接将 count 修改成 total，我们的老用户就无法正常使用了。但如果我们借助委托，就可以很方便地实现这种兼容。我们可以定义一个新的变量 total，然后将其委托给 count，这样的话，2.0 的用户访问 total，而 1.0 的用户访问原来的 count，由于它们是委托关系，也不必担心数值不一致的问题。

### 懒加载委托

懒加载，顾名思义，就是对于一些需要消耗计算机资源的操作，我们希望它在被访问的时候才去触发，从而避免不必要的资源开销。这也是软件设计里十分常见的模式。

```kotlin
//            定义懒加载委托
//               ↓   ↓
val data: String by lazy {
    request()
}

fun request(): String {
    println("执行网络请求")
    return "网络数据"
}

fun main() {
    println("开始")
    println(data)
    println(data)
}

结果：
开始
执行网络请求
网络数据
网络数据
```

如果你去看懒加载委托的源代码，你会发现它其实是一个高阶函数：

```kotlin
public actual fun <T> lazy(initializer: () -> T): Lazy<T> = SynchronizedLazyImpl(initializer)


public actual fun <T> lazy(mode: LazyThreadSafetyMode, initializer: () -> T): Lazy<T> =
    when (mode) {
        LazyThreadSafetyMode.SYNCHRONIZED -> SynchronizedLazyImpl(initializer)
        LazyThreadSafetyMode.PUBLICATION -> SafePublicationLazyImpl(initializer)
        LazyThreadSafetyMode.NONE -> UnsafeLazyImpl(initializer)
    }
```

可以看到，lazy() 函数可以接收一个 LazyThreadSafetyMode 类型的参数，如果我们不传这个参数，它就会直接使用 SynchronizedLazyImpl 的方式。而且通过它的名字我们也能猜出来，它是为了多线程同步的。而剩下的 SafePublicationLazyImpl、UnsafeLazyImpl，则不是多线程安全的。

### 自定义委托

为了自定义委托，我们必须遵循 Kotlin 制定的规则。

```kotlin
class StringDelegate(private var s: String = "Hello") {
//     ①                           ②                              ③
//     ↓                            ↓                               ↓
    operator fun getValue(thisRef: Owner, property: KProperty<*>): String {
        return s
    }
//      ①                          ②                                     ③
//      ↓                           ↓                                      ↓
    operator fun setValue(thisRef: Owner, property: KProperty<*>, value: String) {
            s = value
    }
}

//      ②
//      ↓
class Owner {
//               ③
//               ↓
    var text: String by StringDelegate()
}
```

而如果你觉得这样的写法实在很繁琐，也可以借助 Kotlin 提供的 ReadWriteProperty、ReadOnlyProperty 这两个接口，来自定义委托。

```kotlin
public fun interface ReadOnlyProperty<in T, out V> {
    public operator fun getValue(thisRef: T, property: KProperty<*>): V
}

public interface ReadWriteProperty<in T, V> : ReadOnlyProperty<T, V> {
    public override operator fun getValue(thisRef: T, property: KProperty<*>): V

    public operator fun setValue(thisRef: T, property: KProperty<*>, value: V)
}
```

如果我们需要为 val 属性定义委托，我们就去实现 ReadOnlyProperty 这个接口；如果我们需要为 var 属性定义委托，我们就去实现 ReadWriteProperty 这个接口。这样做的好处是，通过实现接口的方式，IntelliJ 可以帮我们自动生成 override 的 getValue、setValue 方法。

### 提供委托（provideDelegate）

我们希望 StringDelegate(s: String) 传入的初始值 s，可以根据委托属性的名字的变化而变化。我们应该怎么做？

实际上，要想在属性委托之前再做一些额外的判断工作，我们可以使用 provideDelegate 来实现。

```kotlin
class SmartDelegator {

    operator fun provideDelegate(
        thisRef: Owner,
        prop: KProperty<*>
    ): ReadWriteProperty<Owner, String> {

        return if (prop.name.contains("log")) {
            StringDelegate("log")
        } else {
            StringDelegate("normal")
        }
    }
}

class Owner {
    var normalText: String by SmartDelegator()
    var logText: String by SmartDelegator()
}

fun main() {
    val owner = Owner()
    println(owner.normalText)
    println(owner.logText)
}

结果：
normal
log
```

可以看到，为了在委托属性的同时进行一些额外的逻辑判断，我们使用创建了一个新的 SmartDelegator，通过它的成员方法 provideDelegate 嵌套了一层，在这个方法当中，我们进行了一些逻辑判断，然后再把属性委托给 StringDelegate。

如此一来，通过 provideDelegate 这样的方式，我们不仅可以嵌套 Delegator，还可以根据不同的逻辑派发不同的 Delegator。