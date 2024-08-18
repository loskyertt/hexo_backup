---
title: SSH 教程
date: 2024-08-16 17:04:17
tags:
  - "linux"
  - “ssh"
excerpt: "主要记录平时用到的`ssh`指令以及使用方法。"
---


# 1.Windows 安装 SSH 工具

## 1.1 方式一、使用 OpenSSH

可以直接搭配 Windows 的 PowerShell 使用，很轻量，但是功能少。配置方式可以参考我写的这篇文章：[Win10 SSH 服务配置](https://loskyertt.github.io/2024/07/30/Openssh%E9%85%8D%E7%BD%AE/)

## 1.2 方式二、使用  MobaXterm （推荐）

本人用了下，效果还不错，就是没有中文。下载地址：[MobaXterm](https://mobaxterm.mobatek.net/download.html)

## 1.3 方式三、使用 WindTerm （开源）

支持中文，在 GitHub 开源。下载地址：[WindTerm](https://github.com/kingToolbox/WindTerm)

## 1.4 方式三、其他

Linux 下直接下载 OpenSSH 就行了，使用`zsh`或者`bash`终端。 

其他 SSH 连接工具推荐：[视频地址]( https://www.bilibili.com/video/BV1xJ4m1s7zX/?share_source=copy_web&vd_source=9fce48d1dc82a922216accb794549993)

# 2.远程主机的配置

## 2.1 安装 OpenSSH 

Linux 一般默认安装上的，如果没有，可以手动安装，这里以 Arch 系 Linux 为例：
```bash
sudo pacman -S openssh
```

## 2.2 开启 SSH 服务

- 让 sshd 开机自启动：
```bash
sudo systemctl enable sshd
```

- 开启 sshd 服务：
```bash
sudo systemctl start sshd
```

- 查看是否启动成功：
```bash
sudo systemctl status sshd
```

## 2.3 配置防火墙（一定要配置）

- `ufw` 防火墙：
```bash
sudo ufw allow ssh
```

- `iptables`防火墙：
```bash
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

- `firewalld`防火墙：
显示`firewalld`的当前防火墙配置状态：
```bash
sudo firewall-cmd --list-all
```

通常会显示以下信息：
- **`interfaces`** : 显示与当前区域关联的网络接口。
- **`sources`** : 显示与当前区域关联的 IP 源地址或子网。
- **`services`** : 显示允许的服务列表（例如 `ssh`, `http`, `https` 等）。
- **`ports`** : 显示手动添加的开放端口及其协议（例如 `22/tcp`, `80/tcp` 等）。
- **`protocols`** : 显示允许的协议（通常包括 `ipv4` 和 `ipv6`）。
- **`zone`** : 当前配置的区域名称。
- **`forward`** : 显示转发规则（如果有的话）。
- **`masquerade`** : 显示是否启用网络地址转换（NAT）。

确保`22`端口被开放：
```bash
sudo firewall-cmd --add-service=ssh --permanent
sudo firewall-cmd --reload
```

也可以检查 SSH 配置文件：
```bash
sudo nano /etc/ssh/sshd_config
```

## 2.4 查看日志

如果你遇到任何问题，可以查看 SSH 服务的日志以获取更多信息：
```bash
sudo journalctl -u sshd
```

# 3.常用指令

## 3.1 与远程主机进行连接

```bash
ssh <用户名>@<远程主机 IP 地址>
```

## 3.2 检查与远程代码仓库的连接

这里以 GitHub 为例：
```bash
ssh -T git@github.com
```

显示以下内容表示连接成功：
`Hi loskyertt! You've successfully authenticated, but GitHub does not provide shell access.`

## 3.3 传输文件

使用 `scp` 命令可以通过 SSH 传输文件，`scp` 是 Secure Copy Protocol 的缩写。

**示例：**

1. **从本地计算机复制文件到远程计算机** ：
```bash
scp localfile.txt username@remote_host:/path/to/remote/directory/
```
这里 `localfile.txt` 是要复制的本地文件，`username` 是远程计算机上的用户名，`remote_host` 是远程计算机的 IP 地址或主机名，`/path/to/remote/directory/` 是远程计算机上的目标目录。

2. **从远程计算机复制文件到本地计算机** ：
```bash
scp username@remote_host:/path/to/remote/file.txt /path/to/local/directory/
```

3. **复制目录及其内容** ：
使用 `-r` 选项可以递归地复制目录及其内容。
```bash
scp -r local_directory username@remote_host:/path/to/remote/directory/
```

# 4.问题汇总

## 4.1 远程主机密钥更改

### 4.1.1 问题示例

```bsh
24-08-14 16:10:00 [sky@~]
╰─❯ssh root@124.15.31.104
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ED25519 key sent by the remote host is
SHA256:hQeGc/7954FRAh/w8RFXKUNFz6xxH9DnYRdygBIfWiw.
Please contact your system administrator.
Add correct host key in /home/sky/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /home/sky/.ssh/known_hosts:6
Host key for 124.15.31.104 has changed and you have requested strict checking.
Host key verification failed.
```

出现这种警告表明你尝试连接的远程主机的 SSH 密钥已经发生了变化。这种情况可能有几种原因：

1. **主机密钥确实更改了**：远程服务器可能因为重新安装了操作系统、重新生成了密钥对或者更改了 SSH 配置而更换了主机密钥。

2. **可能的安全风险**：如果你没有预期到主机密钥的变化，可能有人试图进行中间人攻击（Man-in-the-Middle Attack），这意味着有人可能在试图窃听你的连接。

### 4.1.2 解决步骤

1. 确认更改的原因

如果你对主机密钥更改有充分的理由（例如，你知道服务器进行了重新安装或者密钥更新），你可以安全地更新你的本地 `known_hosts` 文件。

2. 删除旧的密钥

你可以手动编辑 `~/.ssh/known_hosts` 文件，删除对应的旧密钥。

打开 `known_hosts` 文件：

```bash
nano ~/.ssh/known_hosts
```

查找并删除第6行（或者显示的“Offending ECDSA key”所在的行），这行对应于你连接的服务器的旧密钥。

保存文件并退出编辑器。

如果你更喜欢使用命令行工具，你可以使用以下命令删除该行：

```bash
sed -i '6d' ~/.ssh/known_hosts
```

注意：此命令会删除 `known_hosts` 文件中的第6行。

3. 再次连接服务器

删除旧密钥后，重新尝试连接服务器：

```bash
ssh root@124.15.31.104
```

你将会看到提示要求确认新的主机密钥。确保你确认这个新的密钥是合法的。如果确定是合法的，可以输入 `yes` 来接受新的密钥并将其添加到 `known_hosts` 文件中。
