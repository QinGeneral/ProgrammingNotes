# 浅谈 JVM 2：如何阅读字节码

上文聊到理解字节码和 JVM 执行过程能够帮助我们日常开发中解决疑难杂症。这次，我们先来看看如何阅读字节码。

字节码文件是我们使用 `javac xxx.java` 编译而来的 `xxx.class` 文件，内容为 8 bit 字节流。`class` 中的数据类型有 `u1`、`u2` 和 `u4`。`u1` 代表该数据占用 1 字节，`u2` 代表 2 字节，以此类推。

每个 `class` 文件包含了一个类、接口、模块的定义。这些字节流所代表的内容结构由一个类似 C 语言结构体 `structure` 的数据结构来定义，该定义在虚拟机规范中给出，结构如下。结合该结构我们即可对照 `class` 字节流反编译出该类的内容。

> 摘录自 [Java Virtual Machine Specification](https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-4.html)。

```c
ClassFile {
    u4             magic;
    u2             minor_version;
    u2             major_version;
    u2             constant_pool_count;
    cp_info        constant_pool[constant_pool_count-1];
    u2             access_flags;
    u2             this_class;
    u2             super_class;
    u2             interfaces_count;
    u2             interfaces[interfaces_count];
    u2             fields_count;
    field_info     fields[fields_count];
    u2             methods_count;
    method_info    methods[methods_count];
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

## 1. 示例代码

本次我们就以简单的 `Demo.java` 来介绍阅读字节码流程和如何理解字节码。请先查看以下代码，

```java
public class Demo {

    private int mThisIsInt = 1024;

    public static void main(String[] args) {
        System.out.println("hello world");
    }

    private int getThisIsInt() {
        return mThisIsInt;
    }
}
```

## 2. 编译为 `class` 字节码

执行 `javac Demo.java` 即可得到 `Demo.class` 文件。这是我们本次要阅读分析的目标文件。

## 3. 解读字节码

#### 使用 `hexdump`

`VSCode` 中提供了 `hexdump` 插件，展示字节码的 16 进制数据。安装该插件后，在 `VSCode` 中我们右键反编译出的 `Demo.class` 文件，选择 `Show Hexdump` 选项，即可看到如下内容。

![](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1642312656.57SmartPic.png)

我们可以开始对照上文中的 `ClassFile Structure` 来解读该内容。

#### `magic` 部分

`ClassFile` 结构告诉我们，第 1 部分内容为 `magic`，`u4` 代表占用四字节，对应 `CAFEBABE` 内容，这部分内容用来标识 Java 字节码文件格式，可以看到 Java 图标的源头在此处。

![](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1642313057.74SmartPic.png)

#### `version` 部分

第 2、3 部分内容为 `minor_version` 和 `major_version`，分别占用 2 字节，对应 `0000` 和 `003A` 部分内容。代表此 `class` 文件主要版本为 58，次要版本为 0。此版本标识和 JDK 版本相关联，具体可参考 [Java Virtual Machine Specification](https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-4.html) 表 4.1-A。

#### `constant_pool` 部分

第 4、5 部分内容为 `constant_pool_count` 和 `constant_pool`。代表常量池相关内容，`constant_pool_count` 标明常量池数量，等于 `constant_pool` 中的 item 数量。`constant_pool` 则用于存储每个常量。其结构为 `cp_info`。内容如下所示。即由两部分组成，1 字节的 `tag`，代表 `cp_info` 类型，比如 `7` 代表 `Class`——类，`9` 代表 `Fieldref`——字段。

```
cp_info {
    u1 tag;
    u1 info[];
}
```

在此以 `Methodref` 为例。一个方法的结构描述如下。

```
CONSTANT_Methodref_info {
    u1 tag;
    u2 class_index;
    u2 name_and_type_index;
}
```

- `tag` 为 `10`，代表 `Methodref` 类型；
- `class_index` 是指向常量池中 `CONSTANT_Class_info` 类型数据的索引，代表当前方法属于哪个类；
- `name_and_type_index` 是指向常量池中 `CONSTANT_NameAndType_info` 类型数据的索引，代表方法的名称和描述符，描述符则描述了方法的参数和返回值类型。

这时直接阅读字节码 16 进制数据已变得繁琐。在此，我们借用另外一个工具，来进一步解释常量池和其索引的方式。

#### 使用 `javap`

执行 `javap -v -p Demo.class` 获取到可读性更强的反编译文件。我们先查看一下 `Constant pool` 中的部分内容。

```
// 省略内容...

Constant pool:
   #1 = Methodref #2.#3 // java/lang/Object."<init>":()V
   #2 = Class #4 // java/lang/Object
   #3 = NameAndType #5:#6 // "<init>":()V
   
// 省略内容...
```

常量池中使用数组保存每个常量的内容。每个常量都有自己的索引，`javap` 反编译出的文件中使用 `#n` 表示。

`#1` 代表一个 `Methodref` 方法类型常量，该常量通过引用 `#2` `#3` 号常量表示自己。

`#2` 表示 `Object` 类。

`#3` 使用 `#5`、`#6` 号常量表示自己。`#5` `#6` 都为 `CONSTANT_Utf8` 类型，数据内容分别为 `<init>` 和 `()V`。代表该方法的名称为 `<init>`，没有参数，返回值类型为 `void`。

由此我们可以得出以下结构。

![](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1642315579.34SmartPic.png)

#### `methods` 部分

之后的 `access_flags`、`this_class` 等部分，分别表示该类的访问标识、父类、接口、字段等信息，解读方法和常量池类似，不再详细列出。

值得一看的是，`method_info` 部分。其中包含了类中所有的方法。方法的描述和前文一致，其中的 `attributes` 属性结构如下。

```
attribute_info {
    u2 attribute_name_index;
    u4 attribute_length;
    u1 info[attribute_length];
}
```

我们方法的代码部分，就在此存储。其 `attribute_name_index` 指向内容为 `Code` 的常量，代表此为代码属性。`attribute_length` 为属性长度，之后的 `info` 数组则存储了方法代码对应字节码。我们以 Java 代码中 `getThisIsInt` 方法为例，其 Java 源代码和对应字节码如下。

```
// Java 代码
private int getThisIsInt() {
    return mThisIsInt;
}

// 字节码
private int getThisIsInt();
descriptor: ()I
flags: (0x0002) ACC_PRIVATE
Code:
  stack=1, locals=1, args_size=1
     0: aload_0
     1: getfield #7 // Field mThisIsInt:I
     4: ireturn
  LineNumberTable:
    line 11: 0
```

我们重点关注 `Code` 部分。

`stack=1` 表示操作数栈的最大深度为 1。

`locals=1` 表示局部变量表的最大槽数为 1。

`args_size=1` 表示方法参数的数量为 1。需要注意的是，`getThisIsInt` 方法没有显示的参数。但是该方法为实例方法，内部可以直接访问 `this`，因此翻译为字节码时，默认添加了 `this` 参数提供方法内访问当前对象的功能。也就是说，方法内可以访问 `this` 是通过给方法添加默认参数实现的。

```
0: aload_0
1: getfield #7 // Field mThisIsInt:I
4: ireturn
```

之后便是代码对应的字节码，这部分实现了加载类字段 `#7` 号常量然后返回的功能。具体的执行流程涉及 JVM 指令集，限于篇幅我们以后有时间再来具体聊聊。

其中行首的 `n:` 代表字节码偏移，等于当前指令之前所有指令占用的存储大小。比如 `getfield #7` 之前的 `aload_0` 指令占用 1 字节，所以其偏移为 1。

`LineNumberTable` 部分则是字节码偏移和 Java 源代码的对应关系。`line 11: 0` 表示，Java 源代码第 11 行 `return mThisIsInt;` 对应代码字节码的 0 号偏移 `aload_0` 开始的字节码部分。

#### 使用 jclasslib

除了 `hexdump` 和 `javap` 之外，我们还可以使用 `jclasslib` 工具来阅读字节码。使用起来更加直观和便捷。比如提供了常量池索引间跳转、查看 JVM 规范、替换操作码等功能。

`jclasslib` 既提供了独立的 app 也提供了 IDEA 插件。点击查看 [jclasslib Github](https://github.com/ingokegel/jclasslib) 和 [IDEA 插件](https://plugins.jetbrains.com/plugin/9248-jclasslib-bytecode-viewer)。下图为 IDEA 插件截图。

![](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1642317646.7SmartPic.png)

## 总结

本次我们了解了如何结合 JVM 规范中的 `ClassFile Structure`，同时使用 `hexdump`、`javap`、`jclasslib` 工具来解读 Java 字节码。

解读过程中，也结合字节码，了解到方法中访问 `this` 的实现方式。