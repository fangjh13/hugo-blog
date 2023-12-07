---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: "2021-01-20T07:50:48+08:00"
lastmod: "2023-12-07T11:34:52+08:00"
showToc: true
tags: [Bash, Linux]
title: Bash备忘录
---

# BASH SHELL cheat sheet

记录一些 bash shell 脚本的奇技淫巧，都是从实际使用中 google 的。bash 各 Linux 发行版都自带方便好用特别是文本处理、一些运维之类的小脚本，但有些语法繁琐不好记容易忘整理一下方便查找。

## `trap`

**[trap 命令](http://tldp.org/LDP/Bash-Beginners-Guide/html/sect_12_02.html)** 用于指定在接收到信号后将要采取的动作，常见的用途是在脚本程序被中断时完成清理工作。当 shell 接收到 sigspec 指定的信号时，arg 参数（命令）将会被读取，并被执行。例如：

```bash
trap "exit 1" HUP INT PIPE QUIT TERM
```

表示当 shell 收到`HUP,INT,PIPE,QUIT,TERM`这几个信号时，当前执行的程序会读取参数`"exit 1"`，并将它作为命令执行。

如果要忽略某个信号就参数使用单引号就可以`''`

```bash
trap ''  signals
```

如果启动的时候忽略了信号比如使用了`nohup`，trap 命令是无效的。具体信号可使用`man 7 signal`或者`kill -l`查阅

## `$*`、`$@`、`$#`

直接上例子看，如下脚本`test.sh`

```bash
echo 参数总个数 \$#: $#
echo 第0个参数 \$0: $0
for a in $(seq 1 $#); do
    eval b=\$$a
    echo 第"$a"个参数 \$"$a": $b
done

echo -e "\nUsing \"\$#\":"
echo "$#"

echo -e "\nUsing \$#:"
echo $#

echo -e "\nUsing \"\$*\":"
for a in "$*"; do
    echo $a;
done

echo -e "\nUsing \$*:"
for a in $*; do
    echo $a;
done

echo -e "\nUsing \"\$@\":"
for a in "$@"; do
    echo $a;
done

echo -e "\nUsing \$@:"
for a in $@; do
    echo $a;
done
```

然后运行此脚本，注意最后`3 4`用了双引号

```bash
bash test.sh 1 2 "3 4"
```

结果如下

```bash
❯ bash test.sh 1 2 "3 4"
参数总个数 $#: 3
第0个参数 $0: test.sh
第1个参数 $1: 1
第2个参数 $2: 2
第3个参数 $3: 3 4

Using "$#":
3

Using $#:
3

Using "$*":
1 2 3 4

Using $*:
1
2
3
4

Using "$@":
1
2
3 4

Using $@:
1
2
3
4
```

`$#`是命令的参数总数，不算`$0`所以是 3，加不加引号效果一样。

然后`$*`和`$@`总结一下就是，在没有双引号的情况下`$*`和`$@`效果一样，加上了双引号`$*`会把参数当成一串字符串一次性处理，`$@`加双引号会以空格分隔成列表一般也是我们需要的效果。

## `$!`

`$!`代表最近在后台执行的进程 pid

```bash
❯ nohup sleep 10 > /dev/null 2>&1 &
[1] 79627

❯ echo $!
79712
```

可以配合`wait`命令一起使用，等待任务结束

```bash
❯ nohup sleep 30 > /dev/null 2>&1 &
❯ recent_pid=$!
❯ # do some else ...
❯ wait $recent_pid   # wait for complete finish
❯ echo $?            # check status
0
```

## 计算单词数

可以使用`wc -w`计算，如

```bash
wc -w <<< "a b c"     # 3
```

## `read`命令

从标准输入（std input）读取一行

```bash
while read -r line;
do
        echo "> $line <"
done
```

## 从管道读取内容到变量

如下面为`./run.sh`文件

```bash
INPUT=$(</dev/stdin)

echo "$INPUT"
```

然后`echo hello world | ./run.sh`，其中`INPUT`变量就是输入内容

## fd 命令

取代`find`, 如下一次性把所有当前目录及子目录下的所有apk移动到特定目录

```bash
find . -type f -name '*.apk' -exec mv -t /target/path {} +

# replace use fd
fd -t f -e apk --exec-batch mv -t ./target/path
```

## ack/ag/rg 命令

`ack`和`ag`命令是高级版本的`grep`，[`ack`](https://github.com/samaaron/ack)使用 Perl 编写更能发挥正则的威力，而[`ag`](https://github.com/ggreer/the_silver_searcher)是`ack`的高性能版，而且加上了`.gitignore`搜索速度会更加的快

1. `ack --python 'regex'` 或者`ag --python 'regex'`

   搜索所有 python 文件中包含 regex，ack 默认结果只显示匹配到的内容开头不加行号可以使用`-H`显示 ag 会默认加上行号，可以使用`-l`参数只显示文件名，`-c`显示每个文件匹配到次数，`-w`只匹配单词

2. `ack -g 'regex'` 或者 `ag -g 'regex'`

   只搜索文件名匹配的，列出文件

3. `rg` 比 `ag` 还要快，它是用 Rust 写得。

## 端口扫描 Nmap

1. `nmap -p- -v -A -T4 xxx.xxx.xxx.xxx`

   全范围端口扫描，`-p`指定端口范围`-p-`为全部范围，`-A`指定探测系统和各端口对应的服务`-T4`设定速度但耗时还是会比较长，想要快可以不加参数`nmap -v xxx.xxx.xxx.xxx`

2. `nmap -sT xxx.xxx.xxx.xxx`

   单独扫描`TCP`端口，可以把`-sT`改成`-sU`就是单独扫描`UDP`协议的

3. `nmap -sn 192.168.2.0/24`

   ping 扫描不做端口扫描，上面检测`192.168.2.0/24`网段所有可以 ping 通的机器，检测有多少机器在线很有用

## TCP连接

### 状态统计

```bash
ss -nat | awk 'NR > 1 { ++counts[$1] } END { for (c in counts) print c, counts[c] }'
```

### 特定端口连接查看

```bash
ss -o state established '( dport = :22 or sport = :22 )'`
```

## 远程机器执行本地脚本

```bash
ssh user@machine "$(< script.sh)"
```

## 备份远程机器磁盘

```bash
ssh user@machine "dd if=/dev/sda | gzip -1 -" | dd of=backup.gz
```

## Reference

[https://www.shellscript.sh](https://www.shellscript.sh/trap.html)
[https://www.computerhope.com](https://www.computerhope.com/unix/bash/read.htm)
[https://securitytrails.com](https://securitytrails.com/blog/top-15-nmap-commands-to-scan-remote-hosts)
