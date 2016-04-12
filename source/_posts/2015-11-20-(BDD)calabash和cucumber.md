---
layout: post
title: "BDD开发 calabash 和 cucumber的使用"
description: "Calabash是一款开源的跨平台UI测试工具，目前支持iOS和Android。"
category: iOS
tags: [BDD]
imagefeature: pic-2014-09-08.jpg
comments: true
mathjax: null
featured: true
published: true
---


##【BDD】calabash 和 cucumber的使用

　	本文介绍的是BDD的开发新框架之一----- [Calabash-ios](https://github.com/calabash/calabash-ios)。

　	**Calabash是一款开源的跨平台UI测试工具，目前支持iOS和Android。它使用Cucumber作为测试核心，Cucumber是一个在敏捷团队十分流行的自动化的功能测试工具，它使用接近于自然语言的特性文档进行用例的书写和测试，支持多语言和多平台。**

<!--more-->

　

	Calabash-ios 简单步骤：

	1.安装ruby gem
	2.安装calabash-cucumber，安装后你的mac机器上应该能够使用calabash-ios命令了
	3.git clone或者svn co下来你IOS代码，IOS工程主目录是有一个.xcodeproj的文件的，找到它进入到该目录
	4.calabash-ios setup
	5.calabash-ios gen
	6.In Xcode, build your project using the -cal scheme
	7.cucumber
  以下是详细步骤：
###前期准备工作

　	Calabash是一个ruby的库文件，为避免Calabash在运行中出现各种问题,尽量保证电脑中的ruby环境是最新版的，或者2.1以上版本。mac系统 10.11自带的是2.0版本的.需要升级到最新版本 建议升级ruby 用RVM 工具统一管理ruby库。

详细步骤:[点击此处查看](https://ruby-china.org/wiki/install_ruby_guide)

如果使用RVM升级时出错！！还有大招：通过 [ruby-install](https://github.com/postmodern/ruby-install#readme) 安装(一定要安装到RVM中方便统一管理ruby) 方法如下：

	Using ruby-install with RVM:

	$ ruby-install --rubies-dir ~/.rvm/rubies ruby 2.2.2

安装完成之后再用RVM 的查看ruby库列表 然后设置默认ruby库：[点击此处查看](https://ruby-china.org/wiki/install_ruby_guide)
	

###一：基础环境搭建　Calabash-ios	

　	[Calabash-ios](https://github.com/calabash/calabash-ios) 官方wiki 提供了三种方式的xcode工程环境配置。
　	[点击查看](https://github.com/calabash/calabash-ios/wiki/Tutorial%3A-How-to-add-Calabash-to-Xcode)


　	其中最合理的应该是：添加Calabash Config配置文件这种，相比修改debug config这种来说，不会影响项目中原有的config文件，相比添加 -cal targert这种来说，也不会影响项目中工程文件的修改，当项目中导入其他文件时需要同时导入到原target和-cal target中，而添加Calabash Cofig这种则不会。

　	需要注意的是，在根据[Calabash cofig](https://github.com/calabash/calabash-ios/wiki/Tutorial%3A-Calabash-config)这种方法配置环境时该步骤：

	Create a Gemfile in the same directory as your .xcodeproj:

	source "https://rubygems.org"

	gem "calabash-cucumber", ">= 0.16", "< 2.0"


需要通过 bundle（一种管理ruby gem库的工具） 来管理。bundle 管理库命令（bundle（安装），bundle update（更新））:类似于 pod uodate （更新库），pod install（安装库）。配置文件 Gemfile 也类似于 podfile

　	1.安装bundle方法

	在命令行输入$:sudo gem install bundler
	然后输入密码，就会自动安装。

　	2.在项目根目录中创建文件名为 Gemfile 的文件,文件中写入如下内容(需要注意ruby的官方镜像毕竟慢，可以使用大淘宝的镜像)：

	source "https://ruby.taobao.org" 

	gem "calabash-cucumber", ">= 0.16", "< 2.0"

　	3安装相应的calabash-cucumber库.

	命令行切换到工程根目录下 输入$:bundle 

　	4.如果需要更新Gemfile文件中的库版本，可以通过：

	切换到工程根目录 命令行输入$:bundle update

###二：测试用例编写及其运行 Cucumber
####1.创建cucumber必要的文件


	命令行 切换到工程根目录$:calabash-ios gen

　	就会创建feature的文件夹 文件夹下放着测试用例编写，必须的文件：相关目录结构及其功能，[（cucumber的使用）可以看此处](http://blog.csdn.net/liangliang103377/article/details/49928279) 


####2.编写cucumber测试用例

　	相信通过上面 [cucumber的使用](http://blog.csdn.net/liangliang103377/article/details/49928279) 这一文章已经基本了解cucumber的目录结构：下面就开始编写简单地测试用例吧:

　	cucumber 测试用例编写需要一定的时间去了解它的基本的语法的，所以不要着急先看看一个非常全面的demo吧：[官方demo](https://github.com/calabash/ios-smoke-test-app) 当然先大体看下demo了解下结构和语法，要想真正看懂需要继续补充一些知识。以下就是需要结合demo看的文章(大部分文章都是从[calabash官方wiki](https://github.com/calabash/calabash-ios/wiki) 和 [Cucumber 官方wiki](https://github.com/cucumber/cucumber/wiki/A-Table-Of-Content) 中可以获得)：


　	1).[cucumber 目录结构介绍 官方wiki](https://github.com/cucumber/cucumber/wiki/Cucumber-Backgrounder) 

　	2).[calabash 官方wiki --- Calabash iOS Ruby API](https://github.com/calabash/calabash-ios/wiki/Calabash-iOS-Ruby-API)

　	3).[calabash 官方wiki --- Query Language](https://github.com/calabash/calabash-ios/wiki/Query-Language)

　	4).[calabash 官方wiki --- public API](http://calabashapi.xamarin.com/ios/_index.html)


　	5).最重要的还是看demo 进行学习 [其他demo](https://github.com/jmoody/briar-ios-example)

####3.运行cucumber测试用例

　	运行cucumber的时候特别注意：

　	1).首先运行 对应的项目.如果项目中打印出以下语句及Calabash正常开启：

	DEBUG CalabashServer:222 | Creating the server: <LPHTTPServer: 0x7f80b3d066e0>
	DEBUG CalabashServer:223 | Calabash iOS server version: CALABASH VERSION: 0.16.4
　	2).保证工程中的calabash-ios.framework和calabash-cucumber ruby gem库并且最新：如果不一致可以通过此方法升级

	#下载最新的 .framwork
		项目根目录$: bundle exec calabash-ios download  
	#更新calabash-cucumber ruby gem库  
		项目根目录$:bundle update    

　	3).命令行运行 cucumber 执行命令。***需要注意的是：『cucumber运行，启动的模拟器』 一定要跟 『运行工程时，启动的模拟器』 一致否则会发生错误***

	查看工程运行的模拟器UUID两种方法：
	1.方法一 ：Xcode -> Window -> Devices 查看对应的模拟器UUID
	2.方法二 ：命令行输入$: xcrun instruments -s devices
　	执行 cucumber 命令语句：

	#运行某个tag场景方法
	CMD $: bundle exec cucumber DEVICE_TARGET="90E602DD-376F-4EF9-9B82-208D9016B61D"  --tags @travis
	运行整个feature场景方法
	CMD $: bundle exec cucumber DEVICE_TARGET="90E602DD-376F-4EF9-9B82-208D9016B61D"  features/slider.feature
####4.cucumber运行tags 和 feature 的使用

　	1) [cucumber 官方wiki --- tags的使用](https://github.com/cucumber/cucumber/wiki/Tags)

　	2) [cucumber 官方wiki --- 运行 feature 的方法](https://github.com/cucumber/cucumber/wiki/Running-Features)
　	

 　	3）[ruby 学习网站](http://www.runoob.com/ruby/ruby-tutorial.html)




	
　	
	
