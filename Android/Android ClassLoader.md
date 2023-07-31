# Android ClassLoader

[Java ClassLoader](../编程语言/Java/Java%20ClassLoader.md) 介绍了 Java 中的类加载器，Android 中的类加载器加载的是 dex 字节码，和 Java ClassLoader 有所不同。

- BootClassLoader：ClassLoader 的内部类，Android 系统启动时来预加载常用类；
- DexClassLoader：继承自 BaseDexClassLoader，可以加载 dex 文件和 dex 压缩包；
- PathClassLoader：继承自 BaseDexClassLoader，加载系统类和应用程序类；

![](images/Android%20ClassLoader.png)

SecureClassLoader、URLClassLoader 和 Java ClassLoader 中的类加载器一致，为 ClassLoader 拓展了权限检查方面的安全性和 URL 路径加载类的功能。

IMemoryDexClassLoader 是 Android 8.0 新增的，用于加载内存中的 dex 文件。

## BootClassLoader 加载类流程

1. BaseDexClassLoader 内部加载 dex 文件的代码如下，直接调用 dexPathList.findClass 方法；

```java
130    protected Class<?> findClass(String name) throws ClassNotFoundException {
131        List<Throwable> suppressedExceptions = new ArrayList<Throwable>();
132        Class c = pathList.findClass(name, suppressedExceptions);
133        if (c == null) {
134            ClassNotFoundException cnfe = new ClassNotFoundException(
135                    "Didn't find class \"" + name + "\" on path: " + pathList);
136            for (Throwable t : suppressedExceptions) {
137                cnfe.addSuppressed(t);
138            }
139            throw cnfe;
140        }
141        return c;
142    }
```

2. DexPathList 类中维护了 Element 数组，findClass 时会循环遍历该数组，调用 findClass 方法；

```java
       public Class<?> findClass(String name, List<Throwable> suppressed) {
485        for (Element element : dexElements) {
486            Class<?> clazz = element.findClass(name, definingContext, suppressed);
487            if (clazz != null) {
488                return clazz;
489            }
490        }
491
492        if (dexElementsSuppressedExceptions != null) {
493            suppressed.addAll(Arrays.asList(dexElementsSuppressedExceptions));
494        }
495        return null;
496    }
```

3. 在 Element 的 findClass 方法中，直接调用 dexFile.loadClassBinaryName 方法：

```java
           public Class<?> findClass(String name, ClassLoader definingContext,
723                List<Throwable> suppressed) {
724            return dexFile != null ? dexFile.loadClassBinaryName(name, definingContext, suppressed)
725                    : null;
726        }
```

4. DexFile 的 loadClassBinaryName 方法，会调用 defineClass 方法，最终调用 defineClassNative，进入 native 代码逻辑。