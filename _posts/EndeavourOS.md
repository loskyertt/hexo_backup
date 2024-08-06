---
title: EndeavourOS 使用记录
date: 2024-07-18 16:21:15
tags:
  - "linux"
  - "conda"
  - "docker"
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


# 二、终端代理设置

建议加上，在进行通过终端的下载、更新系统、`conda`下载或者`git clone`时，能走代理来提高下载速度。但是`docker`需要单独配置一套代理。

```bash
kate ~/.zshrc
```

- **在`.zshrc`中添加以下内容：**

```txt
# where proxy
proxy(){
  export http_proxy="http://127.0.0.1:2334"
  export https_proxy="http://127.0.0.1:2334"
  echo "HTTP Proxy on by hiddify"
}

# where noproxy
noproxy(){
  unset http_proxy
  unset https_proxy
  echo "HTTP Proxy off"
}
```
根据实际情况填写自己的代理端口。

```bash
source ~/.zshrc
```

通过在终端输入`proxy`或者`noproxy`来开启或关闭代理。

输入下面指令，可以查看终端代理地址：
```bash
env | grep -i proxy
```


# 三、zsh配置

## 3.1 样式配置（prompt/PS1）

  | Code   | Info                              |
  | ------ |:---------------------------------:|
  | %T     | 系统时间（时：分）                         |
  | %*     | 系统时间（时：分：秒）                       |
  | %D     | 系统日期（年-月-日）                       |
  | %n     | 用户名称（即：当前登陆终端的用户的名称，和whami命令输出相同） |
  | %B     | 开始到结束使用粗体打印                       |
  | %b     | 开始到结束使用粗体打印                       |
  | %U     | 开始到结束使用下划线打印                      |
  | %u     | 开始到结束使用下划线打印                      |
  | %d     | 你当前的工作目录                          |
  | %~     | 你目前的目录相对于～的相对路径                   |
  | %M     | 计算机的主机名                           |
  | %m     | 计算机的主机名（在第一个句号之前截断）               |
  | %l     | 你当前的tty                           |
  | %F{色码} | 用来设定某个颜色的开始                       |
  | %f     | 用来设定成预设的样式， 也可以说是设定好的颜色结束         |

- **换行示例：**

```
NEWLINE=$'\n'
PROMPT="%{$fg[green]%}%d %t %{$reset_color%}%# ${NEWLINE}"
```

- **显示git分支：**
```
parse_git_branch() {
  git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
}
```

### 3.1.1推荐配置：

- **配置1：**
```
# PROMPT
parse_git_branch() {
  git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
}
NEWLINE=$'\n' # 换行
PS1="%B%F{2}%D%f %F{60}%*%f%b [%F{184}%n%f@%F{30}%~%f]$(parse_git_branch)${NEWLINE}%F{111}$%f "
```

- **配置2：**
```txt
# PROMPT
NEWLINE=$'\n' # 换行
PS1="%B%F{2}%D%f %F{60}%*%f%b [%F{184}%n%f@%F{30}%~%f]${NEWLINE}%F{111}╰─❯%f"
```

- **配置3：**
```txt
# PROMPT
ZSH_NEWLINE=$'\n'
export PROMPT=" %F{46}%F %(?.%F{green}√.%F{red}?%?)%f  %B%F{69}%~ ${ZSH_NEWLINE} %F{119}==>%f%b "
```

## 3.2 插件配置

- **备份：**
```bash
cp ~/.zshrc ~/.zshrc.backup
```

- **创建配置文件夹：**
```bash
mkdir -p .zsh/plugins
cp .zshrc .zsh/
mv .zsh_history .zsh/
```
注：若没有`.zsh_history`，那么需要用`touch`指令创建。

- **然后编辑文件夹`.zsh`中的`.zshrc`copy，加上：**
```txt
### ZSH HOME
export ZSH=$HOME/.zsh

### ---- history config ----------
export HISTFILE=$ZSH/.zsh_history

# How many commands zsh will load to memory.
export HISTSIZE=10000

# How maney commands history will save on file.
export SAVEHIST=10000

# History won't save duplicates.
setopt HIST_IGNORE_ALL_DUPS

# History won't show duplicates on search.
setopt HIST_FIND_NO_DUPS
```

- **安装插件：**
```bash
cd ~/.zsh/plugins

git clone  https://github.com/zdharma-continuum/fast-syntax-highlighting.git

git clone https://github.com/zsh-users/zsh-autosuggestions.git

git clone https://github.com/zsh-users/zsh-completions.git
```

- **在`.zshrc`copy中添加：**
```
source $ZSH/plugins/fast-syntax-highlighting/fast-syntax-highlighting.plugin.zsh
fpath=($ZSH/plugins/zsh-completions/src $fpath)

# zsh-autosuggestions:config
source $ZSH/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE="fg=#ff00ff,bg=cyan,bold,underline"
ZSH_AUTOSUGGEST_STRATEGY=(history completion)
ZSH_AUTOSUGGEST_BUFFER_MAX_SIZE=20

# end config
```

- **链接：**
最后就是创建符号链接，这样我们就可以通过更改`~/.zshrc`Copy来同步更改`.zsh/.zshrc`Copy配置文件了。首先需要确认`~`目录下没有`.zshrc`文件，如果有，就`rm .zshrc`。此时可以开始创建符号链接了.
```bash
ln -s ~/.zsh/.zshrc ~/.zshrc

source ~/.zshrc
```

可以通过`ls -la`来查看是否链接成功。

## 3.3 完整的配置内容

```txt
###-----------------zsh config--------------------
### ZSH HOME
export ZSH=$HOME/.zsh

### ---- history config ----------
export HISTFILE=$ZSH/.zsh_history

# How many commands zsh will load to memory.
export HISTSIZE=10000

# How maney commands history will save on file.
export SAVEHIST=10000

# History won't save duplicates.
setopt HIST_IGNORE_ALL_DUPS

# History won't show duplicates on search.
setopt HIST_FIND_NO_DUPS

source $ZSH/plugins/fast-syntax-highlighting/fast-syntax-highlighting.plugin.zsh
fpath=($ZSH/plugins/zsh-completions/src $fpath)

# zsh-autosuggestions:config
source $ZSH/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE="fg=#ff00ff,bg=cyan,bold,underline"
ZSH_AUTOSUGGEST_STRATEGY=(history completion)
ZSH_AUTOSUGGEST_BUFFER_MAX_SIZE=20

# end config
```

## 3.4 问题汇总

### 3.4.1 历史命令问题
出现这种情况`zsh: corrupt history file /home/sky/.zsh/.zsh_history`。有的时候系统因为默写原因强行启动的时候会破坏zsh的历史文件。
解决办法：
```bash
cp ~/.zsh_history ~/.zsh_history_backup
rm ~/.zsh_history
strings -eS ~/.zsh_history_backup > ~/.zsh_history
fc -R ~/.zsh_history
```
如果上述步骤没有解决问题，可能是因为.zsh_history文件严重损坏。在这种情况下，需要放弃旧的历史记录并创建一个新的文件。

### 3.4.2 安装 oh my zsh 可能会出现的问题

**注意：** 如果是安装`oh my zsh`可能会出现下面的问题：

1.发现安装完`oh my zsh`后终端中有些命令不能使用：
编辑`.zshrc`发现里面内容都被替换掉了，之前的配置内容都被转移到一个叫`.zshrc.pre-oh-my-zsh`文件中。


# 四、中文输入法配置

## 4.1 推荐方式

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

## 4.2 其它（没试过）

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

# 五、其它问题汇总

## 5.1 证书安装问题
有时候可能无法安装`ca-certificates`这一系列证书，排除掉网络问题后，大概率就是时间问题。解决办法：

1. **同步系统时间**：
运行以下命令以确保系统时间是准确的：
```bash
sudo ntpd -qg
sudo hwclock --systohc
```

2. **手动更新CA证书**：
- 将信任的CA证书复制到`/etc/pki/ca-trust/source/anchors/`。确保证书是PEM或DER格式。
- 然后，运行以下命令来更新系统中的CA证书：
```bash
sudo update-ca-trust
```
------------一般到这里就能解决了-------------

1. **重新安装软件包**：
可以尝试重新安装`ca-certificates`和`ca-certificates-utils`：
```bash
sudo pacman -S ca-certificates ca-certificates-utils
```

## 5.2 镜像源问题

如果下载或者更新速度较慢，可以用 Arch 清华镜像源：
```bash
kate /etc/pacman.d/mirrorlist
```
然后在首行加上：
```txt
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
```

## 5.3 签名验证问题

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

# 六、miniconda3 配置

**注：** 如果用的是`miniforge`，操作方式与此类似。

## 6.1 把`conda`集成到`zsh`终端中

### 6.1.1 详细步骤

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

### 6.1.2 验证

验证 conda 是否配置好：

```bash
conda --version
```

### 6.1.3 一步到位操作

可以不用管以上操作，直接把这段复制到`.zshrc`中，注意安装miniconda的路径：

```bash
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

## 6.2 问题汇总

### 6.2.1  OpenSSL 问题

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

# 七、docker 配置

## 7.1 安装 docker

- **安装方式一：**

通过`yay`直接安装（全局安装）：

```bash
yay -Sy docker docker-compose
```
`docker-compose`可选安装，它可以通过配置文件`Dockerfile`来构建镜像。

- **安装方式二：**

通过`conda`安装（局部安装，可全局使用）：

首先需要创建虚拟环境用于存储安装的`docker`和`docker-compose`。

```bash
conda create --name docker python=3.12
```
Python版本可自己选择。

```bash
conda install docker docker-compose -c conda-forge
```

这里的`-c conda-forge`一定要加，默认的`conda.org`源没有`docker`。

安装好的`docker`和`docker-compose`是在该路径下：
`~/.conda/envs/docker/bin/docker`
`~/.conda/envs/docker/bin/docker-compose`

## 7.2 docker 基本操作

### 7.2.1 docker 服务

```sh
# 启动docker服务
sudo systemctl start docker

# 关闭docker服务
sudo systemctl stop docker.service docker.socket
```

检查`docker`服务状态：

```sh
sudo systemctl status docker
```

开机自启动设置：

```sh
# 设置开机自启动
sudo systemctl enable docker
sudo systemctl enable docker.socket

# 关闭开机自启动
sudo systemctl disable docker
sudo systemctl disable docker.socket
```

### 7.2.2 添加用户到 docker 组

为了避免每次运行`docker`命令时都需要使用 `sudo`，可以将当前用户添加到 `docker` 组：

```sh
sudo usermod -aG docker $USER
```

然后，重新登录以使更改生效，或者重新加载用户组信息：

```sh
# 登录
newgrp docker

# 退出
exit
```

## 7.3 代理配置（推荐）

1.创建`docker`相关的`systemd`目录，这个目录下的配置将覆盖`docker`的默认配置：

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
```

2.新建配置文件：

```bash
kate /etc/systemd/system/docker.service.d/proxy.conf
```

将以下内容复制到`proxy.conf`中：

```txt
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:2334"
Environment="HTTPS_PROXY=http://127.0.0.1:2334"
```

这里需要根据自己实际的代理端口填写。

3.如果自己建了私有的镜像仓库，需要`docker`绕过代理服务器直连，那么配置`NO_PROXY`变量：

```
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80"
Environment="HTTPS_PROXY=https://proxy.example.com:443"
Environment="NO_PROXY=your-registry.com,10.10.10.10,*.example.com"
```

多个`NO_PROXY`变量的值用逗号分隔，而且可以使用通配符（*），极端情况下，如果`NO_PROXY=*`，那么所有请求都将不通过代理服务器。

4.重新加载配置文件，重启`docker`：

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

5.检查确认环境变量已经正确配置：

```bash
sudo systemctl show --property=Environment docker
```

像输出以下内容就成功了：

```
Environment=HTTP_PROXY=http://127.0.0.1:2334 HTTPS_PROXY=http://127.0.0.1:2334
```

## 7.4 命令简化配置

查看容器：
```bash
# 查看运行中的容器
docker ps

# 查看所用容器
docker ps -a

# 格式化查看容器
docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}\t{{.Names}}"
   ```

命令简化（推荐加到.bashrc或者.zshrc中）：
```txt
alias dps='docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}\t{{.Names}}"'
```
