---
title: 解决 Ubuntu 下中文乱码问题
date: 2024-11-30 21:46:27
tags:
  - "linux"
  - "docker"
excerpt: "解决 Ubuntu 下中文乱码的问题，主要是用于 docker 的 ubuntu 镜像中。"
---

# 1.安装中文字体包

```bash
sudo apt-get install language-pack-zh-hans
```

# 2.修改配置文件

注：下列操作在`docker`的`ubuntu`容器中不需要加`sudo`，默认就是`root`权限。

## 2.1 方式一：修改`~/.bashrc`文件（推荐）

修改`~/.bashrc`文件：
```bash
nano ~/.bashrc
```

添加：
```txt
export LANG=zh_CN.UTF-8
export LANGUAGE=zh_CN:zh
export LC_ALL=zh_CN.UTF-8
```

然后执行：
```bash
source ~/.bashrc
```

## 2.2 方式二：修改`/etc/profile`文件

修改`/etc/profile`配置文件：
```bash
sudo nano /etc/profile
```

添加：
```txt
export LANG=zh_CN.UTF-8
export LANGUAGE=zh_CN:zh
export LC_ALL=zh_CN.UTF-8
```

然后执行：
```bash
source /etc/profile
```
可能每次重启系统后，都得执行这条命令。

## 2.3 方式三：修改`/etc/environment`文件

注：该方式不一定成功。

修改`/etc/environment`配置文件：
```bash
sudo nano /etc/environment
```

加入下面的内容：
```txt
LANG="zh_CN.UTF-8"
LANGUAGE="zh_CN:zh"
LC_ALL="zh_CN.UTF-8"
```

# 3.添加中文配置内容（选做）

再修改`/var/lib/locales/supported.d/local`文件（如果没有，就手动创建）
```bash
sudo nano /var/lib/locales/supported.d/local
```

加入下面内容：
```txt
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_CN.GBK GBK
zh_CN GB2312
```

最后执行：
```bash
sudo locale-gen
```

然后重启系统即可。

# 3.中文空格乱码解决

```bash
sudo apt-get install fonts-droid-fallback ttf-wqy-zenhei ttf-wqy-microhei fonts-arphic-ukai fonts-arphic-uming
```