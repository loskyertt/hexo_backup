---
title: Dockerfile 教程
date: 2024-09-03 12:34:40
tags:
  - "docker"
  - "proxy"
excerpt: "写 Dockerfile 的一些技巧。"
---


# 1.代理配置

有时侯在构建镜像时，需要执行一些下载操作，没有配置国内镜像源的话，下载速度可能会很慢甚至出错，所以需要在 Dockerfile 中配置代理。

在 Dockerfile 中加上：
```Dockerfile
ENV http_proxy=http://your-proxy-address:port/
ENV https_proxy=http://your-proxy-address:port/
```

`http_proxy`和`https_proxy`：这两个环境变量分别用于`HTTP`和`HTTPS`的代理服务器地址。你需要将`your-proxy-address:port`替换为宿主机代理的实际地址和端口。注意这里是宿主机的实际`IP`地址，不是本地回环（`loop`）地址。同过`ifconfig`或者`ip a`查看。