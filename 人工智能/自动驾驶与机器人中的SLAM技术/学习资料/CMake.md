# 什么是CMake？为什么要用CMake？

![Alt text](../Fig/image-5.png)

这里引用了知乎上一个生动的例子：
你开了一家包子铺，打算做包子生意。（程序员）随着你的客流量越来越大，你逐渐感觉有些力不从心。每次做包子都要手动活面，拌馅，千篇一律。（手动输指令编译链接）于是你就想弄一个机器，自动帮你进行这些千篇一律的操作，你只需要打开开关按下按钮就可以了。（机器：Makefile，按钮：make指令）随着你的效率提升了，你想在其他地方开连锁店（不同的操作系统）。但是机器不好使了，某个地方没有小龙虾，只有小凤虾（比如Windows下面部分API在Linux中长另一个样），而另一个地方没有玉米棒子，但用豌豆也能获得差不多的口感（某个库没有对应的平台版本，但有些替代品提供了相同的接口）。由此一来你就得根据不同的地域条件制作不同的机器（编写不同的Makefile），才能采摘原料，做出包子成品（正确运行make指令）。这个时候你想给机器外挂一个自动控制系统，只要输入包子配方以及各种原料在不同地方的替代品，控制系统就能自动设置好机器并开始生产。这样，你只需要做出一份控制系统就可以了。（cmake的用途：根据不同平台生成对应的Makefile，然后你就可以使用make指令快速便捷的编译生成你需要的程序）






**如果想在linux系统上对众多cpp文件编译和链接驾轻就熟，请一定按照下面的例程自己动手构建一遍。下面的每个学习链接都可以帮助你构建知识体系，更为重要的是选择一个，坚持学完！！！**

[在线链接](https://gavinliu6.github.io/CMake-Practice-zh-CN/#/better-hello-world)
[备用 CMake Practice PDF](./CMake_Practice.pdf)（推荐）

[bilibili视频学习](https://www.bilibili.com/video/BV1vR4y1u77h/?spm_id_from=333.337.search-card.all.click)

[github中文](https://github.com/shendeguize/CMakeTutorialCN)

[维基百科](https://zh.wikipedia.org/zh-hans/CMake)



参考链接：https://www.zhihu.com/question/309584159/answer/1426004389