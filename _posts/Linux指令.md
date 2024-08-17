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

## 2.4 解压缩

`tar` 是一个常用的 Unix 和 Linux 工具，用于归档（打包）和压缩文件。

参数如下：

- **`-z`** ：通过 `gzip` 压缩或解压缩数据。这个选项告诉 `tar` 文件是经过 `gzip` 压缩的。用于处理`.tar.gz`或`.tgz`文件。
- **J** ：使用`xz`来处理`.xz`格式的压缩文件。
- **`-x`** ：解压缩归档文件。这个选项指示 `tar` 从归档文件中提取文件。
- **`-v`** ：显示详细的输出。在解压缩或打包过程中，它会列出处理的文件名。
- **`-f`** ：指定归档文件的名称。这个选项后面需要跟上归档文件的名称。

### 使用示例

假设你有一个压缩的 tar 文件 `archive.tar.gz`，你可以使用以下命令来解压它：

```bash
tar -zxvf archive.tar.gz
```

## 2.5 切换为 root 用户

### 2.5.1 使用 `su` 命令

`su`（switch user）命令可以用来切换到其他用户，包括 `root` 用户。

```bash
su -
```

- **`-`** 选项（或 `--login`）会使 `su` 命令模拟一个完整的登录 shell，设置环境变量如 `PATH` 和 `HOME`，相当于从登录开始。
- 执行命令后，你会被提示输入 `root` 用户的密码。如果密码正确，你会切换到 `root` 用户。

如果你只想切换到 `root` 用户而不加载 `root` 用户的环境配置，可以省略 `-` 选项：

```bash
su
```

### 2.5.2 使用 `sudo` 命令

`sudo` 命令允许以 `root` 用户或其他用户的身份执行命令。要切换到 `root` 用户，你可以使用以下命令：

```bash
sudo -i
```

- **`-i`** 选项使 `sudo` 模拟一个登录 shell，切换到 `root` 用户并加载 `root` 用户的环境变量。

如果你只想以 `root` 用户身份执行单个命令，可以直接在命令前加上 `sudo`：

```bash
sudo command
```

例如：

```bash
sudo ls /root
```

这会以 `root` 用户的权限执行 `ls /root` 命令。

### 2.5.3 使用 `sudo su` 命令

你也可以通过 `sudo` 命令切换到 `root` 用户，使用以下命令：

```bash
sudo su -
```

- **`sudo su -`** 组合命令先用 `sudo` 获得临时的 `root` 权限，再用 `su -` 切换到 `root` 用户。

### 2.5.4 注意事项

- **`root` 密码** ：你需要知道 `root` 用户的密码来使用 `su` 命令。`sudo` 需要你当前用户在 `sudoers` 文件中有权限执行 `sudo` 命令，且你需要输入你自己的用户密码。
- **权限管理** ：在某些系统（如 Ubuntu）默认不启用 `root` 用户的密码，而是通过 `sudo` 提供管理员权限。你可能需要使用 `sudo` 命令而不是 `su` 来获得 `root` 权限。

## 2.6 查看文件大小

### 2.6.1 使用 `du` 命令

`du`（disk usage）命令用于显示文件和目录的磁盘使用情况：

```bash
du -sh /path/to/directory
```

**参数解释：**
- **`-s`** ：显示总计，只显示指定目录的总大小，而不是递归显示每个子目录。
- **`-h`** ：以人类可读的格式显示大小（如 KB, MB, GB），使输出更易读。

### 2.6.2 使用 `du` 命令按大小排序

如果你想按大小排序显示子目录，可以使用以下命令：

```bash
du -h /path/to/directory | sort -h
```

### 2.6.3 使用 `ls` 命令查看目录大小

尽管 `ls` 命令主要用于列出目录中的文件，它也可以用来查看目录的大小，但它的显示可能不如 `du` 直观。

```bash
ls -lh /path/to/directory
```

- **`-l`**：长格式列出文件信息，包括大小。
- **`-h`**：以人类可读的格式显示文件大小。

# 三、提高编译速度的方法

## 3.1 增加编译并行度
可以通过设置更多的并行编译任务来加速编译过程。使用与 CPU 核心数量相等或更多的并行任务数。设置 MAKEFLAGS 来实现这一点：

编辑 /etc/makepkg.conf 文件，找到以下行并进行修改：
```
MAKEFLAGS="-j$(nproc)"
```
`$(nproc)`会自动检测 CPU 核心数，并设置相同数量的并行任务。包括在执行`make`指令时，可以通过加`-j<核心数>`来手动指定编译时用CPU的核心数。
