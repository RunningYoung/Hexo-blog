---
layout: post
title: "MAC下Code Review工具Phabricator + Arcanist搭建（part 2）"
description: ""
category: iOS
tags: [Phabricator]
---

## MAC下Code Review工具Phabricator + Arcanist搭建（part 2）

> 上一篇（MAC下Code Review工具Phabricator + Arcanist搭建（part 1））中我们安装了Phabricator 搭建所需的环境。

> 首先在安装Phabricator 之前启动相应的环境.


> ```
> nginx.start
> php-fpm.start
> mysql.start
> ```

<!--more-->

### Phabricator 安装

#### 从github上clone关键组件

先在本机建一个根目录，本文为：~/phabricator (以下用$BASE_DIR代替根目录)，然后

```
git clone https://github.com/facebook/libphutil.git
git clone https://github.com/facebook/arcanist.git
git clone https://github.com/facebook/phabricator.git
```

#### 修改nginx配置文件

```
打开该路径下的配置文件：
/usr/local/etc/nginx/nginx.conf
修改相应的配置如下：
server {
  listen 80;
  server_name pha.yjmyzz.me; # 该域名必须为本机的本地域名或者IP如：127.0.0.1必须带有.的有效ip或者域名
  root      /Users/xxxxx/phabricator/phabricator/webroot; #phabricator安装路径下的对应文件
  try_files $uri $uri/ /index.php;
  location / {
     index   index.php;
     if ( !-f $request_filename ){
       rewrite ^/(.*)$ /index.php?__path__=/$1 last;
       break;
     }
  }
  location /index.php {
    fastcgi_pass   localhost:9000;
    fastcgi_index   index.php;
    fastcgi_param  REDIRECT_STATUS    200;
    fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
    fastcgi_param  QUERY_STRING       $query_string;
    fastcgi_param  REQUEST_METHOD     $request_method;
    fastcgi_param  CONTENT_TYPE       $content_type;
    fastcgi_param  CONTENT_LENGTH     $content_length;
    fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
    fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
    fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;
    fastcgi_param  REMOTE_ADDR        $remote_addr;
  }
}
```

添加上面这一段即可，注意server_name后的域名以及root根目录要换成自己的实际参数。

#### phabricator 启动及配置

直接将php-fpm及nginx启动即可，然后浏览http://pha.yjmyzz.me/ (即：刚才nginx中server配置的域名，本机配置时，可在hosts中增加127.0.0.1 pha.yjmyzz.me以方便测试)，就能看到下面的界面：

![](http://images2015.cnblogs.com/blog/27612/201601/27612-20160109195351387-1637751972.png)

意思是没有配置mysql，系统无法连接mysql，注意下面的4行命令，已经告诉你怎么处理了，按它的提示来就行了，命令行下，进入根目录，输入以下命令：

```
$BASR_DIR/bin/config set mysql.host server-name (127.0.0.1)

$BASR_DIR/bin/config set mysql.port 3306　

$BASR_DIR/bin/config set mysql.user root　

$BASR_DIR/bin/config set mysql.pass ***(换成你的密码)　　
```

设置完成后，再次浏览刚才的界面，就能进去了，可能第1次还会提示创建管理员账号啥的，按提示来就可以了。

进入主界面后，会看到：

![](http://images2015.cnblogs.com/blog/27612/201601/27612-20160109201821793-1196956188.png)

左上角有一段提示：You have xx unresolved setup issues... 这是告诉你还有其它些配置项需要配置，点击这个链接，看提示一个个配。

至此 Phabricator 安装完毕。

### Arcanist 配置

arc 的安装 和 使用 参考一下文章。
[Arcanist的安装](https://secure.phabricator.com/book/phabricator/article/arcanist/)

本文主要 讲解ARC 相对 Phabricator 下的配置。


在需要code review 的相应git工程目录下 创建如下配置文件并写入相应配置如下：

```
创建 .arcconfig 文件

{
  "phabricator.uri" : "http://xx.100.19.xxx:8080", # 相应的phbricator主机地址
  "editor": "vim", # 打开方式
  "base": "git:HEAD^",# 相应的 git 节点，设置为最新git节点
  "arc.feature.start.default": "develop", # 起始 git 分支
  "arc.land.onto.default": "develop", # push 到的 目标分支
  "arc.land.update.default": "rebase", # land 的方式
  "history.immutable": false
}
``` 




