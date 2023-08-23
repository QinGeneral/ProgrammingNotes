# Java Stream 操作

Ref：《Java 性能调优实战》[06 | Stream如何提高遍历集合效率？-极客时间](https://time.geekbang.org/column/article/98582)
Day：2021-12-12

Stream 是一种函数式编程风格。

少量数据时，Steam 操作和并行执行并不能提高执行效率。大量数据有效果。

官方将 Stream 中的操作分为两大类：中间操作（Intermediate operations）和终结操作（Terminal operations）。中间操作只对操作进行了记录，即只会返回一个流，不会进行计算操作，而终结操作是实现了计算操作。

中间操作又可以分为无状态（Stateless）与有状态（Stateful）操作，前者是指元素的处理不受之前元素的影响，后者是指该操作只有拿到所有元素之后才能继续下去。

终结操作又可以分为短路（Short-circuiting）与非短路（Unshort-circuiting）操作，前者是指遇到某些符合条件的元素就可以得到最终结果，后者是指必须处理完所有元素才能得到最终结果。操作分类详情如下图所示：

![](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1692788251.72632SmartPic.png)

管道结构通常是由 ReferencePipeline 类实现的，前面讲解 Stream 包结构时，我提到过 ReferencePipeline 包含了 Head、StatelessOp、StatefulOp 三种内部类。

Head 类主要用来定义数据源操作，在我们初次调用 names.stream() 方法时，会初次加载 Head 对象，此时为加载数据源操作；接着加载的是中间操作，分别为无状态中间操作 StatelessOp 对象和有状态操作 StatefulOp 对象，此时的 Stage 并没有执行，而是通过 AbstractPipeline 生成了一个中间操作 Stage 链表；当我们调用终结操作时，会生成一个最终的 Stage，通过这个 Stage 触发之前的中间操作，从最后一个 Stage 开始，递归产生一个 Sink 链。如下图所示：

![](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1692788238.1624162SmartPic.png)