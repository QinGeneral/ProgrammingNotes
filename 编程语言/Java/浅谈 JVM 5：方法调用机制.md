# 浅谈 JVM 5：方法调用机制

经历了前文中类加载机制、JVM 逻辑区域划分和运行时栈帧及方法字节码执行过程等内容的沉淀之后，我们这次聊聊 JVM 中方法调用是如何发生的，以及静态绑定和动态绑定是什么。

先从一段 Java 代码及其对应字节码看起。

```java
// java
private static int methodA() {
    return 100;
}

private void methodB() {
    methodA();
}
// methodB's bytecode
0 invokestatic #7 <InvokeDemo.methodA : ()I>
3 pop
4 return
```

Java 代码部分比较简单，在方法 `methodB` 中调用了 `methodA`。在 `methodB` 字节码中，`invokestatic #7` 部分对应了方法调用功能。我们可以发现，代表了目标方法的 `#7` 符号包含类名 `InvokeDemo`、方法名 `methodA` 以及方法描述符 `()I`，即方法参数类型和返回值类型。

由此我们知道，`invokestatic` 方法调用指令需要类名、方法名、方法描述符（方法参数类型 + 返回值类型）这些信息来识别一个方法。其实除了此指令外，其他方法调用指令也都需要这些信息，无论是重载的方法还是重写的方法。对于重载的方法，JVM 在编译期就可以完成方法符号的解析和识别，所以对于 JVM 来说不存在重载的概念。而对于重写方法的符号解析需要在运行期确定。

类似于重载方法这种在编译期就可以解析的方法，可以称之为静态绑定；对于重写这种在运行时才能解析的方法，可以称之为动态绑定。

## 方法调用指令

除了 `invokestatic` 之外，方法调用还有其他几种指令。同样，我们先来看一段示例代码。

```java
public class InvokeDemo {
    private static int methodA() {
        return 100;
    }

    private void methodB() {
        methodA(); // invokestatic
    }

    private void methodC() {
        methodB(); // invokevirtual
    }

    interface IMethod {
        void methodD();
    }

    class MethodImpl implements IMethod {
        @Override
        public void methodD() {
        }
    }

    private void methodE() {
        IMethod iMethod = new MethodImpl(); // invokespecial
        iMethod.methodD(); // invokeinterface
    }

    private void methodF() {
        new Thread(
                () -> {} // invokedynamic
        );
    }
}
```

从上述代码可以看出，方法调用指令包括：
- `invokestatic`：静态方法的调用；
- `invokespecial`：构造器、私有实例等方法的调用；
- `invokevirtual`：非私有实例方法的调用；
- `invokeinterface`：接口方法的调用。
- `invokedynamic`：lambda 表达式等调用。

按照绑定类型分类如下：
- 静态绑定：`invokestatic`、`invokespecial`
- 动态绑定：`invokevirtual`、`invokeinterface`

其中 `invokedynamic` 指令较为特殊，在此先不谈及。

对于 `invokevirtual` 和 `invokeinterface`，大部分情况为动态绑定。但如果方法能够唯一确定，比如被 `final` 修饰，就可以使用静态绑定直接解析。

## 方法调用机制

### 方法符号解析

根据上文我们知道，对一个方法的定位需要类名、方法名、方法描述符（方法参数类型 + 返回值类型）信息。以一个类方法的方法定位为例，其过程如下：
1. 在调用类中，根据方法名和描述符查找方法；
2. 未找到时，递归查找它的父类；
3. 仍未找到时，在其接口中查找。

经过以上过程，方法的符号引用被解析为实际引用。对于静态绑定，实际引用为指向方法的的指针；对于动态绑定，实际引用为方法表的索引。

### 方法定位与方法表

我们以 `invokevirtual` 和 `invokeinterface` 两个指令为例，谈谈动态绑定方法的调用。

JVM 采用使用空间换取时间的方式来实现动态绑定，为每个类生成对应方法表来定位目标方法。方法表根据指令分为 `invokevirtual` 的虚方法表 vtable 和 `invokeinterface` 的接口方法表 itable。其中的每个元素指向当前类或祖先类的方法。

方法表的特征要求如下：
1. 跟 Java 的继承特征一致，子类方法表包含父类方法表的所有方法；
2. 子类方法的索引和父类对应方法的索引一致。

### 方法调用

那么，动态绑定方法的调用过程如下：
1. Java 栈中调用对应指令，如 `invokestatic #7 <InvokeDemo.methodA : ()I>`；
2. 找到方法的调用者——Java 堆中的类；
3. 访问该类对应的方法表；
4. 找到方法的地址，调用方法。

除此之外，JVM 还使用了内联缓存、方法内联的技术来优化动态绑定方法的调用。内联缓存就是将动态绑定方法解析后的类型和目标方法缓存起来，下一次碰到相同类型直接使用。仍然是空间换时间的策略。

## 总结

至此，我们可以在脑中构建出 JVM 运行字节码的大致流程。从外部的字节码结构和解读方法，到虚拟机加载字节码流的过程，然后是虚拟机逻辑区域划分和每部分的功能，以及 Java 栈中的计算过程，最后是具体方法调用指令的解析、调用机制。

当然，JVM 相关的内容不止这些。比如 Java 堆的内存管理和回收机制、JVM 的启动过程、JNI 调用机制等，我们还未涉及。

另外，当初使用 Java 作为官方语言的 Android，其内部在前期使用的 Dalvik 虚拟机和后来的 ART 虚拟机在结构和运行字节码上又有何不同？当下力推的 Kotlin 在虚拟机的执行上和 Java 有什么不同吗？