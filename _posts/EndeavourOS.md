---
title: EndeavourOS 使用记录
date: 2024-07-18 16:21:15
tags:
  - "linux"
excerpt: "这篇文章主要记录自己使用EndeavourOS Linux（Arch系的发行版）的过程，有一些配置，以及遇到的一些问题和解决办法。理论上这些解决办法在其它Linux发行版下也适用。"
cover: https://i0.wp.com/endeavouros.com/wp-content/uploads/2023/10/Endy_planet_ARM.png?resize=1536%2C864&ssl=1
---


# 一、N卡驱动

`EndeavourOS`对N卡用户的支持比较友好，在安装该系统时，选择有`Nvidia`那一行的进行安装即可，会自动安装最新的适应当前显卡的闭源驱动。如果是`Arch Linux`，建议看这篇文章，同时提到有双显卡如何进行切换 [Arch Linux 安装使用教程](https://archlinuxstudio.github.io/ArchLinuxTutorial/#/)，如果是`Arch`系其它发行版（像`Manjaro`），请自行找教程，这里提到的方式不一定适用你。

## 1.1 安装方式一

- **全面更新系统以及安装依赖工具**

```bash
sudo pacman -Syu base-devel dkms
```

对于新版本的显卡型号，直接运行下面这条指令即可：
```bash
# 必须安装
sudo pacman -S nvidia nvidia-settings lib32-nvidia-utils
```

## 1.2 安装方式二

如果安装有`mwhd`工具，可以使用以下方法：

- **自动安装推荐的开源驱动程序**
```bash
sudo mhwd -a pci free 0300
```

- **自动安装推荐的闭源驱动程序：**
```bash
sudo mhwd -a pci nonfree 0300
```

终端输入：
```bash
nvidia-smi
```
如果有返回信息，表示安装成功。

## 1.3 查看显卡信息

简单的过滤，显示有关 VGA 设备的基本信息。
```bash
lspci -vnn | grep VGA
```

提供更详细的信息，包括驱动程序信息。
```bash
lspci -k | grep -EA3 'VGA|3D'
```

提供非常详细且易读的图形信息，适合快速了解系统图形设备和驱动状态。
```bash
inxi -G
```

手动安装的话，从官网下载对应型号的驱动，一定要把下载好的驱动文件放到主文件夹（`~`）下！在`BIOS`终端界面，中文会显示乱码。


## 1.4 补充
- **`nvidia-hook`, `nvidia-inst`, `nvidia-utils` 的用途：**

### 1.4.1 nvidia-hook
`nvidia-hook` 是一个 Pacman 钩子脚本，用于在内核更新时自动重新生成 NVIDIA 内核模块。这可以确保你的 NVIDIA 驱动程序在内核更新后继续正常工作，而无需手动干预。

**功能：**
- 自动在内核更新后重新生成 NVIDIA 内核模块。
- 提高系统更新的便利性，减少因驱动兼容性问题导致的系统启动问题。

### 1.4.2 nvidia-inst
`nvidia-inst` 是 EndeavourOS 提供的一个脚本，用于安装和配置 NVIDIA 驱动程序。它简化了在 EndeavourOS 系统上安装 NVIDIA 驱动的过程。

**功能：**
- 自动检测你的显卡型号。
- 安装合适版本的 NVIDIA 驱动程序。
- 配置系统以使用新的驱动程序。

### 1.4.3 nvidia-utils
`nvidia-utils` 包含 NVIDIA 驱动程序的核心用户空间组件和实用工具。这个包提供了运行 NVIDIA 驱动程序所需的库和命令行工具。

**功能：**
- 提供核心的 NVIDIA 驱动库。
- 包含用于查询和配置 NVIDIA 显卡的命令行工具，如 `nvidia-smi`。
- 包含 OpenGL 库和 Vulkan 库，用于支持高性能图形渲染。

 `nvidia-hook`, `nvidia-inst`, `nvidia-utils` 不能完全替代 `nvidia-settings` 和 `lib32-nvidia-utils`。

### 1.4.4 nvidia-settings
`nvidia-settings` 是一个独立的图形化配置工具，用于调整和配置 NVIDIA 显卡的各种设置。它提供了一个图形界面，允许用户进行以下操作：
- 配置显示器布局（多显示器设置）。
- 调整显卡性能参数（如风扇速度、功率模式）。
- 配置 OpenGL 设置。
- 管理显示器色彩和亮度设置。

**替代情况：**
- `nvidia-utils` 提供了一些命令行工具，但没有图形界面和用户友好的配置选项。
- `nvidia-settings` 提供了更丰富和直观的配置选项，尤其适用于需要频繁调整显卡设置的用户。

### 1.4.5 lib32-nvidia-utils
`lib32-nvidia-utils` 是 32 位 NVIDIA 驱动程序库的集合，用于在 64 位系统上运行 32 位应用程序（如一些旧版游戏和软件）。这些库对于兼容 32 位应用程序至关重要。

**替代情况：**
- `nvidia-utils` 包含 64 位库，不包括 32 位库。
- 如果你需要运行 32 位应用程序，仍然需要安装 `lib32-nvidia-utils`。


# 二、中文输入法配置

## 2.1 推荐方式

```bash
yay -Sy fcitx-im fcitx-configtool
```

```bash
kate ~/.xprofile
```
输入以下内容：
```
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

如果在`~`目录创建的`.xprofile`配置文件没有生效，可以这样做：
```bash
kate /etc/environment
```

然后在里面加入：
```txt
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

## 2.2 其它（没试过）

**方式一：**
```bash
# 安装fcitx5
#基础包组
sudo pacman -S fcitx5-im
#官方中文输入引擎
sudo pacman -S fcitx5-chinese-addons
#日文输入引擎
sudo pacman -S fcitx5-anthy
#萌娘百科词库（挂梯子）
yay -S fcitx5-pinyin-moegirl
#中文维基百科词库
sudo pacman -S fcitx5-pinyin-zhwiki
#主题
sudo pacman -S fcitx5-material-color
```

编辑配置文件：
```bash
kate /etc/environment
```

输入以下内容：
```txt
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
```

# 三、问题汇总

## 3.1 安装软件时的证书问题

1. 首先，更新你的系统证书:

```bash
sudo update-ca-trust
```

2. 如果上面的方法不起作用，你可以尝试临时禁用SSL验证。这不是一个安全的长期解决方案，但可以帮助你确定问题是否确实与证书有关:

```bash
makepkg -si --skippgpcheck
```

3. 最后，确保你的系统时间是正确的，因为时间不正确也可能导致SSL证书验证失败:

```bash
timedatectl status
```

如果时间不正确，可以同步:
```bash
sudo timedatectl set-ntp true
```

## 3.2 镜像源问题

如果下载或者更新速度较慢，可以用 Arch 清华镜像源：
```bash
kate /etc/pacman.d/mirrorlist
```
然后在首行加上：
```txt
# arch 清华镜像源
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
# arch 稳定镜像源
Server = https://mirror.archlinux.org/$repo/os/$arch
Server = https://mirrors.kernel.org/archlinux/$repo/os/$arch
```

**如果是在线安装系统的话，建议在`live`版镜像中，先改下镜像源配置：**

进入`root`用户：
```bash
sudo su
```

```bash
nano /etc/pacman.d/mirrorlist
```
然后把镜像源加进去。

## 3.3 签名验证问题

**建议先参考这篇官方文章：** [Signature and keyring](https://discovery.endeavouros.com/signature-and-keyring/pacman-keyring-issues/2021/03/)

```txt
(735/735) 正在检查软件包完整性                        [------------------------------------] 100% **错误：**libinstpatch: 来自 "Brett Cornwall <brett@i--b.com>" 的签名是未知信任的- **:: 文件 /var/cache/pacman/pkg/libinstpatch-1.1.6-3-x86_64.pkg.tar.zst 已损坏 (无效或已损坏的软件包 (PGP 签名))**. **打算删除吗？ [Y/n] ** **错误：**fluidsynth: 来自 "Brett Cornwall <brett@i--b.com>" 的签名是未知信任的**坏** **:: 文件 /var/cache/pacman/pkg/fluidsynth-2.3.6-1-x86_64.pkg.tar.zst 已损坏 (无效或已损坏的软件包 (PGP 签名)).** **打算删除吗？ [Y/n]
```

如果遇到以上这种问题（比如在执行`sudo pacman -Syu`时），可以这样做：
```bash
kate /etc/pacman.conf
```

然后找到`SigLevel`那一行，暂时禁用签名检查（这是一个临时的、有风险的解决方案），修改为：
```txt
SigLevel = Never
```
为了保证主机安全，在安装好后记得要修改回来！！！

## 3.4 文件未通过校验

比如在构建包时有类似的如下报错信息：
```txt
 正在验证 source 文件，
 使用sha512sums... 
  nekoray ... 通过
  nekobox.sh ... 通过
  nekobox.desktop ... 通过
  1350.patch ... 失败. ==> 错误： 一个或多个文件没有通过有效性检查！0
```

1. **清理之前的构建：**
```bash
makepkg --clean
```

2. **更新 PKGBUILD 文件中的校验和。进入构建文件的目录，然后使用以下命令：**
```bash
updpkgsums
```
它会自动下载源文件（如果需要），计算新的校验和，并更新 PKGBUILD 文件。

3. **重新构建：**
```bash
makepkg -si
```

# 四、美化

## 4.1 推荐全局主题

建议去 KDE Store 下载主题，然后把下载的 **全局主题** 解压后，放到`~/.local/share/plasma/look-and-feel`目录下。

壁纸和全局主题斗比较推荐`Utterly Nord`，图标推荐`Dracula Circle`。（注意：要选择`plasma6`的）

参考图：
[![Utterly Nord](https://s21.ax1x.com/2024/08/25/pAkSS6H.png)](https://imgse.com/i/pAkSS6H)