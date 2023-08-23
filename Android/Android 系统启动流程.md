# Android 系统启动流程

1. 按下电源；
2. 加载 BootLoader 到 RAM；
3. BootLoader 加载系统；
4. Linux 内核启动，设置缓存、被保护存储器、计划列表、加载驱动，完成系统设置后启动 init 进程；
5. init 进程，初始化、启动属性服务，启动 Zygote 进程；
6. Zygote 进程，创建 JavaVM，并为其注册 JNI，创建服务端 Socket，启动 SystemServer 进程；
7. SystemServer 进程，启动 Binder 线程池和 SystemServiceManager，启动各种系统服务；
8. ActivityManagerService 启动 Launcher，Launcher 启动后将已安装应用显示到界面上。

![](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1692801387.3806899SmartPic.png)