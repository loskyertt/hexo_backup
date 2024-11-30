---
title: 解决 Ubuntu 下中文乱码问题
date: 2024-11-30 21:46:27
tags:
  - "linux"
  - "docker"
excerpt: "解决 Ubuntu 下中文乱码的问题，主要是在 docker 的 ubuntu 镜像中。"
---

# 1.安装中文字体包

```bash
sudo apt-get install language-pack-zh-hans
```

# 2.修改配置文件

注：下列操作在`docker`的`ubuntu`容器中不需要加`sudo`，默认就是`root`权限。

修改`/etc/environment`配置文件：
```bash
sudo nano /etc/environment
```

加入下面的内容：
```txt
LANG="zh_CN.UTF-8"
LANGUAGE="zh_CN:zh:en_US:en"
```

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