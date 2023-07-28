# macOS 下载编译 aosp 源码

最新的 aosp 源码是支持最新的 mac sdk 11.0 的，但是 android-9.0.0 代码最高支持到 mac sdk 10.13。

> 下载编译环境：  
> 系统：`macOS Catalina 10.15.6`  
> aosp 目标编译平台：`android-9.0.0_r1`  

[TOC]

## 1. 配置 macOS 环境

由于 Git 对区分大小写比较敏感，而下载和管理 aosp 源码用到了 Git。所以首先要划分一块区分大小写的磁盘出来。

### 创建区分大小写的磁盘

使用一下命令即可创建一块区分大小写、日志式的磁盘。

```shell
hdiutil create -type SPARSE -fs ‘Case-sensitive Journaled HFS+’ -size 200g ~/android.dmg
```

其中 size 根据需要调整，如果只是下载源码，100g 差不多够用。如果要编译就要 200g 及以上。假如后续磁盘空间不够，可以使用以下命令调整已创建的磁盘大小：

> 注：这里需要注意的是，根据系统不同，创建出的磁盘文件后缀不同，分别可能是 `android.dmg.sparseimage` 和 `android.dmg`。所以以下命令执行的对应文件名不一致。  

```shell
hdiutil resize -size <new-size-you-want>g ~/android.dmg.sparseimage
```

可以在 `~/.bash_profile` 中添加以下两个方法来方便的挂载、卸载磁盘，执行方法前可能需要执行一下 `source ~/.bash_profile` 导入方法：

```shell
# mount the android file image
function mountAndroid { hdiutil attach ~/android.dmg.sparseimage -mountpoint /Volumes/android; }

# unmount the android file image
function umountAndroid() { hdiutil detach /Volumes/android; }
```

挂载之后可以在 `Finder` 或者 `磁盘工具` 中看到该磁盘，其格式为 MacOS 扩展、区分大小写、日志式磁盘。如下图所示：

另外，在 macOS 中默认同时打开文件数量上限较低，当我们并行编译 aosp 时，可能会超出此上限。所以如果计划并行编译源码（最好是这样，因为代码量实在太多了），就需要设置文件描述符上限，将以下内容添加到 `~/.bash_profile` 即可，同样的可能需要执行 `source ~/.bash_profile` 来使配置生效：

```shell
# set the number of open files to be 1024
ulimit -S -n 1024
```

至此，关于磁盘文件相关的内容就设置好了。下面来看一下如何安装下载时会用到的软件。

### 安装所需软件

#### 安装 Xcode

执行以下命令即可：

```shell
xcode-select --install
```

#### 使用 homebrew 安装 Git

执行以下命令安装 homebrew：[点击跳转 Homebrew 官网](https://brew.sh/)

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

执行以下命令安装、配置 Git：

```shell
# install git by homebrew
brew install git

# config git user info
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

#### 检查是否已安装 python2.7

aosp 要求 python2.7 版本，python3 则不行。macOS 自带了 python2.7 版本，可以使用以下命令进行检查：

```shell
python --version
```

有以下输出即可：

```shell
Python 2.7.16
```

#### 安装 jdk8

[点击下载 jdk 8](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)，选择 macOS 版本进行下载安装。安装完成之后，运行以下命令进行验证：
```shell
java -version
```

#### 下载 repo 工具

aosp 源码是通过 Git 进行管理的，但是由于代码量巨大，单纯使用 Git  已经无法满足日常便捷操作的目标。所以 Google 以 Git 为基础，使用 Python 编写了 repo 这款工具，让开发者可以更方便轻松的管理源码，[点击查看 repo 使用详情](https://source.android.com/source/developing)。

首先，创建 repo 工具存放文件夹。

```shell
mkdir ~/bin
```

然后，下载 repo，并赋予 repo 可执行权限。

```shell
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo

chmod a+x ~/bin/repo
```

最后，将 repo 工具添加到 PATH 路径，这样我们可以在之后在终端直接进行访问，无需关心 repo 所在路径。在 `~/.bash_profile`  添加以下内容即可：

```shell
export PATH=~/bin:$PATH
```

至此，相关软件安装完毕。接下来开始下载源码。

## 2. 使用 repo 下载源码
### 创建文件夹

首先，进入我们之前创建、并挂载的磁盘，创建并进入 aosp 源码下载文件夹。

```shell
cd /Volumns/android
mkdir aosp
cd aosp
```

### 初始化版本库

然后，初始化一个指定版本的 aosp 源码库。如果命令后跟上 -b 参数，则代表初始化指定版本的源码库，否则为默认的 master 分支。分支列表可以参考 [aosp 代号、标记和细分版本号](https://source.android.com/source/build-numbers#source-code-tags-and-builds)。初始化代码如下：

```shell
# default is master
repo init -u https://android.googlesource.com/platform/manifest

# define 9.0.0_r1 branch
repo init -u https://android.googlesource.com/platform/manifest -b android-9.0.0_r1
```

如果存在网络问题，可以使用清华镜像源，具体可以参考 [AOSP 清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)。这里要注意的是，如果单纯阅读代码使用，使用清华源即可。如果计划编译代码，建议能够使用 Google 官方源就用官方源，因为清华源存在没有同步的问题，可能会造成一些不必要的麻烦。

> tips：  
> 使用清华源尽量在夜间进行，白天清华源请求量大，下载源码十分容易中断。晚上则比较顺畅。  

如有下图输出即为初始化成功：

![initSuccess](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1605290827.63SmartPic.png)

### 下载源码

在已经初始化好的 aosp 源码目录下执行以下命令即可开始下载源码。

```shell
repo sync
```

如下图输出即为下载成功：

![downloadSuccess](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1605290938.12SmartPic.png)

此时，单单下载 android-9.0.0 的代码已经占用 99G 了。如下图所示。

![](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1605302350.04SmartPic.png)

## 3. 构建源码
### aosp 构建系统简介

aosp 原本是使用 make 进行构建源码。但是由于在 Android 上 make 十分缓慢，易出错等等原因，Google 转而使用由 Go 语言编写的 Soong 构建系统。Soong 是 make 构建系统的替代品，make 构建系统使用 .mk 文件进行书写编译规则，Soong 则是使用 Blueprint 的 .bp 文件书写。.bp 文件是一种类似 JSON 的语法结构，更加简单明了。Soong 最终将 .bp 文件编译成 ninja 文件，进而由 ninjia 进行编译。ninjia 是一种追求编译速度的小型构建系统，其设计目标是嵌入到一个高级构建系统中，追求尽可能快的编译速度。其构建文件可以读懂但是并不适合手动编写——类似于汇编语言，一般是通过将其他高级构建系统的构建文件编译为 ninjia 文件后作为输入。

此时读者可能产生疑问，Google 由 make 转 Soong 不可能是一夜之间发生的，那如何兼容两者呢？答案是，针对原有的 .mk 文件，Google 开发了 kati 系统，将其也编译为 ninjia 文件，再交给 ninjia 进行编译。

总的来说，Soong 是通过解析 .bp 文件为 ninjia 文件，将 .mk 文件通过 kati 编译为 ninjia 文件，最后进行构建的。

如果你想了解更多，以下资源可以参考：

- [Android Make Build System](https://android.googlesource.com/platform/build/+/master/README.md)
- [kati](https://github.com/google/kati/blob/master/README.md)
- [Soong](https://android.googlesource.com/platform/build/soong/+/refs/heads/master/README.md)
- [Ninja, a small build system with a focus on speed](https://ninja-build.org/)

### 初始化构建环境

执行 `envsetup.sh` 脚本来初始化环境，`envsetup.sh` 会添加一系列命令可供编译使用。

> 注：编译相关命令需要在 bash shell 下执行，如果你使用了 zsh 等其他 shell，需要执行 `bash` 进入 bash shell 后再运行构建命令。

```shell
source build/envsetup.sh
```

### 选择编译的目标系统

```shell
lunch
```

此时输出如下：

```shell
/Volumes/Windows/aosp » lunch

You're building on Darwin

Lunch menu... pick a combo:
     1. aosp_arm-eng
     2. aosp_arm64-eng
     3. aosp_mips-eng
     4. aosp_mips64-eng
     5. aosp_x86-eng
     6. aosp_x86_64-eng
     7. aosp_car_arm-userdebug
     8. aosp_car_arm64-userdebug
     9. aosp_car_x86-userdebug
     10. aosp_car_x86_64-userdebug
......
Which would you like? [aosp_arm-eng]
```

输入英文或者序号都可以，因为我的电脑 CPU 是 x86 架构，所以这里选择 `aosp_x86-eng` 进行编译，这样运行 Android 虚拟机时，速度会比较快。

### 开始编译

执行 `make` 命令即可开始编译，如果需要并行构建的话，需要添加 -j 参数，示例命令如下：

```shell
make

# simplify
m

# multi task
m -j16
```

在编译的时候会碰到很多问题，其解决方法见下一节一一道来。编译成功最后会打印 `build completed successfully`。

### 运行模拟器

如果刚刚编译完，直接执行 `emulator` 即可。否则需要重新执行 `source build/envsetup.sh; lunch aosp_x86-eng;` 导入 `emulator` 命令。

```shell
emulator
```

最后成功运行起来了自己构建出来的 Android 虚拟机了！

![emualtor](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1605291193.02SmartPic.png)

## 4. 可能碰到的问题及其解决

### 问题一：Could not find a supported mac sdk

问题详情输出如下：

```shell
internal error: Could not find a supported mac sdk: [“10.10” “10.11” “10.12” “10.13”]
ninja: build stopped: subcommand failed.
```

执行 `ls /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs` 发现，MacOSX.sdk 之外，还有一个与其对应的 MacOSX10.15.sdk 软链接。说明本机的 mac sdk  为 10.15 版本。

解决方法：

修改 `build/soong/cc/config/x86_darwin_host.go` 文件，在 `darwinSupportedSdkVersions` 下添加 “10.15”,（记得带英文逗号）。

### 问题二：执行上述操作后，重试编译报错

报错详情如下：

```shell
ld: symbol(s) not found for architecture i386
```

其原因是，aosp 并没有支持所有的 mac sdk 版本。比如说我电脑的 mac sdk 版本 10.15，相关配置文件 `build/soong/cc/config/x86_darwin_host.go` 中的配置如下：
```go
darwinSupportedSdkVersions = []string{
        "10.10",
        "10.11",
        "10.12",
        "10.13",
```

从这里可以看到官方测试通过、支持的 mac sdk 为上述几个版本。而更新的版本是不支持的。

最终在 [Unable to make AOSP systemimage on macOS Mojave](https://groups.google.com/g/android-building/c/f4_tNSapWsA) 找到了解决问题的答案。

解决方法：

去 [phracker/MacOSX-SDKs](https://github.com/phracker/MacOSX-SDKs/releases) 下载 10.13 版本。解压后放置到 Mac sdk 文件夹 `/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs` 下。

### 问题三：执行上述操作后，重新编译又回到问题一

其原因应该是 aosp 编译系统查找 mac sdk 的规则问题，使用以下方法可以欺骗一下构建系统。

解决方法：

移除现有的 MacOSX10.15.sdk **软链接**，执行以下命令来给之前复制进来的 10.13 sdk 创建一个名为 MacOSX10.15.sdk 的软连接，来冒充 10.15 sdk。最后这个问题终于解决。

> 这里不能够通过在 Finder 中创建替身的方式来创建软链接。

```shell
cd /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs

sudo ln -s MacOSX10.13.sdk MacOSX10.15.sdk
```

参考资料：[android - Running AOSP build on Mac (Yosemite and later) - Stack Overflow](https://stackoverflow.com/questions/31589866/running-aosp-build-on-mac-yosemite-and-later?answertab=active#tab-top)。

### 问题四：sepolicy_tests_intermediates/sepolicy_tests error

问题详情如下：

```shell
...
[ 89% 64112/71309] build out/target/pr...icy_tests_intermediates/sepolicy_tests
FAILED: out/target/product/generic/obj/ETC/sepolicy_tests_intermediates/sepolicy_tests
...
```

其原因是 libc++_static 库被重复引入了。删除 `system/sepolicy/tests/Android.pb` 文件中的 libc++_static 那一行即可。后来有提交修复了该问题，[点击查看该提交的 diff](https://android-review.googlesource.com/c/platform/system/sepolicy/+/1173528/1/tests/Android.bp)。其改动如下图：

![diff](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1605291337.39SmartPic.png)

参考资料：[点击跳转](https://groups.google.com/g/android-building/c/_VyLXSosgoo)

## 参考资料

- 《Android 进阶指北》
- [Releases · phracker/MacOSX-SDKs · GitHub](https://github.com/phracker/MacOSX-SDKs/releases)
- [AOSP 清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)
- 在解决问题过程中 Google、StackOverflow 帮助很大，感谢

> 注：一些资料已在文中列出在此不再累述