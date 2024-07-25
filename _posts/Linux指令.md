---
title: Linux 指令
date: 2024-07-24 08:51:17
tags:
  - "linux"
excerpt: "主要用于记录下 Linux 指令，主要还是 EndeavourOS，不过除了包管理器，其它指令对各个发行版来说应该大差不差，当然记录的目的是为了方便自己查找。"
cover: https://softwarelab.org/wp-content/uploads/Linux.jpg
---


# 一、包管理器

## 1.1 介绍

  在 `EndeavouOS`（以及其他基于`Arch`的发行版）中，`pacman` 是一个用于软件包管理的命令行工具。下面是 `pacman` 命令的详细解释：

   - `-S`：用于安装软件包。
   - `-y`：强制刷新软件包数据库。
   - `-yy`：强制刷新所有的软件包数据库。通常只需要一个 `-y` 就足够，但 `-yy` 用于解决某些情况下可能出现的数据库同步问题。
   - `-u`：更新所有已安装的软件包，用于更新系统或者`pacman`包含的软件以及以来库。

对于所有用`pacman`进行的操作，都可以通过`yay`实现：
```bash
sudo pacman yay
```
`yay`支持的软件多一些（通过`aur`源）。

## 1.2 下载和更新

- **要更新或下载指定的软件包：**
```bash
sudo pacman -S 包名
```
`pacman` 会检查你指定的软件包是否有新版本，如果有的话，就会下载并安装更新后的版本。

## 1.3 查看包/库信息

- **查看包名信息**：
  显示指定软件包的信息，但不会安装或更新。这个选项可以用来查看某个软件包的详细信息，包括它的版本、依赖关系等。例如：
```bash
pacman -Si <pkgname>
```
查看包的简略信息
```bash
yay -Qs <pkgname>
```
查看包的详细信息
```bash
yay -Qi <pkgname>
```

- **查看可升级的包/库：**
列出有可用更新的已安装软件包及其最新版本。
```bash
sudo pacman -Qu
```

列出所有的外部软件包（即非官方仓库安装的包，如`AUR`软件包）：
```bash
pacman -Qm
```

## 1.4 卸载

### 1.4.1 卸载单个软件包

要卸载单个软件包，可以使用以下命令：

```bash
sudo pacman -R package_name
```

### 1.4.2 卸载软件包及其未使用的依赖

有时卸载一个包后，它的一些依赖包可能不再被其他软件包使用。要卸载软件包及其未使用的依赖，可以使用以下命令：

```bash
sudo pacman -Rns package_name
```

解释：
- `-R`(--remove)：卸载指定的包。
- `-n`(--nosave)：从系统中删除安装包的所有配置文件。
- `-s`(--recursive)：递归地卸载未使用的依赖包。

### 1.4.3 强制卸载（不推荐）

在极少数情况下，可能需要强制卸载一个包，即使这可能会破坏系统的依赖关系。请谨慎使用此选项：

```bash
sudo pacman -Rdd package_name
```

解释：
- `-d`：忽略依赖关系检查。

### 1.4.4 清理未使用的孤立包

系统中可能会有一些未使用的孤立包，这些包是作为依赖安装的，但现在没有任何包依赖它们。可以使用以下命令清理这些孤立包：

```bash
sudo pacman -Rns $(pacman -Qtdq)
```

解释：
- `pacman -Qtdq`：列出所有未使用的孤立包。
- `-Rns`：递归地卸载未使用的包及其配置文件。



# 二、常用指令

## 2.1 查看开机启动项的方法

### 2.1.1 使用 systemctl 查看 systemd 服务

使用 `systemctl` 命令来列出所有启用的（开机自启动的）systemd 服务：
```bash
systemctl list-unit-files --type=service --state=enabled
```

查看当前运行的服务：
```bash
systemctl list-units --type=service --state=running
```

你也可以使用 `systemctl` 查看所有启用的单元文件，包括服务、套接字、目标等：
```bash
systemctl list-unit-files --state=enabled
```

### 2.1.2 查看用户和系统级别的开机自启动应用

用户级别的开机自启动应用通常在 `~/.config/autostart` 目录下。你可以使用以下命令查看该目录中的内容：

```bash
# 用户
ls ~/.config/autostart

# 系统
ls /etc/xdg/autostart
```

### 2.1.3 使用 crontab 查看定时任务

你还可以使用 `crontab` 查看是否有任何定时任务设置为在启动时运行。使用以下命令查看当前用户的 `crontab` 条目：

```bash
crontab -l
```

如果有需要在系统启动时运行的任务，它们通常会使用 `@reboot` 时间标志。

### 2.1.4 检查 /etc/rc.local 文件

虽然 `rc.local` 文件在现代系统中不再常用，但有时仍然会使用它来设置开机自启动任务。你可以检查这个文件（如果存在）来查看是否有任何任务设置为在启动时运行：

```bash
cat /etc/rc.local
```

## 2.2 文件操作

### 2.2.1 touch 指令

创建文件：
```bash
touch <filename>
```

### 2.2.2 mkdir 指令

创建文件夹：
```bash
mkdir <foldername>
```
强行创建，比如当`<folder1>`不存在时：
```bash
mkdir -p <folder1>/<folder2>
```

### 2.2.3 rm 指令

删除文件：
```bash
rm <filename1> <filename2>
```

删除该文件夹下的所有子文件
```bash
rm <foldername>/ -rf
```

删除当前文件夹的所有文件
```bash
rm * -rf
```

**注：** 后缀的`-rf`是指强制删除（不会有警告），`-r`是指递归普通删除（若与其它文件有链接，会提出警告）。

### 2.2.4 cp 指令

复制文件夹操作需要加`-r`：
```bash
cp <foldername1> <foldername2> -r
```

### 2.2.5 mv 指令

可以对文件或者文件夹进行重命名：
```bash
mv test01.txt test02.txt
```
如果`test01.txt`存在，`test02.txt`不存在，会把`test01.txt`命名为`test02.txt`。

对文件或者文件夹进行移动操作：
```bash
mv /test.txt ~/temp
```
把`test.txt`移动到`~/temp`文件夹下。

### 2.5.6 加上 -r 的用处

总结下`rm`、`cp`、`mv`这三种指令，对文件夹操作都会加上`-r`。这里表示`加上`-r`（或`--recursive`）选项后，这些命令会递归地操作目录及其内容。

## 2.3 赋予文件或文件夹权限

1. 使用`chmod`命令递归地更改文件夹内所有文件和子文件夹的权限：
```bash
chmod -R 777 /path/to/your/folder
```
`-R`表示递归。

2. 更改文件的权限而不更改文件夹的权限，可以结合`find`命令来实现：
```bash
find /path/to/your/folder -type f -exec chmod 777 {} +
```
这将只更改所有文件的权限，而不影响文件夹的权限。

3. 单独更改文件夹的权限（例如，给文件夹777权限，而文件保持不变）：
```bash
find /path/to/your/folder -type d -exec chmod 777 {} +
```


# 三、提高编译速度的方法

## 3.1 增加编译并行度
可以通过设置更多的并行编译任务来加速编译过程。使用与 CPU 核心数量相等或更多的并行任务数。设置 MAKEFLAGS 来实现这一点：

编辑 /etc/makepkg.conf 文件，找到以下行并进行修改：
```
MAKEFLAGS="-j$(nproc)"
```
`$(nproc)`会自动检测 CPU 核心数，并设置相同数量的并行任务。包括在执行`make`指令时，可以通过加`-j<核心数>`来手动指定编译时用CPU的核心数。
