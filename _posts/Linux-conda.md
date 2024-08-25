---
title: Linux 下配置 conda 环境
date: 2024-08-19 16:21:15
tags:
  - "linux"
  - "conda"
excerpt: "这是一篇Linux（EndeavourOS）下如何把 conda 环境集成到终端中的教程。"
---


# 一、miniconda3 配置

**注：** 如果用的是`miniforge`，操作方式与此类似。

## 1.1 把`conda`集成到`zsh`终端中

### 1.1.1 详细步骤

1. **添加`conda`环境变量：**

把这行代码加入到`.zshrc`中：

```txt
export PATH=/opt/miniconda3/bin:$PATH
```

终端输入这行可以防止conda自动激活环境（建议加上）：

```bash
conda config --set auto_activate_base false
```

2. **重新加载 `~/.zshrc` 文件：**

在终端中运行以下命令重新加载配置文件：

```sh
source ~/.zshrc
```

3. **运行 `conda init`：**

初始化 conda 环境：

```bash
# 这是conda默认安装的位置
/opt/miniconda3/bin/conda init zsh
```

4. **再次重新加载 `~/.zshrc` 文件：**

再次运行以下命令重新加载配置文件：

```sh
source ~/.zshrc
```

### 1.1.2 验证

验证 conda 是否配置好：

```bash
conda --version
```

### 1.1.3 一步到位操作

可以不用管以上操作，直接把这段复制到`.zshrc`中，注意安装miniconda的路径：

```txt
# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$('/opt/miniconda3/bin/conda' 'shell.zsh' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/opt/miniconda3/etc/profile.d/conda.sh" ]; then
        . "/opt/miniconda3/etc/profile.d/conda.sh"
    else
        export PATH="/opt/miniconda3/bin:$PATH"
    fi
fi
unset __conda_setup
# <<< conda initialize <<<
export CRYPTOGRAPHY_OPENSSL_NO_LEGACY=1
```

## 1.2 问题汇总

### 1.2.1  OpenSSL 问题

当运行`conda activate base`时，可能出现下面问题：

```txt
 $ conda activate base
Error while loading conda entry point: conda-content-trust (OpenSSL 3.0's legacy provider failed to load. This is a fatal error by default, but cryptography supports running without legacy algorithms by setting the environment variable CRYPTOGRAPHY_OPENSSL_NO_LEGACY. If you did not expect this error, you have likely made a mistake with your OpenSSL configuration.)

CondaError: Run 'conda init' before 'conda activate'
```

根据该的错误信息，问题可能与 OpenSSL 版本有关。OpenSSL 3.0 引入了一些变化，可能导致与某些软件包的兼容性问题。在此情况下，设置环境变量`CRYPTOGRAPHY_OPENSSL_NO_LEGACY`可能会解决问题。

- **解决方法：**

编辑 `~/.zshrc` 文件，添加以下行到 `~/.zshrc` 文件：

```txt
export CRYPTOGRAPHY_OPENSSL_NO_LEGACY=1
```

保存并退出编辑器，然后执行`source ~/.zshrc`。
