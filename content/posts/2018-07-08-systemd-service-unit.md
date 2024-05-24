---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: "2018-07-08T02:28:21+08:00"
description: .service 模板
lastmod: "2024-05-24T11:35:54+08:00"
showToc: true
tags: [Linux, Shell]
title: Systemd中Service单元介绍
---

## Systemd中Service单元介绍

> Systemd是一个系统管理守护进程、工具和库的集合，用于取代System V初始进程，集中管理和配置类UNIX系统，可见它非常的强大。

Systemd分为多个单元（unit）如服务（.service），挂载点（.mount），套接口（.socket）和设备（.device）等，这里记录使用最多的服务（service）文件的编写。用户自定义的一般存放在`/etc/systemd/sytem/`文件夹下，还有另外的文件夹类debian系列的如下。

| Directory              | Description                                                |
| ---------------------- | ---------------------------------------------------------- |
| `/lib/systemd/system/` | 系统自带的或者程序自带安装的单元存放在此                   |
| `/etc/systemd/system/` | 用户自定义的，此文件夹优先级最高，可以覆盖上面文件夹的内容 |

下面是系统安装openssh-server后，在`/lib/systemd/system/ssh.service`下的服务。

```shell
[Unit]
Description=OpenBSD Secure Shell server
After=network.target auditd.service
ConditionPathExists=!/etc/ssh/sshd_not_to_be_run

[Service]
EnvironmentFile=-/etc/default/ssh
ExecStart=/usr/sbin/sshd -D $SSHD_OPTS
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartPreventExitStatus=255
Type=notify

[Install]
WantedBy=multi-user.target
Alias=sshd.service
```

如上所示一般每个Unit都有各个块(section)组成，由`[]`包裹就是块名。下面的就是配置，直到另一个块开始为止。

### **[Unit] section**

`[Unit]`一般是第一个块文件配置，配置各种元数据(metadata)和其他单元的关系

| Option           | Description                                                                                |
| ---------------- | ------------------------------------------------------------------------------------------ |
| `Description=`   | 这个单元的描述字符串                                                                       |
| `Documentation=` | 文档链接                                                                                   |
| `Requires=`      | 运行这个单元所需要的依赖单元，否则启动失败                                                 |
| `Wants=`         | 和上面`Requires`相似，但是非强限制。如果列出在此的单元没有启动，本单元也还是能启动持续运行 |
| `BindsTo=`       | 和上面`Requires`相似，区别是列出在此的单元终止了，本单元也会停止                           |
| `Before=`        | 在此列出的单元，只有在本单元启动后才会启动。但不是依赖关系，如需依赖配置上述Requires命令   |
| `After=`         | 在启动本单元之前，先要启动在此列出的单元。但不是依赖关系，如需依赖配置上述Requires命令     |
| `Conflicts=`     | 在此列出的单元，不能和本单元同时运行，和`Requires`相反                                     |
| `OnFailure=`     | 在此列出的单元将会在本单元失败后激活                                                       |

还有很多的如`Condition...`和`Assert...`配置详情可以查看手册`man 5 systemd.unit`

### **[Install] section**

`[Install]`一般是最后一个块文件配置，这个是可选项，也就是说可以不配置。只有在开机启动激活(enable)时触发。

| Option        | Description                                                                                                                                                                                                                                                            |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `WantedBy=`   | 指定该单元如何开机启动(enable)，依赖在此列出的单元，有点类似`[Unit]`块中的`Wants`，不同的是它会创建软链接到`.wants`文件夹，如上sshd如果被enable，该单元会创建一个软链接到`/etc/systemd/system/multi-user.target.wants`文件夹下，如果文件夹不存在则创建文件夹再软链接。 |
| `RequiredBy=` | 和上面`WantedBy`类似，但如果在此列出的单元没有激活，本单元会激活失败，同样在`.requires`文件夹下创建软链接。                                                                                                                                                            |
| `Alias=`      | 设置改单元的别名，可以给`systemctl`使用，例如上面sshd的开机启动可以使用 `systemctl enable ssh.service`和`systemctl enable sshd.service`是一样的                                                                                                                        |
| `Also=`       | 列在此的单元，会随着本单元一起激活。                                                                                                                                                                                                                                   |

### **[Service] section**

以上`[Unit]`,`[Install]`一般是通用的，`[Service]`是单独的服务配置一般在`[Unit]`和`[Install]`之间，只用来配置服务(.service)。

其中`Type=`选项指定此服务的类型，Systemd通过此类型管理服务。默认`Type`为`simple`其他如下。

- `simple` 默认选项，以`ExecStart`设置的指令运行程序。
- `forking` 程序从`ExecStart`fork一个子进程，之后父进程退出。
- `oneshot` 一次性进程和`simple`类似但直到程序运行退出systemd才会开始下一个单元，如果没有设置`simple`和`ExecStart`默认为`oneshot`
- `dbus` 通过D-Bus启动，等待D-Bus返回名称后systemd才会启动下一个单元
- `notify` 通过`sd_notify()`发送一个消息通知后，systemd才会启动下一个单元
- `idle` 所有其他单元任务执行完毕后才会启动此单元

| Option             | Description                                                                                                                                                                                                                                                 |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Type=`            | 为`simple`，`forking`，`oneshot` ，`dbus`，`notify`或`idle`                                                                                                                                                                                                 |
| `ExecStart=`       | 指定启用某个程序或者脚本的命令，如果在命令之前加`-`指定脚本运行非零退出也不标记faild，必须使用绝对路径                                                                                                                                                      |
| `ExecStartPre=`    | 启动程序之前执行的命令，可以指定多条。前面加`-`非零退出也继续执行                                                                                                                                                                                           |
| `ExecStartPost=`   | 启动程序后执行的命令。                                                                                                                                                                                                                                      |
| `ExecStop=`        | 指定`systemctl stop unit-name`时运行的命令，如不指定执行stop时直接发送kill信号                                                                                                                                                                              |
| `ExecReload=`      | 指定`systemctl reload unit-name`时运行的命令如更新配置文件                                                                                                                                                                                                  |
| `Restart=`         | 当服务进程正常退出、异常退出、被杀死、超时的时候， 是否重新启动该服务。该值可以是 `always`， `on-success`， `on-failure`， `on-abnormal`，`on-abort`，或者 `on-watchdog`，如`on-failure`表示仅在服务进程异常退出时重启， 所谓"异常退出"是指： 退出码不为"0" |
| `RestartSec=`      | 重启间隔时间                                                                                                                                                                                                                                                |
| `TimeoutStartSec=` | 配置等待启动的时间。如果守护程序服务未在配置的时间内发出启动完成信号，则该服务将被视为失败，并将再次关闭。                                                                                                                                                  |
| `RemainAfterExit=` | 一般与`Type=onshot`使用，当设置为`yes`时，服务即使退出也为active状态，默认为`no`                                                                                                                                                                            |
| `Environment=`     | 指定环境变量                                                                                                                                                                                                                                                |
| `EnvironmentFile=` | 指定环境变量文件                                                                                                                                                                                                                                            |
| `StandardOutput=`  | 指定标准输出 如输出到文件填`file:/tmp/service.log`                                                                                                                                                                                                          |

还有一些可执行文件的特殊前缀，比如上面提到的`-`

| 前缀 | 效果                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `@`  | 如果在绝对路径前加上可选的 "@" 前缀，则可执行文件中`argv[0]`为第二个参数传递给被执行的进程(而不是实际的文件名)，后面跟着指定的进一步参数。                                                                                                                                                                                                                                                                                                               |
| `-`  | 如果在绝对路径前加上可选的 "-" 前缀，那么即使该进程以失败状态(例如非零的返回值或者出现异常)退出，也会被视为成功退出                                                                                                                                                                                                                                                                                                                                      |
| `+`  | 如果在绝对路径前加上可选的 "+" 前缀，那么进程将拥有完全的权限(超级用户的特权)，并且 User=, Group=, CapabilityBoundingSet= 选项所设置的权限限制以及 PrivateDevices=, PrivateTmp= 等文件系统名字空间的配置将被该命令行启动的进程忽略(但仍然对其他 ExecStart=, ExecStop= 有效)                                                                                                                                                                              |
| `!`  | 与 `+` 类似(进程仍然拥有超级用户的身份)，不同之处在于仅忽略 User=, Group=, SupplementaryGroups= 选项的设置，而例如名字空间之类的其他限制依然有效。注意，当与 DynamicUser= 一起使用时，将会在执行该命令之前先动态分配一对 user/group ，然后将身份凭证的切换操作留给进程自己去执行。                                                                                                                                                                       |
| `!!` | 与 `!` 极其相似，仅用于让利用 ambient capability 限制进程权限的单元兼容不支持 ambient capability 的系统(也就是不支持 AmbientCapabilities= 选项)。如果在不支持 ambient capability 的系统上使用此前缀，那么 SystemCallFilter= 与 CapabilityBoundingSet= 将被隐含的自动修改为允许进程自己丢弃 capability 与特权用户的身份(即使原来被配置为禁止这么做)，并且 AmbientCapabilities= 选项将会被忽略。此前缀在支持 ambient capability 的系统上完全没有任何效果。 |

更具体可参考`systemd.service(5)`

### Specifiers

配置文件中可以使用一些以`%`开头的特殊参数(specifier)，这些参数会在加载单元文件时被替换为相应的参数。

| Specifier | Meaning                                                         |
| --------- | --------------------------------------------------------------- |
| `%i`      | 以`@`结尾的单元文件，启动时传`@`后`.`前面的参数，参考下面的示例 |
| `%I`      | 同上但不转义一些特殊符号                                        |
| `%h`      | 用户主目录，root 用户为 `/root`                                 |
| `%T`      | 临时文件目录                                                    |

更多请参考[手册](https://www.freedesktop.org/software/systemd/man/latest/systemd.unit.html#Specifiers)

### **实例**

如下一个最简单启动脚本的例子，依赖是`network-online.target`

```shell
[Unit]
Description=My Miscellaneous Service
Requires=network-online.target
After=network-online.target

[Service]
Type=simple
User=anonymous
WorkingDirectory=/home/anonymous
ExecStart=some_can_execute --option=123
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

加一个守护gunicorn的服务，我一般用在部署flask应用上。

```shell
[Unit]
Description=Gunicorn instance to serve myproject
Requires=network-online.target
After=network.target

[Service]
User=project user
Group=www-data   # nginx group
WorkingDirectory=/path/to/your/project
Environment="PATH=/path/to/venv/bin"
ExecStart=/path/to/venv/bin/gunicorn --workers 4 --bind unix:ftown.sock -m 007 manage:app
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

如果需要添加多个环境变量一种方法是在`.service`文件中添加多条Environment，另一种可以创建一个以`.d`结尾的文件夹，存放命名`local.conf`的配置，如上面的service名为`my-app.service`则创建`my-app.service.d`的空文件夹，再创建`my-app.service.d/local.conf`文件如下

```shell
[Service]
Environment="PATH=/new/path"
Environment="LD_LIBRARY_PATH=/new/path"
```

使用`@`传参，如文件名为 `gpu_worker@.service` 要以 `@` 结尾

```shell
[Unit]
Description=Inference Service on GPU [%i]

[Service]
Restart=always
Environment="CONFIG_FILE=xxxx.yaml"
Environment="CUDA_VISIBLE_DEVICES=%i"
WorkingDirectory=/path/to/dir
ExecStart=/path/to/bin/file

[Install]
WantedBy=multi-user.target
```

启动时如 `systemctl start gpu_worker@0.service` 则 `%i` 会被 `0` 替换

> 如果参数是带有特殊字符的可以使用 `systemd-escape` 命令转义。 如要传 `1,2` 则使用 `systemctl start "gpu_worker@$(systemd-escape "1,2").service"
` 启动，记得将 service 文件中的 `%i` 换成 `%I` 不开启转义

### Reference

1. [https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html](https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html)
2. [https://man7.org/linux/man-pages/man5/systemd.unit.5.html](https://man7.org/linux/man-pages/man5/systemd.unit.5.html)
3. [https://www.freedesktop.org/software/systemd/man/latest/systemd.unit.html](https://www.freedesktop.org/software/systemd/man/latest/systemd.unit.html#Specifiers)
