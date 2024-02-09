---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: "2018-07-23T12:50:27+08:00"
lastmod: "2024-02-09T13:54:57+08:00"
showToc: true
tags: [Mac, iTerm]
title: Mac终端(Terminal)配置
---

## Mac 终端(Terminal)配置

平时工作中命令行用的比图形界面多，所以有必要配置一个赏心悦目的终端界面来提高工作效率(^\_^)。

![](../images/MacScreenShot-2018-07-20-5-35-35.png)

### iTerm

第一步就是替换原来的自带终端(Terminal)，换成[iTerm](https://www.iterm2.com/)。iTerm 是一个深受广大开发者欢迎的终端 App，代码托管在[Github](https://github.com/gnachman/iTerm2)，可以直接在官网下载安装。最新版为 `Build 3.4.23`

打开*iTerm2 > Preferences > General*，在`Selection`下勾上`Applications in terminal may access clipboard`使在`iTerm`中鼠标选中就能复制到系统剪切板使用`command+v`粘贴

打开*iTerm2 > Preferences > Profiles*，右边点`Keys`把左右 option 键设为`Esc+`，取消勾选`Apps can change this`来启用 Unix 的`Alt + B`和`Alt + F`前进和后退一个单词。

![](../images/ScreenShot-2018-07-20-09-34-18.png)

打开*iTerm2 > Preferences > Terminal*，底部 `Shell Integration` 取消勾选 `Show mark indicators` 不然每次执行命令会出现一个小箭头影响美观
### Zsh & Oh My Zsh

打开 iTerm 安装[Homebrew](https://brew.sh/)

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

使用 Homebrew 安装 zsh，设为默认的终端。从 Big Sur 已经设置 zsh 为默认的终端了，可以跳过此步骤，可以使用`echo $SHELL`检查

```shell
brew install zsh
```

[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)是 zsh 的配置文件，使用下面命令安装

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

打开 iTerm 会耐看很多，应该长成这个样子了

![](../images/MacScreenShot-2018-07-20-8-13.png)

配置`.zshrc`文件可以更改主题或者增加插件，默认启用 robbyrussell 主题和开启了 git 插件，可以根据需要更改[主题](https://github.com/robbyrussell/oh-my-zsh/wiki/Themes)和[插件](https://github.com/robbyrussell/oh-my-zsh/wiki/Plugins)。

```shell
SH_THEME="robbyrussell"
...
plugins=(
  git
)
```

然后加第三方插件

- [_zsh-autosuggestions_](https://github.com/zsh-users/zsh-autosuggestions) 自动提示插件，根据你输入的历史命令给出提示

```shell
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

- [_zsh-syntax-highlighting_](https://github.com/zsh-users/zsh-syntax-highlighting) 高亮不同的命令插件

```shell
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

  - [_zsh-history-substring-search_](https://github.com/zsh-users/zsh-history-substring-search) 高亮查找匹配前缀的历史输入

```shell
git clone https://github.com/zsh-users/zsh-history-substring-search ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-history-substring-search
```

配置 `ctrl+p` 和 `ctrl+n` 触发，在 `.zshrc` 增加如下配置

```shell
bindkey -M emacs '^P' history-substring-search-up
bindkey -M emacs '^N' history-substring-search-down
```

最后加入到`plugins`启用

```shell
plugins=(
  git
  zsh-autosuggestions
  zsh-syntax-highlighting
  zsh-history-substring-search
)
```

- [_zsh-completions_](https://github.com/zsh-users/zsh-completions) 原生 zsh 功能补充，可以认为某些功能的尝鲜版稳定了会被加入的官方版本中

```shell
git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:=~/.oh-my-zsh/custom}/plugins/zsh-completions
```

这个插件安装方式有些不同不能加到 plugins 需要在 `source "$ZSH/oh-my-zsh.sh"` 之前添加下面这一行

```shell
...
fpath+=${ZSH_CUSTOM:-${ZSH:-~/.oh-my-zsh}/custom}/plugins/zsh-completions/src
...
source $ZSH/oh-my-zsh.sh
```

效果图

![](../images/ScreenShot-2018-07-20-8-47.png)

### Pure

[Pure](https://github.com/sindresorhus/pure)是一个 zsh 提示，使用`brew`安装

```shell
brew install pure
```

在`.zshrc`后添加

```shell
...
fpath+=("$(brew --prefix)/share/zsh/site-functions")
...
autoload -U promptinit; promptinit
prompt pure
```

### iterm2-snazzy & Menlo-for-Powerline fonts

- [iterm2-snazzy](https://github.com/sindresorhus/iterm2-snazzy)是一个 iTerm 配色方案

下载[Github](https://github.com/sindresorhus/iterm2-snazzy)页上的`Snazzy.itermcolors`到本地，*iTerm2 > Profiles > Colors Tab*页右下角在`Color Presets...`导入`Snazzy`并选择启用。

- [Menlo-for-Powerline](https://github.com/abertsch/Menlo-for-Powerline) 是 Menlo 的 Powerline 的字体。

下载到本地并双击安装字体。*iTerm2 > Profiles > Text Tab*修改字体为`menlo for powerline`，字体大小选`13`。

> 也可以选 [Nerd Fonts](https://www.nerdfonts.com/) 为字体它打得补钉支持更多得图标和符号

最终效果

![](../images/ScreenShot-2018-07-20-9-25-43.png)

PS: 上面第一张图是[Hyper](https://hyper.is/)加[hyper-snazzy](https://github.com/sindresorhus/hyper-snazzy)插件的效果
