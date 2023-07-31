# Java ClassLoader

- BootstrapClassLoader：C/C++ 实现的类加载器，加载 JVM 运行时核心类；
- ExtensionClassLoader：Java 实现的类加载器，加载 JVM 扩展类；
- AppClassLoader：加载 classpath 中的类。

![](images/Java%20ClassLoader.png)

SecureClassLoader 继承了抽象类ClassLoader，但SecureClassLoader 并不是ClassLoader 的实现类，而是拓展了ClassLoader 类加入了权限方面的功能，加强了ClassLoader的安全性。

URLClassLoader继承自SecureClassLoader，可以通过URL路径从jar文件和文件夹中加载类和资源。

## 双亲委派机制

加载一个类的过程是先上后下。先从当前 ClassLoader 中查找类是否加载，没有加载则去父类加载器中查找，直到 BootstrapClassLoader。如果 BootstrapClassLoader 加载不到，则从上往下委托加载。BootstrapClassLoader 会先去尝试加载，加载不到则向下委托。

双亲委派机制的优点如下：
1. 避免类的重复加载，保证了类的唯一性；
2. 更加安全，双亲委派机制保证类似 String 这些核心类在虚拟机启动时就会加载，防止了核心类被篡改。

```java
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
    // First, check if the class has already been loaded
    Class<?> c = findLoadedClass(name);
    if (c == null) {
        try {
            if (parent != null) {
                // 委托父类加载器加载
                c = parent.loadClass(name, false);
            } else {
                // 去 BootstrapClassLoader 加载器中查找类
                c = findBootstrapClassOrNull(name);
            }
        } catch (ClassNotFoundException e) {
            // ClassNotFoundException thrown if class not found
            // from the non-null parent class loader
        }

        if (c == null) {
            // If still not found, then invoke findClass in order
            // to find the class.
            // 自己加载类
            c = findClass(name);
        }
    }
    return c;
}
```