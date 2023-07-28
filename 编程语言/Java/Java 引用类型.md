# Java 引用类型

[TOC]

不同引用类型会对对象可达性状态和垃圾回收产生影响。

## 强引用

只要强引用存在，垃圾回收器永远不会回收被引用的对象。

## 软引用

只有在内存不足时，垃圾回收器才会回收被引用的对象，在 OOM 抛出之前，会对这些对象进行回收。软引用通常用来实现内存敏感的缓存，内存足够时保留这些缓存，内存不足时会进行回收。

## 弱引用

不会使对象避免垃圾回收，垃圾回收只要发现弱引用，不管内存是否充足，都会回收对象。

检查弱引用指向对象是否被垃圾收集，是判断是否存在特定内存泄露的一种方式。

```java
package org.example;

import java.lang.ref.WeakReference;

public class Main {
    int a = 0;
    public Main(int a) {
        this.a = a;
    }

    @Override
    public String toString() {
        return "a " + this.a;
    }

    public static void main(String[] args) {
        WeakReference<Main> mainWeakReference = new WeakReference<>(new Main(321));
        System.out.println(mainWeakReference + " " + mainWeakReference.get());

        System.gc();

        System.out.println(mainWeakReference + " " + mainWeakReference.get());
    }
}

// 输出
java.lang.ref.WeakReference@18b4aac2 a 321
java.lang.ref.WeakReference@18b4aac2 null
```

## 虚引用（幻象引用）

1. 不能通过它访问对象，可以用来监控对象创建和销毁；
2. 需要结合 ReferenceQueue 使用。

```java
package org.example;

import java.lang.ref.PhantomReference;
import java.lang.ref.ReferenceQueue;

public class Main {
    int a = 0;

    public Main(int a) {
        this.a = a;
    }

    @Override
    public String toString() {
        return "a " + this.a;
    }

    public static void main(String[] args) {
        ReferenceQueue queue = new ReferenceQueue();
        Main main = new Main(321);
        PhantomReference<Main> reference = new PhantomReference<Main>(main, queue);
        System.out.println(reference.get());
        System.out.println(queue.poll());

        main = null;

        System.gc();

        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }

        System.out.println(queue.poll());
    }
}

// 输出
null
null
java.lang.ref.PhantomReference@1dbd16a6
```
