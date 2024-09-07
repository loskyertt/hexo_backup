---
title: Docker 教程
date: 2024-07-24 10:55:53
tags:
  - "docker"
excerpt: "记录下 Docker 的使用过程，比如配置镜像的记录和指令。"
---


#  一、基本指令

## 1.1 查看镜像版本
```bash
docker inspect <镜像ID> | grep -i version
```

## 1.2 查看镜像或容器详细信息
```bash
# 镜像
docker inspect <镜像名或ID>

# 容器
docker inspect <容器名或ID>
```
可以同过这种方式查看镜像里的主要`workdir`在哪里，这样可以知道应该挂载镜像里的哪个目录。

## 1.3 查看容器映射到本地的端口

如过在创建容器时用的是`-P`（随机映射本地端口）而不是`-p xxxx:xxxx`（指定映射本地端口）
```bash
docker port <容器名或ID>
```

## 1.4 在本地对容器内的文件进行操作

可以使用 `docker cp` 命令将容器内的自己需要文件（文件或者文件夹都可以）导出到宿主机（反之亦然）。

从容器复制文件到宿主机:
```bash
docker cp my_container:/path/in/container /path/on/host
```

从宿主机复制文件到容器：
```bash
docker cp /path/on/host my_container:/path/in/container
```

## 1.5 删除镜像或容器

要一次性删除所有`Docker`镜像，可以通过以下步骤：

### 1.5.1 停止并删除所有容器

首先，确保所有容器已经停止并删除。你不能删除正在使用的镜像。

1. 停止所有容器：
```bash
docker stop $(docker ps -aq)
```

2. 删除所有容器：
```bash
docker rm $(docker ps -aq)
```

### 1.5.2 删除所有镜像

在确保所有容器都已停止并删除后，可以删除所有镜像：
```bash
docker rmi $(docker images -q)
```

### 1.5.3 详细说明

- **`docker ps -aq`：**
  - `docker ps`：列出所有运行中的容器。
  - `-a`：列出所有容器，包括运行中的和已停止的。
  - `-q`：只显示容器的 ID。

- **`docker images -q`：**
  - `docker images`：列出所有镜像。
  - `-q`：只显示镜像的 ID。

### 1.5.4 删除所有未使用的镜像、容器、网络和卷

```bash
docker system prune -a
```

- **`docker system prune -a`：**
  - `docker system prune`：清理所有未使用的容器、网络、挂载的卷和镜像。
  - `-a`：删除所有未使用的镜像，不仅仅是悬空的镜像（dangling images）。

## 1.6 网络配置

```bash
docker create --name =<name>  --network host image:tag 
```
使用参数`--network host`可以让容器共享主机的网络栈，通常用于需要高性能网络连接或访问主机网络的应用，也就是说容器内的端口与主机的都对应。默认是桥接模式。

## 1.7 实时查看容器日志

```bash
docker logs -f  <--timestamps> <容器ID或容器名称>
```
参数`--timestamps`可以选加，可以显示时间戳。

## 1.8 指定端口的映射协议

在 Docker 中，`-p` 选项用于指定端口映射时，主要支持以下两种协议：

- **TCP** ：这是 Docker 默认的协议。大多数服务和应用程序使用 TCP 协议，因此 `-p` 选项通常用于映射 TCP 端口。例如：

```bash
docker run -p 8080:80</tcp> my_image
```

这个命令将容器的 `80` 端口映射到宿主机的 `8080` 端口，使用 TCP 协议。`/tcp`可以选加,因为 Docker 的默认端口映射协议就是 TCP。

- **UDP** ：如果你的应用程序使用 UDP 协议，你可以使用 `-p` 选项来映射 UDP 端口。例如：

```bash
docker run -p 8080:80/udp my_image
```

这个命令将容器的 `80` 端口映射到宿主机的 `8080` 端口，使用 UDP 协议。

# 二、容器操作

这里以`ubutnu`镜像为例。

## 2.1 运行并创建容器

```bash
docker run -itd --name ubuntu-test ubuntu
```

**注意：**  这里必须加上`-it`。像ubuntu这种镜像在创建容器时需要分配一个伪终端让它在后台持续运行，否则会在启动容器后立马退出运行。

## 2.2 进入容器

```bash
docker exec -it ubuntu-test bash
```

要退出正在交互模式下的容器终端，可以执行以下操作：
1. 按下 `Ctrl + D` 快捷键。
2. 或者在容器终端中键入 `exit` 命令并按回车键。


# 三、容器挂载（`volume`）与绑定挂载（`bind`）的区别

## 3.1 位置

卷挂载的目录是在`/var/lib/docker/volumes/`下，绑定挂载的目录根据用户自定义。

## 3.2 卷挂载 (`docker volume`)

当使用 `docker volume create temp` 创建一个名为`temp`卷并将它挂载到容器时（例如 `-v temp:/clashmeta`），Docker 会自动管理这个卷。如果容器中指定的挂载目标目录（即 `/clashmeta`）已经有文件，而挂载的卷是空的，Docker 不会覆盖原有的文件。相反，Docker 会将这些文件复制到卷中，确保容器可以正常工作。这就是为什么当你使用 Docker 卷时，`/clashmeta` 的内容不会被覆盖。

## 3.3 绑定挂载 (`bind mount`)

当使用 `mkdir -p ~/temp` 创建一个本地目录，然后将这个目录绑定到容器时（例如 `-v ~/temp:/clashmeta`），情况会有所不同。`bind mount` 是一种直接将宿主机的文件系统目录挂载到容器的方式：

- 如果宿主机的目录已经有内容，那么容器内的目标目录将显示宿主机目录中的内容。
- 如果宿主机目录为空，那么它会 **覆盖** 容器内的目标目录，使得目标目录 `/clashmeta` 的原始内容不可见，宿主机的目录内容（即使为空）优先级更高。

这解释了为什么当使用 `~/temp` 本地目录作为挂载点时，它会覆盖容器的 `/clashmeta` 目录的内容。

## 3.4 总结

- **卷挂载** ：Docker 卷默认是空的，挂载到容器时不会覆盖容器内已有的数据，Docker 会将容器内的文件复制到卷中。
- **绑定挂载** ：绑定挂载会直接使用宿主机的目录替换容器内的目标目录，导致目标目录的内容不可见。如果宿主机目录为空，容器中的内容也会被覆盖为空。

