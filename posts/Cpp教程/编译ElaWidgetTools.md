---
title: 编译 ElaWidgetTools 库
date: 2025-01-03 18:42:26
tags:
    - "c++"
    - "cmake"
excerpt: "这篇教程讲述如何编译`github`上开源的`ElaWidgetTools`库，编译环境是 Arch Linux。"
categories: "C++"
---


# 1.库下载

```bash
git clone https://github.com/Liniyous/ElaWidgetTools.git
```

# 2.编译库

## 2.1 修改点

主要修改的点是`./CMakeLists.txt`、`./src/CMakeLists.txt`和`./example/CMakeLists.txt`。

- 修改`./src/CMakeLists.txt`中的安装路径：
原来的：
```cmake
SET(CMAKE_INSTALL_PREFIX ${CMAKE_SOURCE_DIR}/install CACHE PATH "Installation path" FORCE)
```
改为：
```cmake
SET(CMAKE_INSTALL_PREFIX /opt/ElaWidgetTools CACHE PATH "Installation path" FORCE)
```
当然，这部分其实可以不改，我是习惯把第三方库安装`/opt`目录下，这样可以用`find_package`直接找到。

原来的：
```cmake
configure_package_config_file(
    ${PROJECT_SOURCE_DIR}/${EXPORT_NAME}Config.cmake.in
    ${PROJECT_BINARY_DIR}/${EXPORT_NAME}Config.cmake
    INSTALL_DESTINATION lib/cmake
    PATH_VARS INCLUDE_DIRS LIBRARIES LIB_DIR
    INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX}
)

install(
    FILES ${PROJECT_BINARY_DIR}/${EXPORT_NAME}Config.cmake ${PROJECT_BINARY_DIR}/${EXPORT_NAME}ConfigVersion.cmake DESTINATION lib/cmake
)
```
改为：
```cmake
configure_package_config_file(
    ${PROJECT_SOURCE_DIR}/${EXPORT_NAME}Config.cmake.in
    ${PROJECT_BINARY_DIR}/${EXPORT_NAME}Config.cmake
    INSTALL_DESTINATION lib/cmake/${EXPORT_NAME} # 修改这里
    PATH_VARS INCLUDE_DIRS LIBRARIES LIB_DIR
    INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX}
)

install(
    FILES ${PROJECT_BINARY_DIR}/${EXPORT_NAME}Config.cmake ${PROJECT_BINARY_DIR}/${EXPORT_NAME}ConfigVersion.cmake DESTINATION lib/cmake/${EXPORT_NAME} # 修改这里
)
```

然后就可以开始编译并安装了。

## 2.2 编译测试案例

- 修改`./CMakeLists.txt`：
添加：
```cmake
set(BUILD_ELAWIDGETTOOLS_EXAMPLE TRUE)
```

- 修改`./example/CMakeLists.txt`：
注释掉下面这行：
```bash
# list(APPEND CMAKE_PREFIX_PATH ${CMAKE_SOURCE_DIR}/install/lib/cmake/)
```

# 3.使用示例

`CMakeLists.txt`中查找`ElaWidgetTools`库编写：
```cmake
find_package(ElaWidgetTools REQUIRED)

message(STATUS "ElaWidgetTools include dirs: ${ELAWIDGETTOOLS_INCLUDE_DIRS}")
message(STATUS "ElaWidgetTools libraries: ${ELAWIDGETTOOLS_LIBRARIES}")
message(STATUS "ElaWidgetTools library dirs: ${ELAWIDGETTOOLS_LIBRARY_DIRS}")

target_include_directories(test.out PUBLIC
    ${ELAWIDGETTOOLS_INCLUDE_DIRS}
    # ExamplePage
    # ModelView
)
target_link_directories(test.out PUBLIC
    ${ELAWIDGETTOOLS_LIBRARY_DIRS}
)
target_link_libraries(test.out PRIVATE
    ${ELAWIDGETTOOLS_LIBRARIES}
)
```
**注意：** `target_link_directories`是必需的，当`ElaWidgetTools`库文件所在的目录不在系统默认路径或未被`find_package`自动添加时。这一步骤告诉链接器正确的库文件路径，是解决链接失败的关键。主要原因是`ElaWidgetTools` 没有正确安装到`CMAKE_INSTALL_PREFIX`或其路径未被标准化造成的。一般比较正规的标准库（比如`Qt`、`PCL`、`OpenCV`）是不需要额外写`target_link_directories`的，因为链接器路径会自动设置。

不添加的话就会报这种错误：
```bash
/usr/sbin/ld: cannot find -lElaWidgetTools: No such file or directory
clang++: error: linker command failed with exit code 1 (use -v to see invocation)
```
遇到相似的错误可以试着采用这种方式来解决下。

当然，也可以写成：
```cmake
find_package(ElaWidgetTools REQUIRED)

target_include_directories(test.out PUBLIC
    ${ELAWIDGETTOOLS_INCLUDE_DIRS}
    # ExamplePage
    # ModelView
)
target_link_libraries(test.out PRIVATE
    ${ELAWIDGETTOOLS_LIBRARY_DIRS}/libElaWidgetTools.so
)
```

测试代码：
```cpp
#include <ElaApplication.h>
#include <ElaWindow.h>
#include <QApplication>

int main(int argc, char* argv[])
{
    QApplication app(argc, argv);

    ElaApplication::getInstance()->init();
    ElaWindow w;
    w.show();

    return app.exec();
}
```