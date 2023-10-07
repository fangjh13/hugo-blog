---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: "2017-03-18T05:23:25+08:00"
description: Character Sets and Collations
lastmod: "2017-11-24T12:07:53+08:00"
showToc: true
tags: [Mysql, Database, Mariadb]
title: Mariadb Mysql 字符集设置
---

## Mariadb Mysql Character Sets and Collations

> [MariaDB](https://mariadb.org/)数据库管理系统是 MySQL 的一个分支，主要由开源社区在维护，采用 GPL 授权许可。开发这个分支的原因之一是：甲骨文公司收购了 MySQL 后，有将 MySQL 闭源的潜在风险，因此社区采用分支的方式来避开这个风险。

所以说**mariadb**完全兼容**mysql**的设置，完全可以用**mariadb**代替**mysql**。

初次安装完 mariadb 后运行`mysql_secure_installation`

本人`centos7`编辑`/etc/my.cnf`添加如下配置(`utf8mb4`支持 4 个字节的`emoji`表情完全兼容`utf8`，当然也可以换成`utf8`不使用表情)

    [mysqld]
    init_connect='SET collation_connection = utf8mb4_unicode_ci'
    init_connect='SET NAMES utf8mb4'
    skip-character-set-client-handshake
    # 服务端默认字符集
    character-set-server=utf8mb4
    # 连接层默认字符集
    collation-server=utf8mb4_unicode_ci

    [client]
    # 客户端来源数据默认字符集
    default-character-set=utf8mb4

    [mysql]
    # 客户端来源数据默认字符集
    default-character-set=utf8mb4

重启 mariadb 服务，运行`systemctl restart marriadb.service`

登录后`show variables like '%char%'`和`show variables like '%collation%'`查看是否和如下相同。

    MariaDB [(none)]> show global variables like '%char%';
    +--------------------------+----------------------------+
    | Variable_name            | Value                      |
    +--------------------------+----------------------------+
    | character_set_client     | utf8mb4                    |
    | character_set_connection | utf8mb4                    |
    | character_set_database   | utf8mb4                    |
    | character_set_filesystem | binary                     |
    | character_set_results    | utf8mb4                    |
    | character_set_server     | utf8mb4                    |
    | character_set_system     | utf8                       |
    | character_sets_dir       | /usr/share/mysql/charsets/ |
    +--------------------------+----------------------------+
    8 rows in set (0.00 sec)

    MariaDB [(none)]> show global variables like '%collation%';
    +----------------------+--------------------+
    | Variable_name        | Value              |
    +----------------------+--------------------+
    | collation_connection | utf8mb4_unicode_ci |
    | collation_database   | utf8mb4_unicode_ci |
    | collation_server     | utf8mb4_unicode_ci |
    +----------------------+--------------------+
    3 rows in set (0.00 sec)
