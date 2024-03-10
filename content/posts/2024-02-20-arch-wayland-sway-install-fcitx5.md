---
title: "Arch Sway 安装 Fcitx5"
date: "2024-02-20T21:14:17+08:00"
lastmod: "2024-03-10T21:14:17+08:00"
description: manjaro sway wayland 环境安装 Fcitx5
tags: [Arch, Sway, Fcitx5]
categories: [Arch, Sway, Fcitx5]
draft: false
TocOpen: false
UseHugoToc: true
showToc: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
---

主力机 wm 我之前一直使用的是 [i3](http://i3wm.org/) 最近切换到了 Wayland 桌面环境，使用的是 Manjaro Sway 的发行版，在 Wayland 的生态中 [Sway](https://swaywm.org/) 是之前 X11 上 i3 窗口管理器的直接替代选择

发展了好几年，现在 Wayland 下开发环境逐渐成熟，在输入法这方面 [Fcitx5](https://fcitx-im.org/wiki/Fcitx_5) 也已支持，下面是 Sway 安装 Fcitx5 的一些经验，应该所有 Wayland 环境都可参考

## 安装社区版 sway-im

sway 已经合并了实现 text-input-v3 协议的分支，但是仍然没有完全实现 input-method-v2 协议，因此仍然无法显示弹出窗口。有个支持 v2 的 AUR 包 [sway-im](https://aur.archlinux.org/packages/sway-im) 修复了这个[问题](https://wiki.archlinuxcn.org/wiki/Fcitx5#Sway)，使用 sway-im 替换自带的 sway

```shell
yay -S sway-im
```

## 相关包安装

安装 Fcitx5 相关的包，我使用的是双拼所以还安装了双拼相关的插件

```shell
pacman -S fcitx5 fcitx5-chinese-addons fcitx5-qt fcitx5-gtk fcitx5-config-qt qt5-wayland
```

## 配置 Fcitx5

参考[文档](https://wiki.archlinuxcn.org/wiki/Fcitx5#%E9%9B%86%E6%88%90)编辑 `/etc/environment` 并添加以下几行

```shell
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
```

- `SDL_IM_MODULE` 是为了让一些使用特定版本 SDL2 库的游戏能正常使用输入法。
- `GLFW_IM_MODULE` 是为了让 kitty 启用输入法支持。此环境变量的值只能为 ibus。

按 Fcitx5 上游推荐，环境变量的值一般设置为 `fcitx`。部分并非由 Arch 从源码编译打包的应用程序因兼容性的需求而需要将之临时设置为 `fcitx5`

重启电脑查看状态栏输入法选择配置 `Shunagpin` 根据需要配置方案即可，如果没有启动可以手动启动先测试 `/usr/bin/fcitx5 -D` 记得加入到开机启动比如在 sway config 中添加

> 也可以使用输入 `fcitx5-configtool` 打开配置界面

如果使用 en_US.UTF-8 时，遇到 GTK2 无法激活 fcitx5，可专门为该 GTK2 应用程序设置输入法为 xim，如下启动应用

```shell
env GTK_IM_MODULE=xim *<your_gtk2_application>*
```

请勿将 `GTK_IM_MODULE` 全局设置为 `xim`，因为它也会影响 GTK3 程序。`XIM` 有各种问题（比如输入法重启之后再无法输入），尽可能不要使用。

### 特殊应用

按照上面的配置大部分应用能使用 Fcitx5 引擎了，但有些应用需要特殊配置

#### Chrome/Chromium

chrome内核的浏览器现在 Wayland 上还不支持 GTK IM [^1] ，启动的时候可以加上 `--gtk-version=4`，修改 `~/.config/chrome-flags.conf` 如下

[^1]: https://wiki.archlinux.org/title/Fcitx5#Fcitx5_not_available_in_Chromium_running_on_Wayland

```shell
--gtk-version=4
--ozone-platform=wayland
--ozone-platform-hint=wayland
....
```

上面的启动参数可能会使候选框漂移但是可以呼出输入法的，勉强能用。

#### Kitty

增加环境变量 `GLFW_IM_MODULE=ibus` 上面有提到 [^2]

[^2]: https://wiki.archlinux.org/title/Fcitx5#Fcitx5_not_available_in_kitty

## 配置皮肤

这里推荐两个主题

- [ Fcitx5-Material-Color ](https://github.com/hosxy/Fcitx5-Material-Color)
- [ fcitx5-themes ](https://github.com/thep0y/fcitx5-themes)

我使用的是第一个主题可以直接用 `pacman` 安装，具体可参考文档

```shell
pacman -S fcitx5-material-color
```

安装完后在图形界面配置 `Configure - Addons - Classic User Interface` 中 theme 和 dark-theme 中选择 `Material-Color-Indigo` 即可，顺便还可配置喜欢的字体

![](../images/fcitx5_theme.png#center)

## 使用 RIME 输入法

如果你想使用 RIME 输入法，直接再安装

```shell
pacman -S fcitx5-rime rime-double-pinyin rime-emoji
```

重启，配置中选择 `Rime` 输入法即可, 用户配置目录为 `~/.local/share/fcitx5/rime`

## Reference

- https://wiki.archlinux.org/title/Fcitx5
- https://wiki.archlinuxcn.org/wiki/Fcitx5#%E9%9B%86%E6%88%90
