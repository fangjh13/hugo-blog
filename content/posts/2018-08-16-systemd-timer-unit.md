---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2018-08-16T09:43:23+08:00'
lastmod: '2018-08-16T09:44:27+08:00'
showToc: true
tags: [Linux, Shell]
title: Systemd 中的timer单元
---

## Systemd 中的timer单元

[上一篇](https://www.fythonfang.com/blog/2018/7/8/systemd-service-unit)讲了systemd 中的service单元，这次记录一下 timer 单元。timer 必须依赖 service 单元来配置，可以用来做替代 crontab 的选择。

timer单元以`.timer`结尾，中间包含`[Timer]`块如下面所示是 Ubuntu下的`apt-daily.timer`，该目录下也存在一个`apt-daily.service`服务文件配合一起使用。

```shell
[Unit]
Description=Daily apt activities

[Timer]
OnCalendar=*-*-* 6,18:00
RandomizedDelaySec=12h
AccuracySec=1h
Persistent=true

[Install]
WantedBy=timers.target
```

上面的[Timer]块代表每天上午6点和下午6点都运行 apt 脚本，具体[Timer]块可配置以下参数

#### 单调定时器(Monotonic timer)

| Option | Description |
| --- | --- |
| OnActiveSec= | 相对计时器开始后多少时间执行，格式如2h、2s、2w、2d |
| OnBootSec= | 相对系统启动后多少时间执行 |
| OnStartupSec= | 相对 systemd 启动多少时间后执行 |
| OnUnitActiveSec= | 每隔多少时间再次运行一次 |
| OnUnitInactiveSec= | 服务在最后一次停止后，隔多久再执行一次 |

可以两个参数一起使用，如下每周开机15分钟后执行 foo

```shell
[Unit]
Description=Run foo weekly and on boot

[Timer]
OnBootSec=15min
OnUnitActiveSec=1w 

[Install]
WantedBy=timers.target
```
#### 实时定时器(Realtime timer)

| Option | Description |
| --- | --- |
| OnCalendar= | 相对系统时间指定特定时刻运行，它接受如2h、2s 的格式也可以是 `星期 年-月-日 时:分:秒`的格式，`..`指定区间，`*`代表所有的。可参考systemd.time(7) |
| Persistent= | 是一个布尔值，默认为 no，当使用 OnCalendar 的设置时，指定该功能要不要持续进行。如断电恢复后是不是要执行上次没执行的 |
| AccuracySec= |设置定时器的触发精度。默认值是一分钟。定时器并不必然在所设置的精准时间点上启动匹配单元， 而是在所设置的精准时间点为起点的一小段时间窗口范围内的某个时间点上启动匹配单元， 这个时间窗口的起点由 OnCalendar=, OnActiveSec=, OnBootSec=, OnStartupSec=, OnUnitActiveSec= or OnUnitInactiveSec= 决定， 而这个时间窗口的长度则由该指令决定。|
| RandomizedDelaySec= | 将此单元的定时器随机延迟一小段时间， 这一小段时间的长度介于零到该指令设置的时间长度之间， 以均匀概率分布。|

如下是每月的1到4号12点周一和周二运行 foo，格式如`OnCalendar=*-*-* 4:00:00`代表每天4点

```shell
[Unit]
Description=Run foo weekly

[Timer]
OnCalendar=Mon,Tue *-*-01..04 12:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

所有`timer`单元可以像`service`一样使用`systemctl status|enable|disable name.timer`查看信息，可以使用`systemctl list-timers`列出运行中的 timer，加`--all`参数列出包含未激活的。

```shell
ubuntu➜  ~  ᐅ  sudo systemctl list-timers --all
NEXT                         LEFT     LAST                         PASSED       UNIT                         ACTIVATES
Thu 2018-08-16 06:52:42 CST  7h left  Wed 2018-08-15 06:22:54 CST  17h ago      apt-daily-upgrade.timer      apt-daily-upgrade.service
Thu 2018-08-16 09:48:35 CST  10h left Wed 2018-08-15 20:15:54 CST  3h 10min ago apt-daily.timer              apt-daily.service
Thu 2018-08-16 21:34:50 CST  22h left Wed 2018-08-15 21:34:50 CST  1h 51min ago systemd-tmpfiles-clean.timer systemd-tmpfiles-clean.service
n/a                          n/a      n/a                          n/a          snapd.refresh.timer
n/a                          n/a      n/a                          n/a          snapd.snap-repair.timer      snapd.snap-repair.service
n/a                          n/a      n/a                          n/a          ureadahead-stop.timer        ureadahead-stop.service

6 timers listed.
```