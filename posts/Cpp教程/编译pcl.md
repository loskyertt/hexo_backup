---
title: 如何手动编译 PCL 库
date: 2024-12-04 10:03:27
tags:
    - "c++"
    - "cmake"
excerpt: "这篇教程以 Docker 中的 Ubuntu 镜像作为编译环境。"
categories: "C++"
---

# 1.环境说明

编译环境是 Docker 的 Ubuntu 镜像所运行的容器实例，版本是`"DockerVersion": "24.0.7"`。运行好容器后，第一部还是要配置好镜像源（或网络代理），然后更新软件仓库。

# 2.安装编译工具和依赖

创建安装脚本：
```bash
touch install_build_pcl.sh
```

把下面内容添加到脚本中：
```bash
#!/bin/bash

# 更新包列表并升级系统
echo "更新系统并安装依赖项..."
apt update
apt upgrade -y

# 安装基础工具和开发环境
apt install -y build-essential cmake git pkg-config clang clangd wget unzip

# 安装系统库和硬件相关支持
apt install -y \
  libusb-1.0-0-dev \
  libusb-dev \
  libudev-dev \
  freeglut3-dev \
  libxmu-dev \
  libxi-dev \
  libomp-dev

# 安装数学和算法库
apt install -y \
  libboost-all-dev \
  libeigen3-dev \
  libflann-dev \
  libqhull-dev

# 安装 VTK（可视化工具包）
apt install -y libvtk9-dev

# 安装图形和 GUI 相关支持
apt install -y \
  libglew-dev \
  libglfw3-dev \
  libpng-dev \
  libjpeg-dev

# 安装 Qt5 和 OpenGL 支持
apt install -y \
  qtbase5-dev \
  qttools5-dev-tools \
  libqt5opengl5-dev

# 安装点云处理相关库
apt install -y \
  libopenni-dev \
  libopenni2-dev \
  libpcap-dev

# 安装额外的开发工具（可选）
# apt install -y \
#   mono-complete \
#   openjdk-8-jdk \
#   openjdk-8-jre \
  
# 安装中文包
apt install -y language-pack-zh-hans

# 清理无用的包缓存
apt autoremove -y
apt clean

echo "所有依赖项已安装！准备编译 PCL。"
```

将脚本更改为可执行权限：
```bash
chmod +x install_build_pcl.sh
```

最好配置下`.bashrc`，增加代理和中文字体配置。在`.bashrc`中添加如下内容：
```bash
# 中文字体设置
export LANG=zh_CN.UTF-8
export LANGUAGE=zh_CN:zh
export LC_ALL=zh_CN.UTF-8

# where proxy
proxy(){
  export http_proxy="http://127.0.0.1:7890"
  export https_proxy="http://127.0.0.1:7890"
  echo "HTTP Proxy on"
}

# where noproxy
noproxy(){
  unset http_proxy
  unset https_proxy
  echo "HTTP Proxy off"
}
```

# 3.编译

## 3.1 拉取源码

可以通过`git`拉取最新源码：
```bash
git clone https://github.com/PointCloudLibrary/pcl.git
```

也可以直接下载指定版本源码并进行解压，比如`1.14`版的：
```bash
# 下载
wget https://github.com/PointCloudLibrary/pcl/releases/download/pcl-1.14.1/source.zip

# 解压到当前目录
unzip source.zip -d .
```

## 3.2 开始编译

1. **方式一：安装到默认目录**

```bash
cd pcl

mkdir build

cd build

cmake ..

make -j8

make install
```

2. **方式二：安装到指定目录**

比如要安装到`/root/pcl_1.14/`目录下：
```bash
mkdir /root/pcl_1.14
```

开始编译：
```bash
cd pcl

mkdir build

cd build

cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/root/pcl_1.14

make -j8

make install
```

这种方式会把`pcl`安装到`/root/pcl_1.14`下，`pcl_1.14`下的文件结构是这样的：
```txt
.
├── bin
├── include
├── lib
└── share

5 directories, 0 files
```

如果是安装到指定的目录，在配置`CMakeLists.txt`时，略有不同：
```cmake
# 设置 PCL 的安装目录（这个路径包含 PCL 的 CMake 配置文件）
set(PCL_DIR "/root/pcl_1.14/share/pcl-1.14")

# 查找 PCL 包
find_package(PCL REQUIRED)

# 设置编译选项
include_directories(${PCL_INCLUDE_DIRS})
link_directories(${PCL_LIBRARY_DIRS})

# 添加可执行文件
add_executable(your_project main.cpp)

# 链接 PCL 库
target_link_libraries(your_project ${PCL_LIBRARIES})
```
