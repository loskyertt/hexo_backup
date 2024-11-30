---
title: Linux 下 Grub 引导配置教程以及更换内核
date: 2024-09-03 19:46:39
tags:
   - "linux"
excerpt: "`grub`如何配置，如何切换内核（本文以 EndeavourOS 为例）"
---


# 1.基本说明

`GRUB`（GRand Unified Bootloader）的配置文件 是`/etc/default/grub`，用于配置 GRUB 引导加载器的行为。
```bash
sudo nano /etc/default/grub
```

如果卡在启动界面时（满屏代码的界面，大概率是N卡驱动问题），可以同时按住`ctrl`+`alt`+`F2`（也可能是`F3`）进入`tty`，然后输入账户名和密码。

# 2.文件内容说明

## 2.1 基本选项

1. **`GRUB_DEFAULT='0'`:**
   - 指定默认启动的菜单项编号。`0` 表示默认启动第一个菜单项（通常是最新的内核版本）。
   - 如果你想让 GRUB 记住上次选择的内核，可以将这个值改为 `'saved'` 并取消下面的 `GRUB_SAVEDEFAULT=true`。

2. **`GRUB_TIMEOUT='5'`:**
   - 设置 GRUB 菜单在引导时等待用户选择的超时时间（以秒为单位）。`5` 意味着在 5 秒后，如果没有用户干预，系统将启动默认选项。

3. **`GRUB_DISTRIBUTOR='EndeavourOS'`:**
   - 设置 GRUB 在启动菜单中显示的发行版名称，我这里的是 `EndeavourOS`。

4. **`GRUB_CMDLINE_LINUX_DEFAULT='nowatchdog nvme_load=YES nvidia_drm.modeset=1 loglevel=3'`:**
   - 为 Linux 内核指定启动参数：
     - `nowatchdog`：禁用硬件看门狗（防止系统重启）。
     - `nvme_load=YES`：确保 NVMe 驱动程序加载（通常用于 NVMe SSD）。
     - `nvidia_drm.modeset=1`：启用 NVIDIA DRM 模式设置，用于支持 Wayland 或改进图形性能。
     - `loglevel=3`：设置内核日志级别，`3` 意味着只显示错误信息，减少日志输出。

5. **`GRUB_CMDLINE_LINUX=""`:**
   - 这是为 Linux 内核指定的额外引导参数，当前没有设置任何额外的参数。

## 2.2 模块和加密相关

6. **`GRUB_PRELOAD_MODULES="part_gpt part_msdos"`:**
   - 预加载 GRUB 的模块，以便支持 GPT 和 MBR 分区表。这样可以确保 GRUB 能识别各种分区类型，无论是 GPT（GUID Partition Table）还是 MBR（Master Boot Record）。

7. **`#GRUB_ENABLE_CRYPTODISK=y`（已注释）:**
   - 如果取消注释并设为 `y`，GRUB 将启用对 LUKS 加密磁盘的支持。

## 2.3 超时样式和终端设置

8. **`GRUB_TIMEOUT_STYLE=menu`:**
   - 设置 GRUB 菜单的超时样式。`menu` 表示显示菜单，并允许用户进行选择。

9. **`GRUB_TERMINAL_INPUT=console`:**
   - 启用基本控制台输入，即通过键盘输入。

10. **`#GRUB_TERMINAL_OUTPUT=console`（已注释）:**
   - 取消注释后，GRUB 将禁用图形模式，改用基本的文本输出模式。这是用于系统中没有图形显示器或需要调试的情况。

## 2.4 图形相关

11. **`GRUB_GFXMODE=auto`:**
   - 设置 GRUB 的图形模式为 `auto`，让 GRUB 自动选择适合当前显示器和显卡的分辨率。

12. **`GRUB_GFXPAYLOAD_LINUX=keep`:**
   - 让内核继承 GRUB 使用的分辨率，而不是在加载内核后改变分辨率。

## 2.5 高级和恢复模式

13. **`GRUB_DISABLE_LINUX_UUID=true`（已注释）:**
   - 如果取消注释，GRUB 将使用传统的 `/dev/sdX` 设备标识符代替 UUID（全局唯一标识符）来指定启动分区。

14. **`GRUB_DISABLE_RECOVERY='true'`:**
   - 禁用 GRUB 菜单中的恢复模式条目，这可以减少启动菜单的选项，并且在不需要恢复模式时保持简洁。

## 2.6 外观设置

15. **`GRUB_BACKGROUND='/usr/share/endeavouros/splash.png'`:**
   - 设置 GRUB 启动菜单的背景图片，这里指定了 EndeavourOS 的启动背景图片路径 `/usr/share/endeavouros/splash.png`。

16. **`#GRUB_THEME="/path/to/gfxtheme"`**（已注释）：
   - 允许设置一个主题文件，用于美化 GRUB 菜单。取消注释并指定主题路径即可。
   - 推荐的主题：[GradientGuy](https://www.pling.com/p/2025649/)，下载好后解压到`/boot/grub/themes/ `下，然后找到`GradientGuy`主题文件中的`theme.txt`，把完整路径`/boot/grub/themes/GradientGuy/theme.txt`填入到`GRUB_THEME`。

## 2.7 其他选项

17. **`#GRUB_INIT_TUNE="480 440 1"`（已注释）:**
   - 如果取消注释并设置音调、频率和持续时间，可以让 GRUB 在启动时发出蜂鸣声。

18. **`#GRUB_SAVEDEFAULT=true`（已注释）:**
   - 如果取消注释并启用，同时将 `GRUB_DEFAULT` 设置为 `'saved'`，GRUB 将记住上次选择的启动选项。

19. **`GRUB_DISABLE_SUBMENU='false'`:**
   - 禁用子菜单选项。如果设置为 `'true'`，则 GRUB 将不显示内核的子菜单项，例如高级选项菜单。

20. **`#GRUB_DISABLE_OS_PROBER=false`（已注释）:**
   - 取消注释并将其设置为 `false` 时，GRUB 会启用操作系统探测器（os-prober），从而在 GRUB 菜单中显示其他已安装的操作系统（例如 Windows 或其他 Linux 发行版）。

21. **`GRUB_EARLY_INITRD_LINUX_STOCK=''`:**
   - 指定提前加载的 `initrd`，通常用于特定硬件或驱动程序初始化。当前设置为空，表示未指定。

# 3.内核更换

下载内核，这里以`linux-zen`内核为例，我的系统默认是`6.10.7-arch1-1`内核
```bash
sudo pacman -S linux-zen
```

然后修改`grub`配置文件：
```bash
sudo nano /etc/default/grub
```

主要修改三个地方：
- `GRUB_DEFAULT='0'`改为`GRUB_DEFAULT=saved`
- `#GRUB_SAVEDEFAULT=true`取消前面的`#`
- `GRUB_DISABLE_SUBMENU='false'`改为`GRUB_DISABLE_SUBMENU='true'`或者`GRUB_DISABLE_SUBMENU=y`（反正都一个意思）

保存并退出后，生成（或更新）GRUB 配置文件
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
重启电脑就能看到生效了。

# 4.切换内核的注意点

可能会遇到驱动程序（尤其是 N 卡驱动）不兼容的情况，如果卡在加载界面，只能进入`tty`进行排查问题，或者重启选择原来的内核（所以不要随便删原来默认的内核）。要卸载内核的话需要谨慎，因为有些依赖包可能只能依赖特定内核使用，所以卸载之前建议做好备份。

# 4.参考教程

- [How to Change the Default Grub Bootlaoder Theme in Arch Linux!](https://www.youtube.com/watch?v=1vLGebdIPZ8&list=PL7XUZPsvfw3DfZDpU3bqNYZWzKroAWaT5&index=6)
- [How to Switch Arch Linux Kernels - LTS, Zen, Hardened](https://www.youtube.com/watch?v=KbcUmAlQCHs&list=PL7XUZPsvfw3DfZDpU3bqNYZWzKroAWaT5&index=5)