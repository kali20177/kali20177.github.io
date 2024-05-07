# Manim


# 安装

## MacOS

在 MacOS 上推荐创建虚拟环境后再使用 pip 安装依赖包。直接使用 pip3 安装会导致如下错误：

```zsh
error: externally-managed-environment

× This environment is externally managed
╰─> To install Python packages system-wide, try brew install
    xyz, where xyz is the package you are trying to
    install.

    If you wish to install a non-brew-packaged Python package,
    create a virtual environment using python3 -m venv path/to/venv.
    Then use path/to/venv/bin/python and path/to/venv/bin/pip.

    If you wish to install a non-brew packaged Python application,
    it may be easiest to use pipx install xyz, which will manage a
    virtual environment for you. Make sure you have pipx installed.
```

创建虚拟环境再处理：

```zsh
python3 -m venv /Users/kali/pyvirtual
/Users/kali/pyvirtual/bin/pip install manim
```

一般会提示关于 No package 'pangocairo' found 的相关错误。解决方法：

```zsh
brew install cairo 
brew install pango
```

> 类似的，Arch Linux 上安装同样会出现上述错误：输入 `sudo pacman -S cairo pango` 解决。

然后会提示 Could not build wheels for Pillow 的相关错误，解决方式：

```zsh
brew install libtiff libjpeg webp little-cms2
```

接下来一般能构建成功。

或者使用 pipx：

```zsh
brew install pipx 
pipx install manim --include-deps
```

