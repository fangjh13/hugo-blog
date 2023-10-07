---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2017-12-27T03:00:27+08:00'
description: SSH agent 和 SSH agent forwarding
lastmod: '2018-07-29T09:37:55+08:00'
showToc: true
tags: [Shell, SSH]
title: 配置SSH agent 和 SSH agent forwarding转发
---

### 0x01. ssh-agent转发

经常使用[SSH](https://en.wikipedia.org/wiki/Secure_Shell)而且私钥设置了`passphrase`的同学会遇到一个问题，就是每次登录主机都要输入一遍`passphrase`会很麻烦，这时[`ssh-agent`](https://www.freebsd.org/cgi/man.cgi?query=ssh-agent&sektion=1)命令就有用了。

`ssh-agent`是OpenSSH默认自带的并且在后台一直运行的daemon。假设已经通过`ssh-keygen`[生成](https://help.github.com/articles/connecting-to-github-with-ssh/)了自己的私钥，可以在GitHub上传公钥后使用`ssh -T git@github.com`命令测试通过。

```shell
$ ssh-agent
SSH_AUTH_SOCK=/var/folders/k6/k5s6nj_j2k70jzp7plv120y40000gn/T//ssh-fcn9OCl9La13/agent.2462; export SSH_AUTH_SOCK;
SSH_AGENT_PID=2463; export SSH_AGENT_PID;
echo Agent pid 2463;
```

但上面的命令没用，我们需要`eval`命令导入环境变量

```shell
$ eval $(ssh-agent)
Agent pid 2533
$ echo $SSH_AGENT_PID
2553
$ echo $SSH_AUTH_SOCK
/var/folders/k6/k5s6nj_j2k70jzp7plv120y40000gn/T//ssh-6bHKwvKJ6AO1/agent.2532
```

`ssh-agent`已经运行了，最后需要把私钥加入cache。

```shell
$ ssh-add ~/.ssh/id_rsa
Enter passphrase for /Users/fython/.ssh/id_rsa: 
Identity added: /Users/fython/.ssh/id_rsa (/Users/fython/.ssh/id_rsa)
```

可以使用`ssh-add -l`查看缓存的私钥。现在再登录使用公钥认证的主机不需要输入`passphrase`了。

这个每次开机都要运行一遍上面的命令好像有点麻烦，值得高兴的是大多数的linux发行版都在登录图形界面时都会启动一个`ssh-agent`进程，你不需要任何操作，可以使用`ps -ef | grep ssh-agent`查看。如果你的系统没有这个功能，请在`~/.xsession`文件中加入：

```shell
ssh-agent gnome-session
```
> Note：请使用你自己的窗口管理器取代`gnome-session`。

本人使用Arch，i3桌面环境的时候需要在`～/.profile`中加入才能启用

```shell
export $(gnome-keyring-daemon --start --components=secrets,ssh)
```

### 0x02. 使用SSH agent forwarding多机器转发

设想一下当你有两台server如A和B，你都可以ssh上去但你已经登入了上了A想从A上面ssh到B又不想将私钥上传到A该怎么办，一个很好的办法是开启`ForwardAgent yes`。

修改全局配置`/etc/ssh/ssh_config`或修改个人`~/.ssh/config`配置推荐第二种，没有就新建文件，然后加入

```shell
Host *
　　ForwardAgent yes
```

最后去A和B服务器上`~/.ssh/config`也加入如上的配置。现在可以愉快的在A上免密连接到B了，当然在B上面也可以ssh到A，其实只要在服务器配置中加入了`ForwardAgent yes`这样就能传递了。

### 0x03. 在MacOS上的设置

由于MacOS上每次开机都会忘记Key，只要在`~/.ssh/config`中添加如下配置即可和实现上面的效果

```shell
Host *
 AddKeysToAgent yes
 UseKeychain yes
 ForwardAgent yes
 IdentityFile ~/.ssh/id_rsa
```

或者手动运行`ssh-add -K yourkey`将key加入到Keychain也可以。

#### 参考

[https://wiki.archlinux.org/index.php/SSH_keys](https://wiki.archlinux.org/index.php/SSH_keys#SSH_agents)

[http://mah.everybody.org/docs/ssh](http://mah.everybody.org/docs/ssh)

[https://developer.github.com/v3/guides/using-ssh-agent-forwarding/](https://developer.github.com/v3/guides/using-ssh-agent-forwarding/)