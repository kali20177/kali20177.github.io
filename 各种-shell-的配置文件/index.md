# 各种 Shell 的配置文件


## 1. PowerShell 7

微软的 powershell 更新到 7 之后的版本终于让终端比较流畅，并且支持**自动补全**历史命令记录。安装后使用一下命令产生配置文件：

```powershell
echo $PROFILE
```

一般会生成在 `C:\Users\{username}\Documents\PowerShell\Microsoft.PowerShell_profile.ps1` 下，通过编辑此文件配置终端命令。例如设置 function，作用类似于 linux 下的 alias。

语法：function 别名 { 被替代的命令 }

```powershell
function s {fastfetch}
function c {clear}
function q {exit}
```

如果被替代的命令在环境变量中就无需填写绝对路径。

## 2. Zsh / Bash

Linux 终端的 shell 主流是 bash 和 zsh，二者配置文件编辑方式基本相同。推荐使用 zsh 和 oh-my-zsh。

安装 zsh 和 oh-my-zsh 后，配置 `~/.zshrc`配置命令。需要安装命令行[语法高亮](https://github.com/zdharma-continuum/fast-syntax-highlighting)、[补全](https://github.com/zsh-users/zsh-autosuggestions)和跳转工具 (autojump)

```bash
git clone https://github.com/zdharma-continuum/fast-syntax-highlighting.git \
  ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/fast-syntax-highlighting

git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

```shell
...
plugins=(git
	history
    history-substring-search
	zsh-autosuggestions
	fast-syntax-highlighting
	autojump
)
...
alias c='clear'
alias ra='ranger'
alias s='fastfetch'
alias q='exit
alias ls='lsd'
alias ll='lsd -l'
alias vim='nvim'

# python 添加 pip3 安装可执行文件位置
export PATH=/home/kali/.local/bin/:$PATH

# fcitx5
export XMODIFIERS=@im=fcitx
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx

# rust
export RUSTUP_DIST_SERVER="https://rsproxy.cn"
export RUSTUP_UPDATE_ROOT="https://rsproxy.cn/rustup"
export PATH=$PATH:"$HOME/.cargo/env"

# go 
export GO111MODULE=on
export GOPROXY=https://goproxy.cn
```

## 参考

1. [在 Windows 终端中使用别名 alias 的方法](http://z6b.cn/K69a4)
2. [在 Ubuntu 安装配置 Fcitx 5 中文输入法](https://muzing.top/posts/3fc249cf/)
