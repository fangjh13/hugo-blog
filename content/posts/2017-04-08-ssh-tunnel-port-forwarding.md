---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2017-04-08T07:06:13+08:00'
description: SSH Tunnel - Port Forwarding
lastmod: '2017-07-20T09:29:42+08:00'
showToc: true
tags: [SSH]
title: SSH进阶 端口转发 内网穿透
---

## SSH Tunnel - Port Forwarding

> Secure Shell (SSH) is a cryptographic network protocol for operating network services securely over an unsecured network. The best known example application is for remote login to computer systems by users.

`SSH`是一个网络协议`sftp`、`scp`、`ssh`都是这个协议。`ssh`一般用来用于远程主机的登录，Linux系统自带`OpenSSH`可以方便的使用。它有一个叫`SSH Tunnel`的东西可以转发流量，下面是几个例子，以便理解和快速使用隧道，详细使用方法，请移步[官方手册](https://www.openssh.com/manual.html)。

### 本地端口转发（Local Port Forwarding）

     -L [bind_address:]port:host:hostport
           Specifies that the given port on the local (client) host is to be
           forwarded to the given host and port on the remote side.

使用`-L`参数，指定一个本地端口，流量通过这个端口转发到目标主机(host)上的端口(hostport)。

例如连接远程主机(example.com)本地的`mysql`服务，我们可以本地开一个端口`3307`转发，这样我们就可以通过`SSH`连接到远程的数据库，不用修改远程服务器的配置就可以实现，而且数据是加密的。

`ssh -L 3307:127.0.0.1:3306 user@example.com`

以上命令中`3307`是本地端口，`127.0.0.1`是远程服务器数据库地址相对于example.com而不是本地的，这个地址还可以是任意example.com能访问的地址，`3306`是数据库监听端口。回车之后如果没有用公钥认证输入密码后看起来像正常远程登录一样，但不影响我们远程连接。`mysql -h 127.0.0.1 -P 3307 -u mysqluser -p`连接到远程数据库，就可以了。

如果想静默执行可以在后面加上`-NnT`不需要时按`Ctrl+c`结束。

`ssh -L 3307:127.0.0.1:3306 user@example.com -NnT`

    -N 不执行任何远程命令
    -n 重定向stdin到/dev/null,阻止读取stdin
    -T 不分配TTY

### 远程端口转发（Remote Port Forwarding）
    
    -R [bind_address:]port:host:hostport
         Specifies that the given port on the remote (server) host is to
         be forwarded to the given host and port on the local side.
   
`-R`参数指定一个远程端口，把远程主机(待登录主机)的端口转发到本地端指定机器(host)的端口。因为client是从本地ssh到远程服务器，而流量先是从远程端口流向本地的，返回的数据再经原路返回，两个方向相反，所以也称反向隧道反向代理。

一个比较实用的例子是用于内网穿透，假设A是公司内网的机器没有公网IP但能连接互联网，因为有一层NAT (Network Address Translation) 阻挡着，你不能从公司以外的网络访问A机器。你有一台B机器，有公网IP并装有`sshd`和`ssh`。当然你可以在路由上设置端口映射来实现A能被访问可公司路由不是谁都能进并愉快玩耍的，这个时候简单的一条ssh命令就能派上用场了，因为A机器也可以远程连接到B，我们在A机器上建立远程端口转发使A机器在远程也能访问到，在A上执行以下命令。

`ssh -R 4488:127.0.0.1:22 user@B -NnT`

`4488`代表在B远程机器上本地监听4488端口转发流量，`127.0.0.1`是本地A机器的地址，`22`是本地A机器的默认ssh端口22，然后在B机器上执行`netstat -nlp | grep 4488`应该如下显示

    tcp      0    0 127.0.0.1:4488        0.0.0.0:*            LISTEN      18556/sshd: Fython

在B机器上执行`ssh -p 4488 user@127.0.0.1`远程连接到A机器，就是这么简单，我们已经穿透了公司内网。

我们继续，现在你在家有一台C机器也处于内网之中和A差不多，你想在家也就是C机器上访问公司的机器A，我们可以这样。

1. 在机器B上编辑`/etc/ssh/sshd_config`配置文件，加上`GatewayPorts yes`允许任意的地址连接，如果为`no`只能是B机器能访问。
2. B机器重启`sshd`服务，`systemctl restart sshd`使上面的配置生效。
3. 同样在A上执行`ssh -R 4488:127.0.0.1:22 user@B -NnT`，可以使用`netstat`查看4488连接是否建立
4. C机器上执行`ssh -p 4488 user@B`来连接A机器

> 注意：以上A和B都必须有ssh的客户端和服务端，C就使用客户端即可。

#### **隧道的维持**

一般`ssh`长时间连接就会中断。使用`autossh`代替`ssh`可以保持隧中断自动连接，确保使用了公钥认证登录服务器。

`autossh -M 8898 -R 4488:127.0.0.1:22  user@B -NnTqf`

    -M 为autossh的参数 指定一个监听端口与ssh无关
    -q 为静默模式，ssh的参数
    -f 后台执行, ssh的参数

这样ssh服务就会长时间在A机器的后台运行

### 动态端口转发（dynamic port forwarding）

    -D [bind_address:]port
         Specifies a local “dynamic” application-level port forwarding.
         This works by allocating a socket to listen to port on the local
         side, optionally bound to the specified bind_address.

还有一个`-D`参数在本地建立一个`SOCKS`代理服务器用于动态端口转发。`SOCKS`是位于会话层的传输协议，能动态的代理端口，这就很合适作为浏览器代理，`SOCKS5`支持UDP的代理，openssh支持SOCKS5。

比如一台机器可以科学上网，我们就可以通过这台机器跨越GFW

`ssh -D 1080 user@example.com -NnTqf`

`1080`代表绑定本地的1080端口，之后在浏览器网络中配置SOCKS代理即可。

接上面远程端口转发的例子，如果想在家通过C机器访问公司内网网站的网站（A机器可访问），本地建立一个动态的端口转发到B的4488端口，经A与B建立的远程隧道就能实现了。

`ssh -D 1080 -p 4488 user@B -NnTqf`

***

**参考：**

[https://en.wikipedia.org](https://en.wikipedia.org/wiki/Secure_Shell)

[https://www.openssh.com](https://www.openssh.com/manual.html)

[https://www.ibm.com](https://www.ibm.com/developerworks/cn/linux/l-cn-sshforward/)