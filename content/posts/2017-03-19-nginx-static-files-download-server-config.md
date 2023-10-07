---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2017-03-19T08:57:18+08:00'
description: 用Nginx做内网的文件下载服务器
lastmod: '2018-08-06T12:00:47+08:00'
showToc: true
tags: [Nginx]
title: Nginx局域网搭建静态文件下载服务器
---

测试机器Centos7，本地安装`nginx`默认配置文件在`/etc/nginx/nginx.conf`文件下，保持配置文件不修改，确保`include /etc/nginx/conf.d/*.conf`未被注释，默认是有的

###  基本配置

使用虚拟主机，在`/etc/nginx/conf.d/`文件夹下添加如下`download.conf`配置文件

    server {
            listen 80;
            # 访问日志
            access_log /var/log/nginx/d_access.log;
            # 错误日志
            error_log /var/log/nginx/d_error.log;
            server_name download.com;
    
            # 存放文件的目录
            root /var/www/html;
            
            location / {
                    # 开启文件索引
                    autoindex on;
                    # 关闭文件的实际大小on为bytes，off为M、K、G单位
                    autoindex_exact_size off;
                    # 默认为off，显示的文件时间为GMT时间，on为本地时间
                    autoindex_localtime on;
                    # 修复中文乱码
                    charset utf-8,gbk;
            }
    }

运行`nginx -t`检测配置是否准确

`systemctl start nginx.service`启动服务，现在浏览器输入`http://download.com`会列出`/var/www/html`目录下的文件

> 注意：

> 局域网其他机器需要添加`hosts`使其地址解析到服务器，linux在`/etc/hosts`下加一条`x.x.x.x download.com`，`x.x.x.x`为nginx服务器地址

> nginx需要有进入下载文件夹读取的权限

### 配置`Basic Auth`认证

可以为某一目录设置`basic auth`密码认证

1. `htpasswd -c /etc/nginx/passwd username`输入密码创建一个`passwd`文件用于认证
2.  在`server`部分下增加一个`location`，设`/var/www/html/secret`目录为需要密码进入


        location /secret {
                autoindex on;
                autoindex_exact_size off;
                autoindex_localtime on;

                auth_basic "Restricted";
                auth_basic_user_file /etc/nginx/passwd;
        }
        
测试配置文件通过和重启`nginx`现在文件夹`/var/www/html/secret`是需要密码访问下载的