---
title: Git教程
date: 2024-07-19 03:36:51
tags:
   - "linux"
   - "git"
excerpt: "记录自己的 git 使用过程。"
---


# 一、 SSH 连接

需要提前安装好`OpenSSH`:
```bash
sudo pacman -S openssh
```

## 1.1 生成 SSH 密钥

如果还没有 SSH 密钥，可以生成一个：

```bash
# 推荐
ssh-keygen -t ed25519 -C "your_email@example.com"
```
或者
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

按照提示完成生成过程，默认会在 `~/.ssh/` 目录下生成 `id_ed25519`（不能告诉别人）和`id_ed25519.pub`（公有的） 文件。第二种生成密钥的方式对应的文件是`id_rsa` 和 `id_rsa.pub`。

## 1.2 添加 SSH 密钥到 GitHub

1. 将生成的公钥添加到 GitHub。在终端中运行：

```bash
cat ~/.ssh/id_ed25519.pub
```

输出示例：
```
ssh-ed25519 AAAAC3NzANDAKlZDI1NTE5AAAaaaN/oaWb4GZetVkiI5/8d1gezLHWpT46mnaAnkJAbwXpp loskyertt0403@gmail.com
```

2. 复制输出的公钥内容（从 ssh-ed25519 开始，直到邮箱地址结束）。（或者打开`sh/id_rsa.pub` 文件，复制其中的内容。）
3. 登录到 GitHub，导航到“Settings”。
4. 在左侧菜单中选择“[SSH and GPG keys 设置页面](https://github.com/settings/keys)”。
5. 点击“New SSH key”，粘贴你的公钥并保存。

## 1.3 连接验证

用于检查是否能与远程的 GitHub 仓库进行连接。
```bash
ssh -T git@github.com
```
加上`-v`参数可以显示更详细的连接信息。

## 1.3 配置仓库使用 SSH URL

1. 更新远程仓库的 URL 以使用 SSH：

```bash
git remote set-url origin git@github.com:loskyertt/Recorder_Backup.git
```

## 1.4 无法连接

当使用`ssh -T git@github.com`迟迟未返回连接的信息，这种情况可能是`~/.ssh/config`配置文件发生损坏或者没有该配置文件。

修改或创建`config`配置文件：
```bash
nano ~/.ssh/config
```

添加下面内容：
```txt
Host github.com
  Hostname ssh.github.com
  Port 22
  User git
```
`22`端口如果不行，可以改成`443`端口。

# 二、个人访问令牌（Personal Access Token，PAT）

这种方式主要用于临时快速修改仓库，或者当 SSH 连接不可使用时使用。

1. **生成个人访问令牌：**
   - 登录到你的 GitHub 账户。
   - 进入 [GitHub 个人访问令牌设置页面](https://github.com/settings/tokens)。
   - 点击 `Generate new token` 按钮。
   - 选择 "Personal access tokens" 然后点击 "Generate new token"。
   - 给令牌一个描述性的名称，选择适当的权限范围（至少需要`repo`权限）。
   - 生成令牌并复制（注意： **令牌只会显示一次，请务必保存** ）。

2. **使用个人访问令牌进行认证：**
   - 当 Git 提示输入密码时，使用您刚刚创建的个人访问令牌而不是密码。

3.  **存储凭证（可选但推荐）：**
为了避免每次都输入令牌，您可以让 Git 记住你的凭证：
```bash
git config --global credential.helper store
```
然后，下一次当您输入令牌时，Git 会记住它。

4. **更新远程 URL（如果需要）：**
如果SSH持续出现问题,您可以暂时使用 HTTPS 方式进行操作。可以通过以下命令将仓库的远程URL改为HTTPS:
```
git remote set-url origin <https://remote-url>
```

一定要记住，个人访问令牌与密码具有相同的权限，因此要像对待密码一样谨慎地保护它。如果你认为令牌可能已经泄露，可以随时在 GitHub 设置中撤销它并生成一个新的。


# 三、git基本配置

## 3.1 设置用户信息

```bash
git config --global user.name "用户名"
git config --global user.email "邮箱"
```
## 3.2 代理设置

**注：`linux`只需要在终端配置代理即可。**

```bash
git config --global http.proxy socks5://127.0.0.1:"端口"
git config --global https.proxy socks5://127.0.0.1:"端口"
```

如果要取消代理：
```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```
