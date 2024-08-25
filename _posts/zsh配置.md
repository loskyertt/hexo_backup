---
title: 终端 zsh 配置指南
date: 2024-07-19 21:40:05
tags:
  - "linux"
excerpt: "Linux 下对 zsh 的相关配置，比如终端代理，终端样式等"
---


# 一、终端代理设置

建议加上，在进行通过终端的下载、更新系统、`conda`下载或者`git clone`时，能走代理来提高下载速度。但是`docker`需要单独配置一套代理。`zsh`和`bash`都可以用这种方式。

```bash
kate ~/.zshrc
```
`bash`需要在`.bashrc`中修改。

- **在`.zshrc`中添加以下内容：**

```txt
# where proxy
proxy(){
  export http_proxy="http://127.0.0.1:12334"
  export https_proxy="http://127.0.0.1:12334"
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

# 二、zsh配置

## 2.1 切换 zsh 为默认终端

要确保已经安装了`zsh`：
```bash
chsh -s $(which zsh)
```

通常要重启系统才会生效。

验证默认`shell`
```bash
echo $SHELL
```

## 2.2 样式配置（prompt/PS1）

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

### 2.2.1推荐配置：

- **主机1配置：**
```txt
# PROMPT
NEWLINE=$'\n' # 换行
PS1="%B%F{2}%D%f %F{3}%*%f%b [%F{184}%%n-M%f@%F{30}%~%f]${NEWLINE}%F{111}╰─❯%f"
```

- **主机2配置：**
```txt
# PROMPT
NEWLINE=$'\n' # 换行
PS1="%U%B%F{43}%D%f %F{30}%*%f%b%u [%F{184}%n-%M%f@%F{30}%~%f]${NEWLINE}%F{220}==>%f"
```

推荐两套主机用不同的配置，这样在用 SSH 进行远程操控时，方便辨别。

## 2.3 插件配置

- **备份（你面配置失败，最好备份下）：**
```bash
cp ~/.zshrc ~/.zshrc.backup
```

- **创建配置文件夹：**

```bash
mkdir -p .zsh/plugins
```

```bash
mv .zshrc .zsh/
```

```bash
mv .zsh_history .zsh/
```
注：若没有`.zsh_history`，那么需要用`touch`指令创建。
```bash
touch .zsh_history
```

- **然后编辑文件夹`.zsh`中的`.zshrc`，加上：**
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

进入安装插件的目录：
```bash
cd ~/.zsh/plugins
```
插件一：
```bash
git clone  https://github.com/zdharma-continuum/fast-syntax-highlighting.git
```
插件二：
```bash
git clone https://github.com/zsh-users/zsh-autosuggestions.git
```
插件三：
```bash
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
最后就是创建符号链接，这样我们就可以通过更改`~/.zshrc`Copy来同步更改`.zsh/.zshrc`Copy配置文件了。首先需要确认`~`目录下没有`.zshrc`文件，如果有，就`rm .zshrc`。

此时可以开始创建符号链接了：
```bash
ln -s ~/.zsh/.zshrc ~/.zshrc
```

重新加载配置文件：
```bash
source ~/.zshrc
```
可以通过`ls -la`来查看是否链接成功。

# 三、问题汇总

## 3.1 历史命令问题
出现这种情况`zsh: corrupt history file /home/sky/.zsh/.zsh_history`。有的时候系统因为默写原因强行启动的时候会破坏zsh的历史文件。
解决办法：
```bash
cp ~/.zsh_history ~/.zsh_history_backup
rm ~/.zsh_history
strings -eS ~/.zsh_history_backup > ~/.zsh_history
fc -R ~/.zsh_history
```
如果上述步骤没有解决问题，可能是因为.zsh_history文件严重损坏。在这种情况下，需要放弃旧的历史记录并创建一个新的文件。

## 3.2 安装 oh my zsh 可能会出现的问题

**注意：** 如果是安装`oh my zsh`可能会出现下面的问题：

1.发现安装完`oh my zsh`后终端中有些命令不能使用：
编辑`.zshrc`发现里面内容都被替换掉了，之前的配置内容都被转移到一个叫`.zshrc.pre-oh-my-zsh`文件中。