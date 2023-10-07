---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: "2021-04-19T03:25:09+08:00"
lastmod: "2023-05-31T11:25:36+08:00"
showToc: true
tags: [Mac, Launchd]
title: Mac Launchd 介绍和使用
---

## Mac Launchd 介绍和使用

Linux 上如果想开机开机启动一个服务或者定时运行一个服务有很多的选择比如之前介绍过的[Systemd](../2018-08-16-systemd-timer-unit/)或者用 crontab 也可以，而在 Mac 不同它有一个类似的叫 Launchd 的系统，对应使用`launchctl`命令控制

### Daemons and Agents

Launchd 管理 Daemons 和 Agents 两种类型分别存放在不同的文件夹下，主要的区别是

1. Agents 是用户登录后执行的
2. Daemons 是开机后就执行，可以通过`UserName`指定用户比如`root`用户

### 配置文件

Launchd 配置文件以`.plist`结尾，本质上是`xml`格式的文件，Daemons 和 Agents 各存放的路径也不同

| 类型           | 路径                            | 说明                                  |
| -------------- | ------------------------------- | ------------------------------------- |
| User Agents    | `~/Library/LaunchAgents`        | 用户 Agents 当前用户登录时运行        |
| Global Agents  | `/Library/LaunchAgents`         | 全局 Agents 任何用户登录时都会运行    |
| System Agents  | `/System/Library/LaunchAgents`  | 系统 Agents 任何用户登录时都会运行    |
| Global Daemons | `/Library/LaunchDaemons`        | 全局 Daemons 内核初始化加载完后就运行 |
| System Daemons | `/System/Library/LaunchDaemons` | 系统 Daemons 内核初始化加载完后就运行 |

系统运行开机首先会加载内核启动`kernel_tas(0)`，然后启动`launchd(1)`好后去启动指定好的 Daemons 最后用户登录再运行相应的 Agents 任务

一般文件名都以`com.domain.programName.plist`格式命名，不管是 Daemons 还是 Agents 格式都是一样的，只是存放位置不同。看下面一个 hello world 的例子 `~/Library/LaunchAgents/com.example.hello.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.example.hello</string>

    <key>ProgramArguments</key>
    <array>
        <string>/bin/echo</string>
        <string>hello world</string>
    </array>

</dict>
</plist>
```

上面定义了一个最简单的任务只使用了`Label`和`ProgramAgruments`两个键

- `Label`这是个必须的键，指定这个任务名
- `ProgramArguments`是带参数的可执行文件上面等同于运行`/bin/echo hello world`命令，如果执行的程序不带参数可以使用`Program`键，但一个任务中必须包含这两个中的其中一个键

还有一些常用的键名，所有的键可参考`man 5 launchd.plist`或者[这里](https://www.launchd.info/)

| Keys                   | Description                             |
| ---------------------- | --------------------------------------- |
| `EnvironmentVariables` | 设置运行环境变量                        |
| `StandardOutPath`      | 标准输出到文件                          |
| `StandardErrorPath`    | 标准错误到文件                          |
| `RunAtLoad`            | 是否再加载的时候就运行                  |
| `StartInterval`        | 设置程序每隔多少秒运行一次              |
| `KeepAlive`            | 是否设置程序是一直存活着 如果退出就重启 |
| `UserName`             | 设置用户名只在 Daemons 可用             |
| `WorkingDirectory`     | 设置工作目录                            |

### launchctl 命令

现在我们就加载和运行一个任务，上面定义了`~/Library/LaunchAgents/com.example.hello.plist`，我们修改一下增加一个键(`StandardOutPath`)用于标准输出

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.example.hello</string>

    <key>StandardOutPath</key>
    <string>/tmp/hello.log</string>

    <key>ProgramArguments</key>
    <array>
        <string>/bin/echo</string>
        <string>hello world</string>
    </array>

</dict>
</plist>
```

上面的配置把标准输出重定向到了`/tmp/hello.log`，我们运行测试下。检查配置文件是否写正确可以使用`plutil`命令

```shell
❯ plutil ~/Library/LaunchAgents/com.example.hello.plist
/Users/fython/Library/LaunchAgents/com.example.hello.plist: OK

❯ launchctl load ~/Library/LaunchAgents/com.example.hello.plist

❯ launchctl start com.example.hello

❯ cat /tmp/hello.log
hello world

❯ launchctl list | grep hello
-	0	com.example.hello

❯ launchctl remove com.example.hello  # remove jobs
```

一个任务首先需要被加载(load)，然后启动(start)正常运行完退出，所以我们查看`/tmp`目录下会有日志输出

1. 任务一般都要手动启动(start)，如果设置了`RunAtLoad`或者`KeepAlive`则在`launchctl load`时就启动
2. 使用`launchctl list`列出当前加载的任务，第一列代表进程 id，因为上面的程序运行一次就退出了所以显示`-`，第二列是程序上次运行退出的 code，`0`代表正常退出，如果是正数代表退出的时候是有错误的，负数代表是接收到信号被终止的
3. `launchctl stop <service_name>`可以终止一个在运行中的任务，`launchctl unload <path>`指定路径卸载一个任务，`launchctl remove <service_name>`通过服务名卸载任务
4. `launchctl load <path>`只会加载没有被**disable**的任务，可以加`-w`参数 `launchctl load -w <path>`覆盖如果设置了 disable 的，下次开机启动一定会起来。`launchctl unload <path>`只会停止和卸载这个任务，但下次启动还会加载，可以使用`-w`参数`launchctl unload -w <path>`停止任务，下次启动也不会起来，也就是标记了**disable**
5. 调试一个任务可以配合使用`plutil`命令检查语法，设置`StandardOutPath`、`StandardErrorPath`、`Debug`键，也可以看看苹果自带的`Console.app`应用中的`system.log`

### 一些例子

以下是我一些使用过的配置文件

#### 调换 mac 键盘右 `command` 和 `option` 键

文件路径 `~/Library/LaunchAgents/com.fython.swapKey.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.fython.swapKey</string>

    <key>RunAtLoad</key>
    <true/>

    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/hidutil</string>
        <string>property</string>
        <string>--set</string>
        <string>{"UserKeyMapping":
    [{"HIDKeyboardModifierMappingSrc":0x7000000e7,
      "HIDKeyboardModifierMappingDst":0x7000000e6},
     {"HIDKeyboardModifierMappingSrc":0x7000000e6,
      "HIDKeyboardModifierMappingDst":0x7000000e7}]
}</string>
    </array>

</dict>
</plist>
```

习惯了传统的键盘布局，希望改变右边 `option (alt)`键的位置，系统设置里面没有开放出来，可以使用上面的命令设置。开机自动执行。

放在用户目录 `~/Library/LaunchAgents/` 用户登录时执行 `hidutil` 命令，然后执行 `launchctl load -w ~/Library/LaunchAgents/com.fython.swapKey.plist` 设置开机启动

#### 开机启动 clash 代理（TUN 模式）

文件路径：` /Library/LaunchDaemons/com.fython.clash.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>

    <key>Label</key>
    <string>com.fython.clash</string>

    <key>RunAtLoad</key>
    <true/>

    <key>UserName</key>
    <string>root</string>

    <key>StandardErrorPath</key>
    <string>/Users/fython/bin/clash/stderr.log</string>

    <key>StandardOutPath</key>
    <string>/Users/fython/bin/clash/stdout.log</string>

    <key>WorkingDirectory</key>
    <string>/Users/fython/bin/clash</string>

    <key>ProgramArguments</key>
    <array>
      <string>/Users/fython/bin/clash/clash</string>
      <string>-f</string>
      <string>config.yaml</string>
      <string>-d</string>
      <string>/Users/fython/bin/clash</string>
    </array>

    <key>KeepAlive</key>
    <true/>

  </dict>
</plist>
```

因为要监听 53 端口所以需要 root 用户启动，而且需要用户登录前就运行所以存放在`/Library/LaunchDaemons/`目录下，配置了`KeepAlive`和`UserName`，也设置了工作目录`WorkingDirectory`，日志也存在这目录下。这个任务加载和其他操作都需要加 sudo，`sudo launchctl load /Library/LaunchDaemons/com.fython.clash.plist`因为配置了`RunAtLoad`它会自动启动，不需要在 start 了

### Reference

1. [https://www.launchd.info/](https://www.launchd.info/)
2. [https://developer.apple.com](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html)
3. [https://apple.stackexchange.com](https://apple.stackexchange.com/questions/280855/changing-right-hand-command-alt-key-order-to-be-like-a-windows-keyboard)
