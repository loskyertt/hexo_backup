---
title: Linux 常用指令汇总
date: 2024-12-10 15:42:05
tags:
  - "linux"
excerpt: "这里的发行版示例是 ArcoLinux，但是在任何发行版上都能使用。"
---


# 1.查看开机启动项

## 1.1 使用 systemctl 查看 systemd 服务

使用 `systemctl` 命令来列出所有启用的（开机自启动的）systemd 服务：
```bash
sudo systemctl list-unit-files --type=service --state=enabled
```

查看当前运行的服务：
```bash
sudo systemctl list-units --type=service --state=running
```

你也可以使用 `systemctl` 查看所有启用的单元文件，包括服务、套接字、目标等：
```bash
sudo systemctl list-unit-files --state=enabled
```

如果是查看当前用户级别的服务，只需要加上参数`--user`即可，不需要`sudo`：
```bash
systemctl --user list-unit-files --type=service
```

## 1.2 查看用户和系统级别的开机自启动应用

用户级别的开机自启动应用通常在 `~/.config/autostart` 目录下。你可以使用以下命令查看该目录中的内容：

```bash
# 用户
ls ~/.config/autostart

# 系统
ls /etc/xdg/autostart
```

## 1.3 使用 crontab 查看定时任务

你还可以使用 `crontab` 查看是否有任何定时任务设置为在启动时运行。使用以下命令查看当前用户的 `crontab` 条目：

```bash
crontab -l
```

如果有需要在系统启动时运行的任务，它们通常会使用 `@reboot` 时间标志。

## 1.4 检查 /etc/rc.local 文件

虽然 `rc.local` 文件在现代系统中不再常用，但有时仍然会使用它来设置开机自启动任务。你可以检查这个文件（如果存在）来查看是否有任何任务设置为在启动时运行：

```bash
cat /etc/rc.local
```

---

# 2.文件操作

## 2.1 touch 指令

创建文件：
```bash
touch <filename>
```

## 2.2 mkdir 指令

创建文件夹：
```bash
mkdir <foldername>
```
强行创建，比如当`<folder1>`不存在时：
```bash
mkdir -p <folder1>/<folder2>
```

## 2.3 rm 指令

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

## 2.4 cp 指令

复制文件夹操作需要加`-r`：
```bash
cp <foldername1> <foldername2> -r
```

## 2.5 mv 指令

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

总结下`rm`、`cp`、`mv`这三种指令，对文件夹操作都会加上`-r`。这里表示`加上`-r`（或`--recursive`）选项后，这些命令会递归地操作目录及其内容。

## 2.6 find 指令

1. **基本语法** ：
```bash
find [路径] [选项] [表达式]
```

2. **查找特定目录下的所有文件** ：
```bash
find /path/to/directory
```
这会列出`/path/to/directory`目录下的所有文件及其子目录下的文件。

3. **根据文件名查找** ：
```bash
find /path/to/directory -name "filename.txt"
```
使用 `-iname` 可以进行不区分大小写的查找：
```bash
find /path/to/directory -iname "filename.txt"
```

4. **查找特定类型的文件** ：
查找所有目录：
```bash
find /path/to/directory -type d
```
查找所有普通文件：
```bash
find /path/to/directory -type f
```

3. **根据文件大小查找** ：
查找大于 1MB 的文件：
```bash
find /path/to/directory -size +1M
```

4. **根据修改时间查找** ：
查找最近 7 天内修改过的文件：
```bash
find /path/to/directory -mtime -7
```

5. **执行命令** ：
找到文件后执行命令，例如删除找到的文件：
```bash
# 删除文件
find /path/to/directory -name "filename.txt" -exec rm {} \;

# 删除文件夹
find /path/to/directory -type d -name 'folder' -exec rm -rf {} +
```

6. **查找并列出文件的详细信息**：
```bash
find /path/to/directory -name "filename.txt" -ls
```

7. **结合多个条件** ：
查找所有 `.txt` 文件并且大小大于 1MB：
```bash
find /path/to/directory -name "*.txt" -size +1M
```

---

# 3.权限

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

4. 把文件改为可执行
```bash
chmod u+x script.sh
```
`u`参数代表文件的所有者`user`，可以不加，但是就会改变组`group`和其他用户的权限。

---

# 4.解压缩

`tar` 是一个常用的 Unix 和 Linux 工具，用于归档（打包）和压缩文件。

参数如下：

- **`-z`** ：通过 `gzip` 压缩或解压缩数据。这个选项告诉 `tar` 文件是经过 `gzip` 压缩的。用于处理`.tar.gz`或`.tgz`文件。
- **J** ：使用`xz`来处理`.xz`格式的压缩文件。
- **`-x`** ：解压缩归档文件。这个选项指示 `tar` 从归档文件中提取文件。
- **`-v`** ：显示详细的输出。在解压缩或打包过程中，它会列出处理的文件名。
- **`-f`** ：指定归档文件的名称。这个选项后面需要跟上归档文件的名称。

假设你有一个压缩的 tar 文件 `archive.tar.gz`，你可以使用以下命令来解压它：

```bash
tar -zxvf archive.tar.gz
```

---

# 5.切换为 root 用户

## 5.1 使用 `su` 命令

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

## 5.2 使用 `sudo` 命令

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

## 5.3 使用 `sudo su` 命令（常用）

你也可以通过 `sudo` 命令切换到 `root` 用户，使用以下命令：

```bash
sudo su -
```

- **`sudo su -`** 组合命令先用 `sudo` 获得临时的 `root` 权限，再用 `su -` 切换到 `root` 用户。

## 5.4 注意事项

- **`root` 密码** ：你需要知道 `root` 用户的密码来使用 `su` 命令。`sudo` 需要你当前用户在 `sudoers` 文件中有权限执行 `sudo` 命令，且你需要输入你自己的用户密码。
- **权限管理** ：在某些系统（如 Ubuntu）默认不启用 `root` 用户的密码，而是通过 `sudo` 提供管理员权限。你可能需要使用 `sudo` 命令而不是 `su` 来获得 `root` 权限。

---

# 6.查看文件大小

## 6.1 使用 `du` 命令

`du`（disk usage）命令用于显示文件和目录的磁盘使用情况：

```bash
du -sh /path/to/directory
```

**参数解释：**
- **`-s`** ：显示总计，只显示指定目录的总大小，而不是递归显示每个子目录。
- **`-h`** ：以人类可读的格式显示大小（如 KB, MB, GB），使输出更易读。

## 6.2 使用 `du` 命令按大小排序

如果你想按大小排序显示子目录，可以使用以下命令：

```bash
du -h /path/to/directory | sort -h
```

## 6.3 使用 `ls` 命令查看目录大小

尽管 `ls` 命令主要用于列出目录中的文件，它也可以用来查看目录的大小，但它的显示可能不如 `du` 直观。

```bash
ls -lh /path/to/directory
```

- **`-l`**：长格式列出文件信息，包括大小。
- **`-h`**：以人类可读的格式显示文件大小。

---

# 7.查看文件详细信息

1. **使用 `stat` 命令：**
```bash
stat filename
```
这会显示文件的详细信息，包括创建时间（如果文件系统支持），通常在“Birth”字段下。如果你的文件系统不支持创建时间，它可能不会显示这个字段。

2. **使用 `debugfs` 工具（仅适用于 ext 文件系统）：**
```bash
sudo debugfs -R 'stat <inode>' /dev/sdXn
```
替换 `<inode>` 为实际的 inode 号，`/dev/sdXn` 为文件所在的分区。这种方法可以提供更详细的文件信息。

3. **文件系统支持：**
一些文件系统（如 ext4）支持创建时间，而其他文件系统（如 ext3）则可能不支持。如果你在 `stat` 输出中看不到创建时间，可能是因为你的文件系统不支持这一功能。

---

# 8.提高编译速度的方法

## 8.1 增加编译并行度

可以通过设置更多的并行编译任务来加速编译过程。使用与 CPU 核心数量相等或更多的并行任务数。设置 MAKEFLAGS 来实现这一点：

编辑 /etc/makepkg.conf 文件，找到以下行并进行修改：
```
MAKEFLAGS="-j$(nproc)"
```
`$(nproc)`会自动检测 CPU 核心数，并设置相同数量的并行任务。包括在执行`make`指令时，可以通过加`-j<核心数>`来手动指定编译时用CPU的核心数。

---

# 9.用户操作

## 9.1 创建一个新用户

```bash
sudo useradd -m <new_username>
```
`-m`：创建用户的同时生成一个主目录。

为新用户设置密码：
```bash
sudo passwd <new_username>
```

如果希望新用户拥有管理员权限，可以将其加入`sudo`组：
```bash
sudo usermod -aG sudo <new_username>
```
Arch 系的发行版是用下面这种方式：
```bash
sudo usermod -aG wheel <new_username>
```

## 9.2 移除用户

删除用户及其主目录：
```bash
sudo userdel -r <username>
```
`-r`：删除用户的同时移除用户的主目录及文件。如果不使用`-r`，只会删除用户账号，不会删除用户的主目录和文件。

---

# 10.grep 指令

`grep` 是一个强大的文本搜索工具，它在 Unix 和类 Unix 系统（如 Linux）中用于搜索文件中匹配指定模式的行。`grep` 命令的名字来源于全局正则表达式打印（Global Regular Expression Print）。

## 10.1 基本用法

```bash
grep [选项] 模式 文件
```

- 模式：你想要搜索的文本模式或正则表达式。
- 文件：你想要搜索的文件。

## 10.2 常用选项

- `-i`：忽略大小写。
- `-v`：显示不匹配的行（即显示除了匹配行之外的所有行）。
- `-c`：显示匹配行的数量，不显示具体内容。
- `-n`：显示匹配行及行号。
- `-l`：显示包含匹配行的文件名。
- `--color`：将匹配的文本高亮显示。
- `-r` 或 `-R`：递归搜索指定目录下的所有文件。
- `-e`：允许多个搜索模式。

## 10.3 示例

1. **搜索包含特定文本的行：**
```bash
grep "search_text" filename.txt
```

2. **忽略大小写搜索：**
```bash
grep -i "SearchText" filename.txt
```

3. **显示不匹配的行：**
```bash
grep -v "text_to_exclude" filename.txt
```

4. **显示匹配行的行号：**
```bash
grep -n "search_text" filename.txt
```

5. **递归搜索目录下所有文件：**
```bash
grep -r "search_text" /path/to/directory
```

6. **显示匹配行的数量：**
```bash
grep -c "search_text" filename.txt
```

7. **使用正则表达式搜索：**
```bash
grep "^[0-9]" filename.txt
```
这个命令会搜索所有以数字开头的行。

8. **使用多个模式搜索：**
```bash
grep -e "pattern1" -e "pattern2" filename.txt
```