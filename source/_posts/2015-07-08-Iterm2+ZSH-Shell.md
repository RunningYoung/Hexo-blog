---
layout: post
title: "Iterm2+ZSH 打造终极Shell"
description: "Iterm2+ZSH 打造终极Shell"
category: iOS
tags: [工具类]
imagefeature: pic-2014-09-08.jpg
comments: true
mathjax: null
featured: true
published: true
---

##Iterm2+ZSH 打造终极Shell



  　[点击查看图片](../images/blog/zshShell.png "Title")


<!--more-->


  　![点击查看图片](http://m1.yea.im/1Oa.png "Title")



　效果图如上：

###配置方法：
1.打开任意终端

输入如下

	 sudo curl -L https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh | sh

或者

	wget --no-check-certificate https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | sh

　这里安装oh my sh 里面包含很多主题 这并不重要，最重要的是会出现的问题

[官方主题预览](https://github.com/robbyrussell/oh-my-zsh/wiki/themes)

[最牛叉的主题](https://github.com/jeremyFreeAgent/oh-my-zsh-powerline-theme)

[相关问题解决方案](http://www.zhihu.com/question/31458342)

	修改/root/.zshrc
	ZSH_THEME="agnoster
	DEFAULT_USER="你的名字" 这里只需要设置你的用户就够root不需要管。

其实最重要的就是修改root下的.zshrc 怎么修改呢，可以用vim打开 有时候vim会找不到这个文件
还一种方法是通过命令行显示所有隐藏文件 就能找到了 然后对其进行相应的修改 找了半天才打开的
 
2.修改默认终端

	sudo chsh -s /bin/zsh
	设置zsh为默认终端，zsh是bash的改良版，opensuse 13.1原带

重启

3.重新打开！ xfce！终端

是不是有效果了，可是有乱码？颜色不对？

需要下载新的字库因为自带的有些不支持 所以会出现乱码

[字库下载](https://github.com/powerline/fonts)

　	终端-编辑-首选项-外观-字体-“文泉驿等宽正黑” 12号字体-允许粗体字
继续————颜色项卡-预设-Solarised方案(不同终端操作不同，随机应变即可)
成功！！

###相关文章
　[终极 Shell](http://macshuo.com/?p=676)

　[巴蛮子博客](http://www.cnblogs.com/bamanzi/p/zsh-simple-guide.html)　

　[oh-my-zsh github地址](https://github.com/robbyrussell/oh-my-zsh/wiki/themes)　
　
　
　
　
　