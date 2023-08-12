# Java 环境搭建

## 编辑器

你需要一个代码编辑器来编写 Java 代码，IDEA、VSCode、Vim 是常见的编辑器。VSCode 可以满足基本的需求，它是完全免费的。IDEA 在 Java 项目上是更加强大的编辑器，选择 IDEA 的 Community 社区免费版本即可。Vim 则是一个免费的命令行编辑器，它的学习成本较高，但是在熟练后可以达到甚至超越其他编辑器。

上述软件可以在以下网址下载安装：
- [IDEA](https://www.jetbrains.com/idea/download/)
- [VSCode](https://code.visualstudio.com/download)
- [Vim](https://www.vim.org/download.php)

## JDK

JDK：Java Development Kit，Java 开发工具包，包含了 Java 运行环境（JRE）和一些开发工具。

安装 JDK 在不同的操作系统中的命令如下：
- Windows
  - 安装 Chocolatey
  - 执行 `choco install jdk8`
- macOS
  - 安装 Homebrew
  - 执行 `brew cask install java`
- Linux
  - 执行 `sudo apt install default-jdk`

打开一个 shell 窗口，执行以下命令以验证安装成功：

```shell
java -version
```

## 编译和运行

### Shell 命令

Java 代码需要先编译成字节码，再由 JVM 运行。编译器和 JVM 都包含在 JDK 中。

编译 Java 代码：

```shell
javac HelloWorld.java
```

运行 Java 代码：

```shell
java HelloWorld
```

对于 IDE，可以直接配置对应环境变量后，点击 IDE 的运行按钮即可。