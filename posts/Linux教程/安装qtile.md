---
title: Linux 如何安装 qtile 平铺窗口
date: 2024-12-14 21:00:29
tags:
  - "linux"
excerpt: "注意：该教程用的发行版示例是 Arco Linux ，已经安装好了系统（只有 tty），现在安装 qtile 平铺管理器教程。"
---

# 1.安装 qtile

1. 安装`qtile`：
```bash
sudo pacman -S arcolinux-qtile-distrotube-git
```

2. 拉取`arco-qtile`仓库：
```bash
git clone https://github.com/arcolinuxd/arco-qtile.git
```

3. 依次执行里面的脚本（下面这个是美化脚本）：
```bash
./100-display-manager-and-desktop.sh
```

其它脚本按需选择执行，然后重启系统即可。

效果图如下：
![PixPin_2024-12-14_21-10-14.png](https://s2.loli.net/2024/12/14/wKk2ijJC79qxm8h.png)