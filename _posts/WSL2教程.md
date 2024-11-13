---
title: WSL2 的使用教程
date: 2024-11-12 21:16:31
tags:
    - "WSL"
    - "linux"
excerpt: "本文主要介绍 WSL2 的使用教程，以及需要注意的坑点。"
---

# 1.安装和升级 WSL2

## 1.1 安装前的准备

首先需要在`控制面板 -> 程序 -> 启用或关闭Windows功能`处开启`Hyper-V`和`适用于Windows的Linux子系统`，然后重启电脑即可。默认有2个版本，1和2，1是使用和主机相同的 IP 地址；2是更加独立的 Linux 子系统，有单独的 IP 地址，通过 Windows 主机访问互联网，建议安装2，可以用`systemd`。如果安装2还需要安装一个软件：[wsl_update_x64](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi),该软件来自于[Windows 官网](https://learn.microsoft.com/zh-cn/windows/wsl/install-manual#step-4---download-the-linux-kernel-update-package)。

## 1.2 升级 WSL2

设置默认版本：
```bash
wsl --set-default-version 2
```

更新版本：
```bash
wsl --update
```
这种方式大概率会卡住，建议离线下载安装：[microsoft/WSL](https://github.com/microsoft/WSL)，下载好后直接双击安装即可。

验证是否安装成功：
```bash
wsl --version
```
输入这行指令有这样的输出，表示安装成功：
```txt
WSL 版本： 2.3.24.0
内核版本： 5.15.153.1-2
WSLg 版本： 1.0.65
MSRDC 版本： 1.2.5620
Direct3D 版本： 1.611.1-81528511
DXCore 版本： 10.0.26100.1-240331-1435.ge-release
Windows 版本： 10.0.19045.2673
```


# 2.WSL 的指令

# 2.1 常用指令

- 搜索可安装版本
```bash
wsl --list --online
```

- 安装指定的版本
```bash
wsl --install -d Ubuntu
```

- 查看安装的版本信息
```bash
wsl -l -v
```

- 关闭 wsl
```bash
wsl --shutdown
```

## 2.2 备份与还原

下面的一系列操作以发行版`Ubuntu-22.04`为例。

- 停止指定的 WSL
```bash
wsl -t Ubuntu-22.04
```

- 开启指定的 WSL
```bash
wsl -d Ubuntu-22.04
```

- 导出（备份）WSL 子系统（需要先停止 WSL 子系统）
```bash
wsl --export Ubuntu-22.04 D:\Ubuntu-22.04.tar
```
`D:\Ubuntu-22.04.tar`是导出的路径。

- 卸载 WSL 子系统
```bash
wsl --unregister Ubuntu-22.04
```

- 导入（还原）WSL 子系统
```bash
wsl --import Ubuntu-22.04 D:\WSL D:\Ubuntu-22.04.tar
```
`D:\WSL`是导入路径，`D:\Ubuntu-22.04.tar`是前面备份子系统的路径。

还原后的子系统一般默认是`root`用户，需要修改为其它用户。只要修改`/etc/wsl.conf`配置文件即可，如下：
```bash
nano /etc/wsl.conf
```
加入下面的内容：
```txt
[user]
default=用户名
```
然后按前面的方法进行重启 WSL 子系统即可。


# 3.WSL 的配置

## 3.1 启用 systemd

编辑`/etc/wsl.conf`配置文件。
```bash
nano /etc/wsl.conf
```

在文件中添加下面的内容：
```txt
[boot]
systemd=true
```

## 3.2 更换内核

在 Windows 下的资源管理器中的地址栏输入`%UserProfile%`，然后按回车。在里面新建一个`.wslconfig`文件（如果没有的话）。在文件中天下下面的内容：
```txt
[wsl2]
kernel=e:\\wsl2linuxkernel\\bzImage
```
这里的`e:\\wsl2linuxkernel\\bzImage`是内核路径，可以自己编译内核，手动编译内核参考官方仓库：[microsoft/WSL2-Linux-Kernel](https://github.com/microsoft/WSL2-Linux-Kernel)。也可以用仓库里线现成的：[Nevuly/WSL2-Linux-Kernel-Rolling](https://github.com/Nevuly/WSL2-Linux-Kernel-Rolling)。


# 4.释放 WSL 占用的磁盘空间

有时后会发现，你在 WSL 中删掉了一些文件，但是磁盘剩余空间并没有增加。这是因为 WSL 的磁盘是动态扩容的，但不会自动释放。

1. 关闭 WSL 中的 Linux 发行版：
```bash
wsl --shutdown
```

2. 在`powershell`或者`cmd`中运行管理计算机的驱动器的 DiskPart 命令：
```bash
diskpart
```

3. 选择虚拟磁盘文件（根据自己实际情况填写路径）：
```bash
select vdisk file="D:\WSL\Ubuntu24.04\ext4.vhdx"
```

4. 只读，附加`vhdx`磁盘镜像文件：
```bash
attach vdisk readonly
```

5. 压缩 vhdx 磁盘镜像文件：
```bash
compact vdisk
```

6. 分离`vhdx`磁盘镜像文件：
```bash
detach vdisk
```

7. 退出
```bash
exit
```
或者按住`ctrl+c`。


# 5.参考链接

- [Windows 安装 Linux 子系统 wsl](https://xujinzh.github.io/2023/08/04/windows-wsl-install/index.html)
- [wsl2子系统的备份和还原](https://www.cnblogs.com/Chary/p/18011740)
- [WSL2 Arch+Docker个人配置过程和踩坑记录，以及一些建议](https://blog.azurezeng.com/wsl2-arch-docker-configuration/)
- [wsl2编译升级linux内核](https://nxdong.com/wsl-update-kernel/)
- [在WSL2中删除文件后，磁盘空间未释放怎么办](https://blog.csdn.net/qq_23865133/article/details/141642123)