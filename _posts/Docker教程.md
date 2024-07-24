---
title: Docker 教程
date: 2024-07-24 10:55:53
tags:
  -"docker"
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

## 1.3 查看容器映射到本地的端口

如过在创建容器时用的是`-P`（随机映射本地端口）而不是`-p xxxx:xxxx`（指定映射本地端口）
```bash
docker port <容器名或ID>
```

## 1.4 在本地对容器内的文件进行操作

可以使用 `docker cp` 命令将容器内的自己需要文件导出到宿主机（反之亦然）。

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


# 二、Docker-MySQL 配置

## 2.1 拉取镜像与创建容器

1. **拉取 MySQL 镜像：**
   通过运行 `docker pull mysql[:版本号]`，版本号可选，默认是latest。

2. **创建并运行 MySQL 容器（推荐）：**
   使用以下命令来运行 MySQL 容器。这里假设想在主机上绑定3306端口，并设置一个root用户的密码：

```bash
docker run \
--name some-mysql \
-e MYSQL_ROOT_PASSWORD=1234 \
-p 3306:3306 \
-v ~/docker_volume/mysql/data:/var/lib/mysql \
-v ~/docker_volume/mysql/init:/docker-entrypoint-initdb.d \
-v ~/docker_volume/mysql/conf:/etc/mysql/conf.d \
-d mysql:<版本号>
```

**以上命令的说明：**
- `--name some-mysql`：给容器指定一个名称，比如 `some-mysql`。
- `-e MYSQL_ROOT_PASSWORD=1234`：设置root用户的密码为 `1234`。
- `mysql:latest`：使用最新版本的 MySQL 镜像。
- `p`：设置端口，`3306:3306`中，前者是宿主机的端口，可以自定义，后者是映射到容器的端口。
- `-v`：挂载容器里的文件（`~/docker_volume/mysql/data`为本地目录）。**注意：** 要先创建好对应的文件路径！！同时本地文件夹会把容器内的文件夹完全覆盖！

## 2.2 连接到容器

1. **连接到 MySQL 容器：**

进入到容器内部然后进行连接：
```bash
docker exec -it some-mysql bash

mysql -uroot -p
```

直接连接到容器：
```bash
docker exec -it some-mysql mysql -uroot -p
```
这会启动一个终端会话，并要求输入上面设置的root用户的密码 (`1234`)。

以上命令的说明：
1. **docker exec：**
- **含义：** 在一个已经运行的容器内执行命令。
- **用途：** 允许你在运行的容器中执行新的命令。

2. **-it：**
- **`-i` (interactive)：** 保持标准输入（stdin）打开，使得你可以与容器中的进程进行交互。
- **`-t` (tty)：** 分配一个伪终端，提供一个终端会话环境。这两个选项一起使用可以进入容器并交互。

3. **some-mysql：**
- **含义：** 容器的名称或 ID。

4. **mysql：**
- **含义：** 这是在容器内执行的命令。在这个例子中，是启动 MySQL 客户端。
- **用途：** 连接到 MySQL 数据库。

5. **-uroot：**
- **`-u`：** 指定 MySQL 客户端的用户名。
- **root：** MySQL 数据库的用户名。在这个例子中，使用的是 `root` 用户。

6. **`-p`：**
- **含义：** 提示输入 MySQL 用户的密码。
- **用途：** 在执行命令后，你会被提示输入 `root` 用户的密码。这个密码是在运行容器时通过 `MYSQL_ROOT_PASSWORD` 环境变量设置的。

以上命令的 `-p 3306:3306` 会将主机的3306端口映射到容器的3306端口，这样可以从主机上的MySQL客户端连接到容器中的MySQL服务。

## 2.3 更改容器信息

### 2.3.1 方法一：停止并删除现有容器

首先，停止并删除现有的 `mysql-test` 容器：
```bash
docker start mysql-test        # 运行
docker stop mysql-test        # 停止
docker rm mysql-test        # 删除
```
然后再重新创建`mysql-test`即可。

### 2.3.2 方法二：不同的容器名称

可以使用不同的名称来运行容器，这样就不需要删除现有的容器。例如：
```bash
docker run --name mysql-test-2 -e MYSQL_ROOT_PASSWORD=0403 -p 3307:3306 -d mysql:latest
```
这样会启动一个新容器，名称为 `mysql-test-2`，并绑定主机的3307端口到容器的3306端口。

## 2.4 查看容器信息

查看容器或镜像内部信息（如端口，ip地址，挂载卷等）：
```
docker inspect <容器名/容器ID/镜像名/镜像ID>
```

# 三、Docker-Ubuntu 配置

## 3.1 运行并创建容器

```bash
docker run -itd --name ubuntu-test ubuntu
```

**注意：**  这里必须加上`-it`。像ubuntu这种镜像在创建容器时需要分配一个伪终端让它在后台持续运行，否则会在启动容器后立马退出运行。

## 3.2 进入容器

```bash
docker exec -it ubuntu-test bash
```

要退出正在交互模式下的容器终端，可以执行以下操作：
1. 按下 `Ctrl + D` 快捷键。
2. 或者在容器终端中键入 `exit` 命令并按回车键。
