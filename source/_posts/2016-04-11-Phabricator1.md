---
layout: post
title: "MAC下Code Review工具Phabricator + Arcanist搭建（part 1）"
description: ""
category: iOS
tags: [Phabricator]
---


## MAC下Code Review工具Phabricator + Arcanist搭建（part 1）



> 环境:OS X Yosemite 10.10.5

> 前提：phabricator主要是由php写的，而且是以website方式运行的，所以mac上要先安装好 php + nginx(或apache) + mysql(很多配置会保存在数据库里)。

<!--more-->

### 前期环境搭建

本教程主要采用 homebrew 安装相关环境，所以首先保证电脑中已经安装了homebrew。安装命令如下：

```
终端中输入
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/    Homebrew/install/master/install)"
```

#### PHP环境搭建

最新版的php包中都自带php-fpm，所以只需要安装带有php-fpm的php包即可，命令如下：

```
//因为已经安装homebrew 所以可以直接使用homebrew安装php-fpm
brew tap homebrew/dupes
brew tap homebrew/php
brew install --without-apache --with-fpm --with-mysql php56
```

安装完成 现在我们将php－fpm 添加入环境变量中 方便我们通过终端直接进行启动

```
echo 'export PATH="/usr/local/sbin:$PATH"' >> ~/.bash_profile
#If you use bsh
echo 'export PATH="/usr/local/sbin:$PATH"' >> ~/.zshrc
#If you use ZSH
```

创建文件夹 并启动服务

```
mkdir -p ~/Library/LaunchAgents
ln -sfv /usr/local/opt/php56/homebrew.mxcl.php56.plist ~/   Library/LaunchAgents/
launchctl load -w ~/Library/LaunchAgents/   homebrew.mxcl.php56.plist
```

如果没有报出什么bug的话 在终端中键入

```
lsof -Pni4 | grep LISTEN | grep php
```


应该会有下图的显示

```
php-fpm   69659  frdmn    6u  IPv4 0x8d8ebe505a1ae01        0t0  TCP 127.0.0.1:9000 (LISTEN)
php-fpm   69660  frdmn    0u  IPv4 0x8d8ebe505a1ae01        0t0  TCP 127.0.0.1:9000 (LISTEN)
php-fpm   69661  frdmn    0u  IPv4 0x8d8ebe505a1ae01        0t0  TCP 127.0.0.1:9000 (LISTEN)
php-fpm   69662  frdmn    0u  IPv4 0x8d8ebe505a1ae01        0t0  TCP 127.0.0.1:9000 (LISTEN)  `
 
```

#### mysql 安装

```
brew install mysql
ln -sfv /usr/local/opt/mysql/*.plist ~/Library/LaunchAgents
```

添加mysql到环境变量中

```
launchctl load ~/Library/LaunchAgents/  homebrew.mxcl.mysql.plist
```


测试mysql

```
mysql -uroot -p
```

输入权限密码 这是你应该能看见

```
Type \'help;\' or \'h\' for help. Type \'c\' to clear the     current input statement.
 
mysql> exit  # 退出mysql
```

#### nginx的安装

使用homebrew来安装nginx

```
brew install nginx
```

我们必须确保80端口是开启的，因为nginx是基于80端口的

```
sudo cp -v /usr/local/opt/nginx/*.plist /Library/LaunchDaemons/
sudo chown root:wheel /Library/LaunchDaemons/homebrew.mxcl.nginx.plist
```

第一次开始nginx

```
sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.nginx.plist
```

默认的配置设置是将监听8080端口而非http默认的80端口

```
curl -IL http://127.0.0.1:8080
```

终端中应该有如下显示

```
HTTP/1.1 200 OK
Server: nginx/1.6.2
Date: Mon, 19 Oct 2014 19:07:47 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Mon, 19 Oct 2014 19:01:32 GMT
Connection: keep-alive
ETag: \"5444dea7-264\"
Accept-Ranges: bytes`
```

我们停止nginx 对其进行进一步配置

```
sudo launchctl unload /Library/LaunchDaemons/homebrew.mxcl.nginx.plist
```

#### 在nginx 中 配置 php-fpm  

```
打开该路径下的nginx配置文件
/usr/local/etc/nginx/conf.d
下相应位置添加如下配置
location ~ \.php$ {
    try_files      $uri = 404;
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include        fastcgi_params;
}
```

#### 最后对PHP启动相关的命令 添加全局变量

```
// If you use Bash 打开如下路径下的文件
~/.bash_profile
// If you use ZSH 打开如下路径下的文件
~/.zshrc

在 aliases 的位置 添加如下配置：

alias nginx.start='sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.nginx.plist'
alias nginx.stop='sudo launchctl unload /Library/LaunchDaemons/homebrew.mxcl.nginx.plist'
alias nginx.restart='nginx.stop && nginx.start'
alias php-fpm.start="launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.php56.plist"
alias php-fpm.stop="launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.php56.plist"
alias php-fpm.restart='php-fpm.stop && php-fpm.start'
alias mysql.start="launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist"
alias mysql.stop="launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist"
alias mysql.restart='mysql.stop && mysql.start'
alias nginx.logs.error='tail -250f /usr/local/etc/nginx/logs/error.log'
alias nginx.logs.access='tail -250f /usr/local/etc/nginx/logs/access.log'
alias nginx.logs.default.access='tail -250f /usr/local/etc/nginx/logs/default.access.log'
alias nginx.logs.default-ssl.access='tail -250f /usr/local/etc/nginx/logs/default-ssl.access.log'
alias nginx.logs.phpmyadmin.error='tail -250f /usr/local/etc/nginx/logs/phpmyadmin.error.log'
alias nginx.logs.phpmyadmin.access='tail -250f /usr/local/etc/nginx/logs/phpmyadmin.access.log'
```
现在我们可以输入以下命令更新我们的变量 让我们的设置生效

```
source ~/.bash_profile
//or
source ~/.zshrc`
```

现在你可以使用更加简短的命名来优雅的开关服务 而不必打一长串的指令 路径 那很傻

```
nginx.start
nginx.stop
nginx.restart
```

### 总结 

至此 PHP + Mysql + nginx 环境搭建完成。在进行下篇文章（Phabricator安装）之前需要启动相应的服务：

```
nginx.start
php-fpm.start
mysql.start
```

























