---
title: Git教程
date: 2024-07-19 03:36:51
tags:
   - "linux"
   - "git"
excerpt: "记录自己的 git 使用过程。"
---


# 一、 SSH 密钥

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

## 1.3 配置仓库使用 SSH URL

1. 更新远程仓库的 URL 以使用 SSH：

```bash
git remote set-url origin git@github.com:loskyertt/Recorder_Backup.git
```

## 1.4 推送更改

现在可以使用 SSH 进行推送：

```bash
git push origin main
```


# 二、个人访问令牌（Personal Access Token，PAT）

1. **生成个人访问令牌**：
   - 登录到你的 GitHub 账户。
   - 进入 [GitHub 个人访问令牌设置页面](https://github.com/settings/tokens)。
   - 点击 `Generate new token` 按钮。
   - 在令牌的描述中输入一个名称。
   - 选择所需的权限（例如 `repo`）。
   - 生成令牌并复制。

2. **使用个人访问令牌进行认证**：
   - 在你运行 `git push` 时，会提示你输入用户名和密码。在提示用户名时，输入你的 GitHub 用户名。在提示密码时，输入刚刚生成的个人访问令牌。

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
