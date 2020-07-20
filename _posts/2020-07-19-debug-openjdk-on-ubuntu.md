---
title: 在Ubuntu中编译和调试OpenJDK
tags:
    - OpenJDK
    - Ubuntu
    - CLion
---

## 配置环境

### 构建编译环境

安装GCC编译器：

```bash
sudo apt install build-essential
```

安装OpenJDK依赖库：

| 工具       | 库名称                              | 安装命令                                                     |
| ---------- | ----------------------------------- | ------------------------------------------------------------ |
| FreeType   | The FreeType Project                | `sudo apt install libfreetype6-dev`                          |
| CUPS       | Common UNIX Printing System         | `sudo apt install libcups2-dev`                              |
| X11        | X Window System                     | `sudo apt install libx11-dev libxext-dev libxrender-dev libxrandr-dev libxtst-dev libxt-dev` |
| ALSA       | Advanced Linux Sound Architecture   | `sudo apt install libasound2-dev`                            |
| libffi     | Portable Foreign Function Interface | `sudo apt install libffi-dev`                                |
| Autoconf   | Extensible Package of M4 Macros     | `sudo apt install autoconf`                                  |
| zip/unzip  | unzip                               | `sudo apt install zip unzip`                                 |
| fontconfig | fontconfig                          | `sudo apt install libfontconfig1-dev`                        |

假设要编译大版本号为N的JDK，我们还要安装一个大版本号**至少为N-1**的、已经编译好的JDK作为“Bootstrap JDK”：

```bash
sudo apt install openjdk-11-jdk
```

### 获取源码

可以直接访问准备下载的JDK版本的仓库页面（譬如本例中OpenJDK 11的页面为[https://hg.openjdk.java.net/jdk-updates/jdk11u/](https://hg.openjdk.java.net/jdk-updates/jdk11u/)），然后点击左边菜单中的“Browse”，再点击左边的“zip”链接即可下载当前版本打包好的源码，到本地直接解压即可。

也可以从Github第三方Repositories中获取（[https://github.com/unofficial-openjdk/openjdk](https://github.com/unofficial-openjdk/openjdk)、[https://github.com/AdoptOpenJDK/](https://github.com/AdoptOpenJDK/)），点击Clone按钮下的**Download ZIP**按钮下载打包好的源码，到本地直接解压即可。

### 进行编译

首先进入解压后的源代码目录，本例解压到的目录为`~/openjdk/`：

```bash
cd ~/openjdk
```

要想带着调试、定制化的目的去编译，就要使用OpenJDK提供的编译参数，可以使用`bash configure --help`查看. 本例要编译FastDebug版、仅含Server模式的HotSpot虚拟机，对应命令如下：

```bash
bash configure --enable-debug --with-jvm-variants=server
```

> 对于版本较低的OpenJDK，编译过程中可能会出现了源码**deprecated**的错误，这是因为>=2.24版本的glibc中 ，readdir_r等方法被标记为deprecated。若读者也出现了该问题，请在`configure`命令加上`--disable-warnings-as-errors`参数，如下：
>
> ```bash
> bash configure --enable-debug --with-jvm-variants=server --disable-warnings-as-errors
> ```
>
> 此外，若要重新编译，请先执行`make clean`

执行`make`命令进行编译：

```bash
make
```

生成的JDK在`build/配置名称/jdk`中，测试一下，如：

```bash
cd build/linux-x86_64-normal-server-fastdebug/jdk/bin
./java -version
```

### 生成Compilation Database

`CLion`可以通过`Compilation Database`来导入项目. 在`OpenJDK 11u`及之后版本中，`OpenJDK`官方提供了对于`IDE`的支持，可以使用`make compile-commands`命令生成`Compilation Database`：

```bash
make compile-commands
```

> 对于版本较低的OpenJDK，可以使用一些工具来生成`Compilation Database`，比如：
>
> - [Bear](https://github.com/rizsotto/Bear)
> - [scan-build](https://github.com/rizsotto/scan-build)
> - [compiled](https://github.com/nickdiego/compiledb)

然后检查一下`build/配置名称/`下是否生成了`compile_commands.json`.

```bash
cd build/linux-x86_64-normal-server-fastdebug
ls -l
```

### 导入项目至CLion

打开`CLion`，选择`Open Or Import`，选择上文生成的`build/配置名称/compile_commands.json`文件，弹出框选择`Open as Project`，等待文件索引完成.

接着，修改项目的根目录，通过`Tools -> Compilation Database -> Change Project Root`功能，选中你的源码目录.

### 配置调试选项

#### 创建自定义`Build Target`

点击`File`菜单栏，`Settings -> Build -> Execution -> Deployment -> Custom Build Targets`，点击`+`新建一个`Target`，配置如下：

- `Name`：`Target`的名字，之后在创建`Run/Debug`配置的时候会看到这个名字

- 点击`Build`或者`Clean`右边的三点，弹出框中点击`+`新建两个`External Tool`配置如下：

  ```yaml
  # 第一个配置如下，用来指定构建指令
  # Program 和 Arguments 共同构成了所要执行的命令 "make all"
  Name: make
  Program: make
  Arguments: all
  Working directory: {项目的根目录}
  
  # 第二个配置如下，用来清理构建输出
  # Program 和 Arguments 共同构成了所要执行的命令 "make clean"
  Name: make clean
  Program: make
  Arguments: clean
  Working directory: {项目的根目录}
  ```

- `ToolChain`选择`Default`；`Build`选择`make`（上面创建的第一个`External Tool`）；`Clean`选择`make clean`（上面创建的第二个`External Tool`）

#### 创建自定义的`Run/Debug configuration`

点击`Run`菜单栏，`Edit Configurations`， 点击`+`，选择`Custom Build Application`，配置如下：

```yaml
# Executable 和 Program arguments 可以根据需要调试的信息自行选择
# NameL：Configure 的名称
Name: linux-x86_64-normal-server-fastdebug
# Target：选择上一步创建的 “Custom Build Target”
Target: linux-x86_64-normal-server-fastdebug
# Executable：程序执行入口，也就是需要调试的程序
Executable: 这里我们调试`java`，选择`{source_root}/build/{build_name}/jdk/bin/java`。
# Program arguments: 与 “Executable” 配合使用，指定其参数
Program arguments: 这里我们选择`-version`，简单打印一下`java`版本。
# Before luanch：这个下面的Build可去可不去，去掉就不会每次执行都去Build，节省时间，但其实OpenJDK增量编译的方式，每次Build都很快，所以就看个人选择了。
```

### 开始调试

`Ubuntu`下默认使用的调试器是`gdb`，调试的时候可能会发现`gdb`报错：`Signal: SIGSEGV (Segmentation fault)`。解决办法是，在`compile_commands.json`同一目录或用户目录下创建`.gdbinit`，内容如下：

```bash
handle SIGSEGV pass noprint nostop
handle SIGBUS pass noprint nostop
```

在`${source_root}/src/java.base/share/native/libjli/java.c`的`401`行打断点，点击`Debug`即可开始调试.

### 参考文章

- [Tips & Tricks: Develop OpenJDK in CLion with Pleasure](https://blog.jetbrains.com/clion/2020/03/openjdk-with-clion/)
- [OpenJDK 编译调试指南(Ubuntu 16.04 + MacOS 10.15)](https://juejin.im/post/5ef8c6a86fb9a07e7654ef9d)
- [JVM-在MacOS系统上使用CLion编译并调试OpenJDK12](https://www.howieli.cn/posts/macos-clion-build-debug-openjdk12.html)