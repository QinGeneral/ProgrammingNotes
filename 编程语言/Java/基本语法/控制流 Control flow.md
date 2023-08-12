# 控制流 Control flow

## 条件控制 if-else

if-else 语句的语法如下：

```java
if (bool_expression) {
    // do something
} else if (bool_expression) {
    // do something
} else {
    // do something
}
```

`bool_expression` 是一个布尔表达式，它的值为 `true` 或 `false`。如果 `bool_expression` 的值为 `true`，则执行 `if` 语句块中的代码；如果 `bool_expression` 的值为 `false`，则执行 `else` 语句块中的代码。

以下 Java 代码在输入大于 10 时，输出 `a > 10`，否则输出 `a <= 10`。

```java
public class Main {
    public static void main(String[] args) {
        int a = Math.random() * 20;
        if (a > 10) {
            System.out.println("a > 10");
        } else {
            System.out.println("a <= 10");
        }
    }
}
```

## switch

当存在多种条件时，使用 if-else 语句会显得很冗长。switch 语句可以更好地解决这个问题，switch 是实现多路选择的干净简洁的方式。

switch 语句的语法如下：

```java
switch (expression) {
    case value1:
        // do something
        break;
    case value2:
        // do something
        break;
    default:
        // do something
        break;
}
```

switch 选择器的类型可以是 byte、short、int、char、String 或枚举类型。switch 语句中的每个 case 常量值必须是唯一的。

switch 的执行过程如下：

1. 选择器表达式的值与每个 case 常量值进行比较，如果相等，则执行该 case 语句块中的代码；
2. 如果没有 case 常量值与选择器表达式的值相等，则执行 default 语句块中的代码；
3. 如果没有 default 语句块，则 switch 语句结束；
4. 如果 case 语句块中没有 break 语句，则会继续执行下一个 case 语句块中的代码；
5. 如果 case 语句块中有 break 语句，则 switch 语句结束。

## 循环控制 

### while、do-while

while 语句的语法如下：

```java
while (bool_expression) {
    // do something
}
```

如果 `bool_expression` 的值为 `true`，则继续执行 `while` 语句块中的代码；如果 `bool_expression` 的值为 `false`，则跳过 `while` 语句块中的代码。

以下 Java 代码输出 0 到 9。

```java
public class Main {
    public static void main(String[] args) {
        int i = 0;
        while (i < 10) {
            System.out.println(i);
            i++;
        }
    }
}
```

do-while 语句的语法如下：

```java
do {
    // do something
} while (bool_expression);
```

do-while 和 while 的区别在于，do-while 语句块中的代码至少会执行一次。在日常使用中，while 语句的使用频率更高。

### for

for 语句的语法如下：

```java
for (initialization; bool_expression; step) {
    // do something
}
```

`initialization` 是初始化语句，`bool_expression` 是布尔表达式，`step` 是步进语句。for 语句的执行过程如下：

1. 执行 `initialization` 语句；
2. 如果 `bool_expression` 的值为 `true`，则执行 `for` 语句块中的代码，然后执行 `step` 语句；
3. 如果 `bool_expression` 的值为 `false`，则跳过 `for` 语句块中的代码，执行 `for` 语句后面的代码；
4. 重复步骤 2 和 3，直到 `bool_expression` 的值为 `false`。

初始化语句和步进语句可以是多条语句，多条语句之间用逗号操作符分隔。如下：
    
```java
for (int i = 0, j = 0; i < 10; i++, j++) {
    // do something
}
```

初始化语句和步进语句也可以为空。

以下 Java 代码输出 0 到 9。

```java
public class Main {
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            System.out.println(i);
        }
    }
}
```

### for-each

for-each 在其他语言中成为 for-in，它的语法如下：

```java
float[] f;
...
for (float x: f) {
   // do something
}
```

for-each 语句的执行过程如下：

1. 从 `f` 中取出一个元素，赋值给 `x`；
2. 执行 `for` 语句块中的代码；
3. 重复步骤 1 和 2，直到 `f` 中的元素全部取出。

以下 Java 代码输出 0 到 9。

```java
public class Main {
    public static void main(String[] args) {
        int[] i = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
        for (int x: i) {
            System.out.println(x);
        }
    }
}
```

for-each 语句可以处理任何 Iterable 对象。

## break、continue、return

break 语句用于跳出循环，它的语法如下：

```java
while (bool_expression) {
    // do something
    if (bool_expression) {
        break;
    }
}
```

continue 语句用于跳过循环中的一次迭代，它的语法如下：

```java
while (bool_expression) {
    // do something
    if (bool_expression) {
        continue;
    }
}
```

return 语句除了会打断当前程序的执行，还可以设置方法中的返回值，它的语法如下：

```java
public int foo() {
    // do something
    if (bool_expression) {
        return 0;
    }
    // do something
    return 1;
}
```

## label 标签

Java 中虽然将 goto 作为了保留字，但是并没有实现 goto 语句。而是提供了 label 标签机制。

label 标签的语法如下：

```java
label1:
for (int i = 0; i < 10; i++) {
    for (int j = 0; j < 10; j++) {
        ...
        break;
        ...
        continue;
        ...
        continue label1;
        ...
        break label1;
    }
}
```

label 标签的作用是标记一个代码块，然后在 break 或 continue 语句中使用。`break lable1` 语句会跳出两个循环，不再进入循环执行；`continue label1` 语句会跳出两个循环，重新进入循环执行。

使用 Java 标签的唯一场景是用到了嵌套循环，而且需要使用 break 和 continue 跳出多层嵌套。