---
title: 使用Docker时的问题汇总和解决方式
date: 2024-07-24 10:57:16
tags:
    - "docker"
excerpt: false
---


# 1.启动 Docker 失败

如果发现`docker`启动失败，可以通过以下命令输出`docker`运行日志来定位问题所在：
```bash
sudo journalctl -u docker.service --no-pager | tail -n 50
```

如果问题仍无法解决，可以尝试更彻底的方法：

1. 卸载 Docker：

```bash
sudo pacman -R docker
```

2. 删除 Docker 相关的所有数据和配置：

```bash
sudo rm -rf /var/lib/docker
sudo rm -rf /etc/docker
sudo rm -rf ~/.docker
```

3. 重新安装 Docker：

```bash
sudo pacman -S docker
```

4. 启动 Docker 服务：

```bash
sudo systemctl start docker
```

5. 检查 Docker 服务状态：

```bash
sudo systemctl status docker
```

## 1.1 网络冲突导致启动失败

日志内容如下：
```TXT
failed to start daemon: Error initializing network controller: error creating default "bridge" network: cannot create network c13e4912d03075485921194eecec303af8f714e77ad529759990f5b7ca155c68 (docker0): conflicts with network 1b9dcc9575e5af33281149149481d0eca4a5d60de0fac2f9264f0b265ab955db (docker0): networks have same bridge name
```
这个错误表明 Docker 试图创建默认的 bridge 网络，但是遇到了冲突，因为已经存在一个同名的网络。

**解决办法：**

1. 停止 Docker 服务：

```bash
sudo systemctl stop docker
```

2. 删除现有的 Docker 网络配置：

```bash
sudo rm -rf /var/lib/docker/network
```

3. 删除 docker0 网桥：

```bash
sudo ip link delete docker0
```

4. 重新启动 Docker 服务：

```bash
sudo systemctl start docker
```

5. 检查 Docker 服务状态：

```bash
sudo systemctl status docker
```