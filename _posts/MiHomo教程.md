---
title: MiHomo(Clash Meta) 纯内核使用教程
date: 2024-07-25 06:49:04
tags:
    - "docker"
    - "linux"
    - "proxy"
    - "clash"
excerpt: "这篇文章主要是记录自己使用 Clash Meta 的一些心得，因为强迫症的原因，所以是通过 docker 来部署整个代理服务的。"
---


# 1.前言（说明）

在 [github](https://github.com/) 上其实有不少的客户端和服务端的软件，如果不喜欢折腾的话，这篇文章可能不怎么适合你，建议直接使用第三方客户端的代理软件。这篇文章主要是记录自己使用 mihomo 的一些心得，因为强迫症的原因，所以是通过 docker 来部署整个代理服务的。当然，用 docker 部署也是有好处的，配置好后可以连同镜像一起打包备份或者`push`到 [Docker Hub](https://hub.docker.com/) 上，后续电脑环境改变了仍然能使用。

# 2.下载必要的文件

先把存放下载文件的项目文件夹创建好：
```bash
mkdir ~/clashmeta && cd clashmeta
```
后续下载好的文件都放在`clashmeta`文件夹下。

## 2.1 内核

[稳定版下载地址](https://github.com/MetaCubeX/mihomo/releases)

我这里用的是 debian 镜像，所以下载的是这个版本：
![image01.png](https://s2.loli.net/2024/07/25/n5DSvEiMquTJ7fC.png)

## 2.2 GeoIP & GeoSite 库

[GeoIP 库下载](https://gcore.jsdelivr.net/gh/MetaCubeX/meta-rules-dat@release/geoip.dat)
[GeoSite 库下载](https://gcore.jsdelivr.net/gh/MetaCubeX/meta-rules-dat@release/geosite.dat)

## 2.3 config.yaml 配置文件

[config.yaml 下载](https://github.com/loskyertt/clash_meta/blob/master/config.yaml)

需要把`config.yaml`这部分内容修改以下：
![image02.png](https://s2.loli.net/2024/07/25/uwHzqYafTrVlxE2.png)

填上自己的机场链接即可。

## 2.4 UI 文件

[ui 下载](https://github.com/loskyertt/clash_meta/blob/master/ui.tgz)

# 3.拉取镜像

mihomo 的作者在 Docker Hub 上是有镜像的，但是里面没有说明文档，不知道怎么用，所以就没用它的镜像。

直接拉取最新的稳定版本的 debian 镜像即可：
```bash
docker pull debian
```

# 4.开始配置

## 4.1 创建容器
```bash
docker run -it --name clashmeta -p 7890:7890 -p 9090:9090 -v ~/clashmeta:/root/clashmeta debian:latest /bin/bash
```
此时直接进入了创建好的容器内部。
**说明：** 这里的`7890`端口是代理端口，`9090`端口是本地网站端口。

## 4.2 容器内的操作

```bash
cd /root/clashmeta
```

执行：
```bash
dpkg -i mihomo-linux-amd64-v1.18.6.deb
```
把`mihomo`安装到容器内。可以通过`which mihomo`来查看是否安装成功。

开启代理，输入：
```bash
 mihomo -d ./
```
然后打开浏览器输入`http://127.0.0.1:9090/ui`就能进入代理界面。

# 5.使用现成的镜像配置

如果以上步骤觉得太麻烦，可以使用我推送到 Docker Hub 上的镜像：
```bash
docker pull loskyertt/clashmeta:debian-v1.8.16
```

然后先创建临时容器：
```bash
docker run -it --name=temp loskyertt/clashmeta:debian-v1.8.16 /bin/bash
```
然后输入`exit`退出。

把`clashmeta`文件复制到宿主机：
```bash
docker cp temp:/root/clashmeta_backup/ ~/
```

重命名：
```bash
mv clashmeta_backup clashmeta
```

可以把该临时容器停止并删除：
```bash
docker stop temp && docker rm temp
```

重新创建一个容器，可以实现代理（记得修改`config.yml`文件）：
```bash
docker run -it --name clashmeta -p 7890:7890 -p 9090:9090 -v ~/clashmeta:/root/clashmeta loskyertt/clashmeta:debian-v1.8.16 /bin/bash
```

开启代理
```bash
cd /root/clashmeta

mihomo -d ./
```
然后打开浏览器输入`http://127.0.0.1:9090/ui`就能进入代理界面。
![image03.png](https://s2.loli.net/2024/07/25/ulftdepWhZK3YR4.png)

![image04.png](https://s2.loli.net/2024/07/25/qxW3TZfYP5lBS9p.png)

# 6.注意事项

如果发现仍然不能科学上网，可以打开“设置”检查下“代理设置”，手动设置代理地址和端口`127.0.0.1:7890`（火狐浏览器可能需要在内置的浏览器设置中进行配置下）。对于 Windows 用户来说，其实操作逻辑都是一样的（当然也可以不用 docker ），配置完后注意设置防火墙就行。

# 6.参考教程

- [【进阶使用】Clash Meta 纯内核使用教程|多机场融合|规则自动更新|Tun虚拟网卡模式|避免DNS泄露|WebRTC泄露](https://www.youtube.com/watch?v=d-2vCYLjXHs&t=23s)