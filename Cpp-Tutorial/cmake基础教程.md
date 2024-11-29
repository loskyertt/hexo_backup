---
title: CMake 基础教程
date: 2024-11-29 14:39:43
tags:
    - "c++"
    - "cmake"
excerpt: "本文讲述 CMake 的一些基础指令。"
categories: "C++"
---

# 1.构建指令

现代`CMake`提供了更方便的`-B`和`--build`指令，不同平台，统一命令！

在源码目录用`-B`直接创建`build`目录并生成`build/Makefile`:
```bash
cmake -B build
```
这种方式免去了先创建`build`目录再切换进去再指定源码目录的麻烦。

示例如下：
![示例1](https://s2.loli.net/2024/11/29/tL9qsWvMekI3zcw.png)

自动调用本地的构建系统在`build`里构建，相当于`make -C build -j4`：
```bash
cmake --build build -j4
```
这种方式统一了不同平台（Linux 上会调用`make`，Windows 上调用`devenv.exe`）

调用本地的构建系统执行`install`这个目标，即安装：
```bash
sudo cmake --build build --target install
```
相当于 Linux 下的`sudo make install`。

结论：从现在开始，如果在命令行操作`cmake`，请使用更方便的`-B`和`--build`命令。

# 2.生成器

- Linux 系统上的`CMake`默认用是`Unix Makefiles`生成器；Windows 系统默认是`Visual Studio 2019`生成器；MacOS 系统默认是`Xcode`生成器。
- 可以用`-G`参数改用别的生成器，例如`cmake -G Ninja`会生成`Ninja`这个构建系统的构建规则。`Ninja`是一个高性能，跨平台的构建系统，Linux、Windows、MacOS上都可以用。
- `Ninja`可以从包管理器里安装，没有包管理器的 Windows 可以用 Python 的包管理器安装：`pip install ninja`（有趣的事实：`CMake`也可以通过`pip install cmake`安装）。
- 事实上，`MSBuild`是单核心的构建系统，`Makefile`虽然多核心，但因历史兼容原因效率一般。而`Ninja`则是专为性能优化的构建系统，它和`CMake`结合都是行业标准了。

```bash
cmake -G Ninja -B build
```
注：`-G Ninja`还可以写成`-GNinja`和`-G "Ninja"`（推荐这种）。在 Linux 上没有指定的话，默认是`-G "Unix Makefiles"`。

如图所示：
![示例2](https://s2.loli.net/2024/11/29/5veQ3nHVUhT7ipl.png)

## 2.1 `Ninja`和`Makefile`的简单对比：

`ninja`花费的时间：
![ninja.png](https://s2.loli.net/2024/11/29/sha1dGMkHTNuQ8A.png)

`make`花费的时间：
![make.png](https://s2.loli.net/2024/11/29/kfcOwtZbQV4UMA5.png)

从两幅图对比可以看出，`ninja`花费的时间更少，效率更高。

事实上，在性能上：`Ninja > Makefile > MSBuild`。`Makefile`启动时会把每个文件都检测一遍，浪费很多时间。特别是有很多文件，但是实际需要构建的只有一小部分，从而是`I/O Bound`的时候，`Ninja`的速度提升就很明显。然而某些专利公司的`CUDA toolkit`在 Windows 上只允许用`MSBuild`构建，不能用`Ninja`（怕不是和`Bill Gates`有什么交易，哈哈）。

# 3.编译器设置

可以在`CMakeLists.txt`中添加：
```CMake
set(CMAKE_C_COMPILER clang)
set(CMAKE_CXX_COMPILER clang++)
```
指定编译器为`clang`和`clang++`。不指定的话，默认是`gcc`和`g++`。

也可以在命令行阶段指定：
```bash
cmake -B build -G Ninja -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++
```