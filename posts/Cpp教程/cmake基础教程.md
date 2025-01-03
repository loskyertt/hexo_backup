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

# 3.设置

## 3.1 编译器设置

可以在`CMakeLists.txt`中添加：
```CMake
set(CMAKE_C_COMPILER clang)
set(CMAKE_CXX_COMPILER clang++)
```
指定编译器为`clang`和`clang++`。不指定的话，默认是`gcc`和`g++`。这两条命令应该放在文件的开始位置（`cmake_minimum_required`之下，其他命令之上），否则可能无效。

也可以在命令行阶段指定：
```bash
cmake -B build -G Ninja -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++
```

# 4.查找第三方库（非默认路径）

有时安装的第三方库没有安装在系统环境变量路径，此时需要手动添加。

## 4.1 `set(CMAKE_PREFIX_PATH "...")`

- **作用：**  
  设置一个搜索路径列表，供 CMake 的 `find_package()` 和其他查找命令使用。  
  - CMake 会在指定的路径下，递归搜索各种配置文件（如 `*.cmake`）。
  - 它是一个通用的全局路径变量，不仅适用于 Qt，还适用于其他外部库。
  - ***完全控制搜索路径，不使用默认路径，也就是说会覆盖默认的`CMAKE_PREFIX_PATH`。***

- **适用场景：**
  当你需要 CMake 查找多个不同的外部库或模块，并希望统一管理搜索路径时，使用 `CMAKE_PREFIX_PATH` 是最佳选择。

- **工作机制：**
  - 查找时会从 `CMAKE_PREFIX_PATH` 指定的目录中依次搜索。

- **优点：**
  - 更通用，可以适配不同的路径组织结构。
  - 统一管理多个依赖库的搜索路径。

示例：
```cmake
set(CMAKE_PREFIX_PATH "/opt/Qt/6.8.0/gcc_64/lib/cmake/Qt6/")
find_package(Qt6 REQUIRED COMPONENTS Widgets Core)
```
这里会在 `/opt/Qt/6.8.0/gcc_64/lib/cmake/Qt6/` 下寻找 `Qt6Config.cmake` 配置文件。

可以同时添加多条路径：`set(CMAKE_PREFIX_PATH "/opt/custom_lib1;/opt/custom_lib2")`。

## 4.2 `list(APPEND ...)`（推荐）

**作用：** 该命令的目的是将一个新的路径添加到 CMake 的 CMAKE_PREFIX_PATH 变量，让 CMake 在该路径下查找依赖库（find_package 查找时会用到）。通过`list(APPEND ...)`，可以在`CMAKE_PREFIX_PATH`变量的末尾添加新的路径。

示例：
```cmake
list(APPEND CMAKE_PREFIX_PATH /opt/Qt/6.8.0/gcc_64/lib/cmake/Qt6/)
find_package(Qt6 REQUIRED COMPONENTS Widgets Core)
```

可以同时添加多个路径：
```cmake
list(APPEND CMAKE_PREFIX_PATH 
    ${CMAKE_SOURCE_DIR}/install/boost_1_82_0 
    ${CMAKE_SOURCE_DIR}/install/pcl_1_14 
    ${CMAKE_SOURCE_DIR}/install/qt6
)
```

## 4.3 `set(Qt_dir "...")`

- **作用：**
  这是一个项目自定义变量（`Qt_dir` 的名字是随意的），用于直接指向 Qt 的具体路径（通常是 `Qt6Config.cmake` 所在的目录）。
  - CMake 本身对 `Qt_dir` 变量没有特殊处理。
  - 它可以在代码中手动使用，比如直接传递给 `find_package()` 的 `HINTS` 参数。

- **适用场景：**
  如果你只需要为当前项目指定一个特定的 Qt 路径，可以用这种方式。不过，这种方式需要手动确保路径的正确性，并显式使用。

- **工作机制**:
  - `Qt_dir` 不会影响 CMake 的全局搜索行为，除非在查找时手动传递它。

- **优点**:
  - 更精确地指定路径，避免其他可能的干扰。
  - 灵活性更高，适合需要手动控制路径的场景。

示例：
```cmake
set(Qt_dir /opt/Qt/6.8.0/gcc_64/lib/cmake/Qt6)
find_package(Qt6 REQUIRED COMPONENTS Widgets HINTS ${Qt_dir})
```