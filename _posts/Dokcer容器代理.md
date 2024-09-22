---
title: 在 Docker 容器中使用宿主机的代理
date: 2024-08-16 13:45:53
tags:
  - "docker"
  - "proxy"
excerpt: "因为有时候需要在容器里进行下载任务，所以需要在容器里配置代理。"
---

# 1.拉取镜像

这里以`ubuntu`镜像为例子：
```bash
docker pull ubuntu
```

# 2.容器内配置代理的方式

## 2.1 方式一（host 模式）

运行完后退出容器：
```bash
docker  run -it --name=test --network=host ubuntu
```

因为在容器里默认是没有安装编辑器的，要么通过在容器内下载（`apt install vi`），要么通过在宿主机里对文件进行编辑后再通过`docker cp`复制到容器里。

编辑宿主机的`.bashrc`文件，在里面添加代理配置：
```bash
kate ~/.bashrc
```

打开后在里面添加：
```txt
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
`7890`是`clash`的代理端口。

然后把`.bashrc`复制到`test`容器的`/root`下：
```bash
docker cp ~/.bashrc test:/root/
```

可以重启下容器：
```bash
docker restart test
```

进入容器：
```bash
docker exec -it test bash
```

开启并查看代理：
```bash
# 开启代理
proxy

# 查看代理
env | grep -i proxy
```

### 2.1.1 原理

***一、`host` 网络模式的特点***

1. **共享网络堆栈** ：
   - 容器与宿主机共享网络堆栈，因此容器和宿主机的网络接口和端口是直接共享的。
   - 容器中的服务可以直接通过宿主机的 IP 地址和端口访问，不需要额外的端口映射。

2. **无需端口映射**：
   - 在 `host` 网络模式下，你不需要使用 `-p` 或 `--publish` 选项来映射端口。
   - 容器中的服务将直接监听宿主机的端口，可以通过宿主机的 IP 地址直接访问。

3. **网络性能** ：
   - 由于没有虚拟网络桥接的开销，`host` 网络模式通常提供更高的网络性能。

4. **网络隔离** ：
   - 使用 `host` 网络模式时，容器与宿主机网络没有隔离，这意味着容器可以访问宿主机上所有可访问的网络资源，也可能对宿主机网络造成安全风险。

***二、注意事项***

- **安全性** ：由于容器与宿主机共享网络，容器可能会有更大的安全风险，因为它可以直接访问宿主机的网络资源。
- **网络冲突** ：如果容器和宿主机上的应用程序尝试监听相同的端口，可能会发生端口冲突。

## 2.2 方式二（bridge 模式）

运行完后退出容器：
```bash
docker run -it --name=test ubuntu
```

这里同样需要修改`.bashrc`文件，但是代理地址不再是宿主机的本地回环地址了，惹事宿主机的实际 `ip`地址。

查询宿主机的`ip`地址：
```bash
ip a

# 或者
ifconfig
```

编辑`.bashrc`文件：
```bash
kate ~/.bashrc
```

打开后在里面添加：
```txt
# where proxy
proxy(){
  export http_proxy="http://<宿主机的 ip 地址>:7890"
  export https_proxy="http://<宿主机的 ip 地址>:7890"
  echo "HTTP Proxy on"
}

# where noproxy
noproxy(){
  unset http_proxy
  unset https_proxy
  echo "HTTP Proxy off"
}
```

### 2.2.1 原理

如果是在桥接模式下在容器中使用 `127.0.0.1` 来访问宿主机的服务（如代理服务），这个 IP 地址将指向容器本身，而不是宿主机。因此要让容器访问宿主机上的服务（例如宿主机上的代理服务），应该使用宿主机的实际 IP 地址，或者在 Docker 中使用特定的地址来实现宿主机的访问。
