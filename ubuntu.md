[TOC]

### 零，开发系统

我们主要使用的开发系统是 ubuntu，其他debian系的系统理论上也可以，使用apt作为包管理软件，所以我们要先装一个ubuntu系统，我们当前使用的版本是ubuntu22.04。

![alt text](assets/image.png)

为什么需要使用ubuntu系统？



有很多原因，一，它有着开箱即用的包管理支持，可以使用指令来下载你需要的所有依赖库

二，linux系统所耗性能一般比较小，内存不会吃那么紧，而ubuntu是众多linux发行版中比较完备的一款，适合新人上手。

三，ubuntu系统开放性更高，方便处理串口，USB，网卡，雷达，工业相机等外设。

四，ROS与机器人开源生态更成熟，兼容有机器人驱动，算法框架和ROS工具链。





什么是终端？

---

终端是与linux交互最常用的途径，ubuntu的默认终端叫bash，全称是Bourne Again SHell，快捷键是 Ctrl + Alt + T 。

linux的一些常见符号

| 符号 | 含义                 | 示例          |
| ---- | -------------------- | ------------- |
| `/`  | 根目录               | `/home`       |
| `~`  | 当前用户的家目录     | `~/Documents` |
| `.`  | 当前目录             | `./main`      |
| `..` | 上一级目录           | `cd ..`       |
| `./` | 从当前目录开始的路径 | `./build/app` |

linux的一些基础指令

pwd           # 查看当前目录
ls            # 查看当前目录中的内容
cd ..         # 回到上一级目录
cd Projects   # 进入 Projects 目录（假设位于其父目录，/home/xiaoming/Projects，Projects的父目录是xiaoming）
cd ~          # 返回家目录


### 一，安装一些小工具

开发CPP项目，我们首先选择GCC和CMAKE,用于编译代码和管理项目

**GCC**：GNU Compiler Collection（GNU 编译器集合）的简称。它是一套编译工具，其中最常用的是 `g++`，负责把 C/C++ 源代码编译、链接成可执行程序或库。
例如：`g++ main.cpp -o main`

**CMake**：项目构建管理工具。它通过 `CMakeLists.txt` 描述项目中的源文件、依赖库和编译选项，然后生成 Makefile、Ninja 文件或其他构建文件，交给 GCC 等编译器完成实际编译。

按 Ctrl+Alt+T 打开终端 把下面这些指令复制进去



``` bash
sudo apt-get install -y \
    build-essential lsb-release \
    gnupg software-properties-common \
    wget curl unzip git \
    gcc g++ gdb make cmake
```



下载clangd用于支持cpp语法补全：

**Clang**：把 C/C++/Objective-C 代码翻译成机器能执行内容的“编译器”。它和 GCC 类似，负责检查代码、编译代码，也能生成可执行文件。

**clangd**：写代码时的“智能助手”。它在编辑器里提供代码补全、跳转到定义、查找引用、实时错误提示和代码分析等功能。它不是主要用来生成程序的编译器，而是帮助你更高效地写代码。


在终端输入

``` bash
touch install.sh
```

创建一个install.sh文件，在主目录找到install.sh并且打开，将下面脚本复制进这个文件

``` bash
#!/usr/bin/env bash
sudo apt-get update -y
sudo apt-get install -y gnupg wget

wget -O /tmp/llvm-snapshot.gpg.key https://apt.llvm.org/llvm-snapshot.gpg.key
sudo apt-key add /tmp/llvm-snapshot.gpg.key
rm /tmp/llvm-snapshot.gpg.key

sudo touch /etc/apt/sources.list.d/llvm-apt.list
sudo echo "deb https://mirrors.tuna.tsinghua.edu.cn/llvm-apt/jammy/ llvm-toolchain-jammy main" |\
sudo tee -a  /etc/apt/sources.list.d/llvm-apt.list
sudo apt-get update -y

version=`apt-cache search clangd- | grep clangd- | awk -F' ' '{print $1}' | sort -V | tail -1 | cut -d- -f2`
sudo apt-get install -y clangd-$version
```

记得保存一下。

然后在终端运行 

``` bash
chmod +x ./install.sh && ./install.sh
```



---



安装完之后，你的系统会出现clangd-22或者其他版本，可以使用软链接将可执行文件链接成标准命名，方便其他软件使用。

1,查看系统用的什么版本clangd  

方法一：在终端输入 clangd- 然后按 Tab键补全，就可以看到现在用的是什么版本

方法二：在终端输入 

```bash
ls /usr/bin/clangd*
```



就可以直接查看



2,用软链接将可执行文件链接成标准命名



！！注意这个clangd-22要替换成你系统拥有的版本



``` bash
sudo ln -s /usr/bin/clangd-22 /usr/bin/clangd
```

3,下载clang

在终端输入

 ``` bash
 sudo apt install clang
 ```



---



### 二，安装编辑器VSCode


1,打开浏览器


注：一开始默认的是火狐浏览器，建议使用microsoft edge,可以先去下载microsoft edge,然后在设置里面改默认浏览器就行了。


在ubuntu 平台下，安装包的格式一般是.deb格式，所以下载的是x86平台下的deb包（Debian），如果是windows平台，下载的是exe格式的安装包。

2,搜索vscode，然后下载deb包


进入下载目录之后输入

``` bash
sudo apt install ./code*****.deb

```



下载完毕之后，打开vscode的拓展（Ctrl + Shift + X ），搜索Chinese ，下载中文简体，然后右下角或者左下角会弹出一个提示，点击reload重启vscode，就可以切换成中文了。



然后我们再下载一些插件，打开拓展，下载 clangd（LLVM） 和cmake （twxs） 就可以得到完整的cpp开发体验。



注意！！！有时候会弹出微软官方推荐下载C/C++插件，这个不要下载！建议直接禁用，否则会和clangd冲突（clangd更好使）。

---



### 三，简介构建流程



我们编写的代码源文件，都需要通过如下的完整构建流程进行build



源文件(.c/.cpp) → [预处理] → 预处理文件(.i) → [编译] → 汇编文件(.s) → [汇编] → 目标文件(.o) → [链接] → 可执行文件



这些操作都可以通过gcc/g++进行，也可直接编译源文件，生成可执行文件。



gcc命令虽然对于简单项目或单一文件编译操作简单，但遇到大型项目时，问题便来了： 我们需要将多个文件（可能上百个乃至更多）编译成一个或多个可执行文件，我们当然可以简单地通过 g++ src1.cpp src2.cpp ... 来编译，但是特别繁琐并且还会有其他诸多问题。



所以我们的现代编译工具链就是：



cmake + clang +clangd 



cmake很好的解决了以上问题，实现了多文件编译、管理外部依赖、代码模块化的功能. 



cmake作为构建工具，可以帮我们自动生成makefile,省去了我们去编写复杂的makefile的过程，通过简单的CMakeLists编写，我们就可以很快的进行构建；clang作为更高效现代的c/c++编译器，相比gcc能够更快速地进行编译；clangd作为IDE中的插件，能够帮助我们解决头文件路径问题、进行代码诊断、作为代码格式化工具等。



在这里着重说一下，在vscode中刚clone下来的新项目，经常会遇到一堆报错的情况，仔细看都是can not find dictionary or file ,但如果你试着编译一下会发现编译通过。这是vscode找不到头文件路径而clang可以找到导致的IDE虚假报错，但我们可以通过clangd插件来解决这个问题的，只需要在CMakeLists中加入  set(CMAKE_EXPORT_COMPILE_COMMANDS ON) 。



在 CMakeLists.txt 中加入 set(CMAKE_EXPORT_COMPILE_COMMANDS ON) 后，CMake 会生成 compile_commands.json。clangd 会读取该文件，从而获得正确的头文件路径、编译选项和宏定义，减少 VSCode 中的虚假报错。然后保存刷新一下就可以了

---



### 四：成功编译一个c++项目



首先创建如下目录结构

``` 
build/
src/
    main.cpp
CMakeLists.txt
```



然后在main.cpp里面写下 "梦的开始" 

``` c++
#include <iostream>

int main() {
  std::cout << "Hello World" << '\n';
  return 0;
}

```



然后打开CMakeLists.txt,输入

``` 
cmake_minimum_required(VERSION 3.20)

project(hello)

add_executable(
    ${PROJECT_NAME}
    src/main.cpp
)
```

记得按 Ctrl + S保存！！！



打开vscode 的终端（ Ctrl + ` ）终端默认打开的是当前目录的根目录



输入 

``` bash
cmake -B build

cmake --build build -j
```

