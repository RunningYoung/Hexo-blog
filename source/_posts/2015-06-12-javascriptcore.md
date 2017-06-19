---
layout: post
title: "JavaScriptCore框架介绍"
description: ""
category: 库
tags: [JS, iOS]
author: 陈灿
---  

#JavaScriptCore

JavaScriptCore.framework提供了Objc与javascript互相调用的可能。javascript是脚本语言，这意味着我们可以动态改变Objc程序行为。同时，苹果官方对通过JavaScriptCore动态改变app的行为，也不是完全不允许的。  

大伙们利用这个 framework，封装了一些框架，比较有代表性的是:  

* *Xamarin*: 目标是 *Write once, run anywhere*。这个目标很远大，但缺陷很多，这个是微软支持的技术，我们就不细说了。
* [*React Native*](http://facebook.github.io/react-native/docs/getting-started.html#content): 目标是 *Learn once, write anywhere*。它是由facebook开发和维护。听说[淘宝也大量使用这个框架](http://www.zhihu.com/question/27852694/answer/43990708)。用它能用在一些比较简单的场景。  
* [*JSPatch*](https://github.com/bang590/JSPatch): 给程序打补丁。

前面两个框架太重，文档也比较完备，志向也比较远大。  
第三个却非常简单，运用起来灵活些。但需要做一些增强。  

##框架使用

苹果官方没有提供关于JavaScriptCore.framework的 Guide 文档，不过在 [wwdc2013](https://developer.apple.com/videos/wwdc/2013/)，有个主题是 *Integrating JavaScript into Native Apps*。

我是这么理解这个框架的：

* JSContext是javascriptcore给oc留的大门，进去的是NSString，出来的是JSValue。
* JSValue是指向JS运行环境里的对象的引用，部分类型的js对象可以转换成OC或者swift对象，如果需要增加类型，需要自定义一个JEExport协议，并定义一个OC或者swift类型遵守这个协议，这样，JS那边会自动生成一个JS的类型。


##ZCJSPatchService

> JSPatch让脚本语言获得调用所有原生OC方法的能力，不像web前端把能力局限在浏览器，使用上会有一些安全风险：  
1.若在网络传输过程中下发明文JS，可能会被中间人篡改JS脚本，执行任意方法，盗取APP里的相关信息。可以对传输过程进行加密，或用直接使用https解决。  
2.若下载完后的JS保存在本地没有加密，在未越狱的机器上用户也可以手动替换或篡改脚本。这点危害没有第一点大，因为操作者是手机拥有者，不存在APP内相关信息被盗用的风险。若要避免用户修改代码影响APP运行，可以选择简单的加密存储。

除了上面两点，还有对补丁的管理也是个问题。  

但由于补丁很少用到，我们不用那么复杂。可以这么弄:

* 向后台发送一个get请求，使用https，传入App版本号和build号，获得最新的补丁。对补丁（可以为空）md5，如果和之前的不一样，则保存下来。运行从网络获取的补丁。
* 如果网络失败，运行本地补丁。
* 对补丁对保存和读取，是用加密，并验证版本。
* 对补丁的本地保存，只是应对没有网络或请求失败的情况。
* 不管从网络、还是本地获取补丁，其实都是对版本做了验证。

ZCJSPatchService实现上面的这些逻辑。

##ZCPatch

ZCPatch是对ZCJSPatchService的进一步封装。

它的逻辑是：MAPI的start接口，可以屏蔽ZCJSPatchService。如果不屏蔽，会从友盟获取一个在线配置，这个配置是获取js代码的url，我们再用这个url下载js代码。


###打补丁的步骤

补丁和OC代码应该是独立的，它们在不同的git仓库里面，下面步骤里面的`cd patch_zuche`和`cd zuche`，表示进入不同的git本地仓库里。  

patch_zuche库里面每次打补丁，都是在develop上开发，完成后，merger到master，但是不需要打tag。

当遇到用户或测试反馈的bug时，我们需要做的事情是：

1. 检查和修改OC代码
2. 确定bug影响的版本范围，分析并确定给哪些版本做补丁
3. 修改的代码和JS代码，需要经过code review和测试

  
总结了一个打补丁的步骤：

1. 首先看看最新的develop分支代码有无问题，如果有则进行修改，并让测试人员测试。
2. 完成OC代码后，分析下这个bug影响到哪些版本，针对每个版本进行下面几个步骤的修改。
3. 比如3.0.8有问题，`git co 3.0.8`。
4. `cd patch_zuche;git co develop;git pull --rebase;git co -b code_review_branch1;open .`，并将3.0.8.js拖入zuche项目的Xcode工程(如果没有3.0.8.js，新建一个，并`git add 3.0.8.js`)，注意**copy item if needed 也不能勾选**。将`kZCH_ZCPatch_LocalJSPatch`设为`3.0.8.js`，然后开启`mZCH_ZCPatch_useLocal`宏。  
5. js代码写好后，`arc diff`。
6. code reivew通过后，`arc land code_review_branch1`。然后将3.0.8.js代码放到测试的服务器上，让测试人员看看。
7. 测试通过后，将3.0.8.js代码放到生产的服务器，并`git co master;git merge --no-ff develop;git push`。
8. `cd zuche;git co .;git co develop;git pull --rebase`。

注意：  

1. ZCPatch是3.0.7和以后的版本才集成了
  所以之后3.0.7和以后的版本，才支持补丁。
2. kZCH_ZCPatch_LocalJSPatch在3.0.9和以后的tag才存在
  所以，如果需要给3.0.7和3.0.8打补丁，需要使用最新develop的ZCPatch.m才行，需要先拷贝这个文件的内容，执行上面第3步，再粘贴内容到ZCPatch.m。

因为本地运行和走网络运行，有少许差别，最好预生产测试一下。  
预生产补丁配置方法：   
1. http://caradminpre.zuche.com/caradmin/login.jsp xuser/Zc123456  
2. 右上方，更多->其他管理
3. 左上方，网站管理->iOS升级信息（名字没取好）


##React Native

既然facebook和淘宝都用了，说明这个东西还是有些用处的。

[Integration with Existing App](http://facebook.github.io/react-native/docs/embedded-app-ios.html) 是集成到现有app的文档。

下面是写集成文档的作者多一些看法：  

> 我也在我的课程项目中用了React Native。实际上我个人对React Native的看法是，它特别适用于很多初创产品的构建。初创产品迭代快、快速试错，用React Native可以很快地满足这一需求，而不是用原生代码在那里死抠。之前Hybrid App在我心里实际上也是这个作用，只不过React Native在体验上比Hybrid App好很多。  
将React Native大规模部署到现有的大型应用中可能还是要仔细评估一下的，不谈性能，光从工程开发的角度可能存在一些问题。官方文档里面Integration with Existing App最初是我起草的。在做这个文档的过程中经过试验，我个人认为每个React Native应用还是要单独开发，然后再利用一套自动化流程（grunt或者gulp、webpack）整合到现有的native应用中，而不是直接插在现有项目里面开发。这样第一点保持了模块化，测试好跑，第二点对于现有的React Native项目而言，这样做也是比较自然的。  
然后有一个缺点就是目前涉及到RCTxxx模块的调用的一些函数，单元测试没法跑…比如，我自己要做一个模块封装AsyncStorage，叫做Storage，可是并没有办法测试它，因为测试的宿主环境我没有办法模拟。现在只有iOS，后面还有Android，就更不知道怎么测试了。我暂时还没找到解决方法。  
另外需要注意苹果的一个条款，应用中不能下载网络上的代码来执行，所以就算要升级bundle也是要整个应用发一个版本。所有bundle必须一起打包到ipa。  
所以，我觉得小小的用一下可以，铺开还是需要时间。


