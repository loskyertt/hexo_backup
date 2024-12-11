---
title: VSCode 配置 C++ Qt 开发环境
date: 2024-12-11 19:04:07
tags:
    - "linux"
    - "c++"
    - "WSL"
excerpt: "注意：这里的系统环境是 Arch-WSL2 子系统。再其它 WSL2 下或者 Linux 发行版下该教程都有用。"
---


# 1.安装开发包

```bash
sudo pacman -S qt6-base
```
如果要安装其它版本的，比如`qt5`，则是`qt5-base`。

对于`WSL`下，是只能连接到`x11`服务的，比较 Windows 也没有`wayland`会话。所以还需要安装`xcb`支持库：
```bash
sudo pacman -S qt6-xcb-private-headers
```
需要与安装的`qt`版本对应，因为我安装的是`qt6-base`，我这里自然安装的就是`qt6`对应的`xcb`支持库。

安装好后需要配置环境变量，把下面这行内容添加到`.bashrc`或者`.zshrc`中（取决于自己用的`shell`）：
```bash
export QT_QPA_PLATFORM=xcb
```

进行这步配置主要是为了避免运行`qt`程序时，出现下面的问题：
```txt
qt.qpa.plugin: Could not find the Qt platform plugin "wayland" in ""
```

# 2.项目构建

一个简单的`qt`开发模板：
```bash
cmake_minimum_required(VERSI0N 3.12)

set(CMAKE_C_COMPILER clang)
set(CMAKE_CXX_COMPILER clang++)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

project(qtguide)

find_package(Qt6 REQUIRED COMPONENTS Core Widgets REQUIRED)

file(GLOB sources *.cpp)
add_executable(qtguide ${sources})
target_link_libraries(qtguide PRIVATE Qt6::Core Qt6::Widgets)
```
建议设置`clang`和`clang++`作为编译器。我在 Arch-WSL2 下使用`gcc`和`g++`会出现问题。还有就是设置编译器这部分，一定要放在`cmake_minimum_required`下面，其它的上面。因为编译器的选择会影响后续变量、库、目标等配置。CMake 初始化后再更改编译器可能会导致配置问题。

对于`qt`模板，可以参考我的仓库中的模板配置：[cpp_template](https://github.com/loskyertt/cpp_template)。