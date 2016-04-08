---
layout: post
title: "ReactiveCocoa 学习之路(史上最全攻略)"
description: "ReactiveCocoa 学习之路"
category: iOS
tags: [ReactiveCocoa]
imagefeature: pic-2014-09-08.jpg
comments: true
mathjax: null
featured: true
published: true
---

##ReactiveCocoa 学习之路之史上最全攻略

　	本文介绍的是史上最牛叉的ios开发新框架之一，**最大的特点就是：直观和灵活。直观的代码容易编写、阅读和维护，灵活的特性便于应对变态的需求，当然最重要的还是高效。**各路大神(阳神，喵神，唐神等等)都不吝啬对它的赞美之词，而且该框架已被美团等广泛使用，好用的不要不要的..。**本文主要介绍本人在学习ReactiveCocoa的时候的学习过程以及对一些学习资料的总结，欢迎大家批评发炎。**闲话不多吹下面进入正题。

###什么是ReactiveCocoa？　	

　	ReactiveCocoa（其简称为RAC）是由[Github](https://github.com/ReactiveCocoa/ReactiveCocoa) 开源的一个应用于iOS和OS X开发的新框架。RAC具有函数式编程和响应式编程的特性。

　	是在iOS平台上对FRP的实现。FRP的核心是信号，信号在ReactiveCocoa(以下简称RAC)中是通过RACSignal来表示的，信号是数据流，可以被绑定和传递。它主要吸取了.Net的 [Reactive Extensions](https://msdn.microsoft.com/en-us/data/gg577609)的设计和实现。

　	[大神leezhong在博客中提到的比喻](http://limboy.me/ios/2013/06/19/frp-reactivecocoa.html)，可以更好地帮我们理解ReactiveCocoa.


>　	可以把信号想象成水龙头，只不过里面不是水，而是玻璃球(value)，直径跟水管的内径一样，这样就能保证玻璃球是依次排列，不会出现并排的情况(数据都是线性处理的，不会出现并发情况)。水龙头的开关默认是关的，除非有了接收方(subscriber)，才会打开。这样只要有新的玻璃球进来，就会自动传送给接收方。可以在水龙头上加一个过滤嘴(filter)，不符合的不让通过，也可以加一个改动装置，把球改变成符合自己的需求(map)。也可以把多个水龙头合并成一个新的水龙头(combineLatest:reduce:)，这样只要其中的一个水龙头有玻璃球出来，这个新合并的水龙头就会得到这个球。　


###为什么要用ReactiveCocoa
　	Native app有很大一部分的时间是在等待事件发生，然后响应事件，比如等待网络请求完成，等待用户的操作，等待某些状态值的改变等等，等这些事件发生后，再做进一步处理。 但是这些等待和响应，并没有一个统一的处理方式。Delegate, Notification, Block, KVO, 常常会不知道该用哪个最合适。有时需要chain或者compose某几个事件，就需要多个状态变量，而状态变量一多，复杂度也就上来了。为了解决这些问题，Github的工程师们开发了ReactiveCocoa。

　	**其实用简单的一句话来说就是： RAC统一了对KVO、UI Event、Network request、Async work的处理，因为它们本质上都是值的变化(Values over time)。**  

###ReactiveCocoa试图解决什么问题呢
　	[大神唐巧在他博客中这样写道](http://www.devtang.com/blog/2014/02/11/reactivecocoa-introduction/)主要解决三个问题：

　	1.传统iOS开发过程中，状态以及状态之间依赖过多的问题。

　	2.传统MVC架构的问题：Controller比较复杂，可测试性差

　	3.提供统一的消息传递机制	

　	详情请查看[大神博文](http://www.devtang.com/blog/2014/02/11/reactivecocoa-introduction/)在此就不再赘述

###那么问题来了？如何学习ReactiveCocoa呢
　	　	网上教程很多，本人只是做了一下总结，又初级到高级所需要看的一些文章。

####初级 教程
   　	　	对一个新手来说需要掌握最基本的API的使用。必看的一篇文章是[RayWenderlich](http://www.raywenderlich.com/62699/reactivecocoa-tutorial-pt1) 网站提供的系列教程，其详细程度非常牛逼。当然国内大神们早就对该教程进行翻译了 小伙伴们轻松了！！

   　	　	第一部分地址：[ReactiveCocoa入门教程——第一部分](http://benbeng.leanote.com/post/ReactiveCocoaTutorial-part1)

   　	　	第一部分地址：[ReactiveCocoa入门教程——第一部分](http://benbeng.leanote.com/post/ReactiveCocoaTutorial-part2)

####进阶教程
　	进阶教程主要是进一步分析框架的结构以及实现原理。

　	参考资料：

　	1.[ReactiveCocoa github上的readme的中文翻译](http://www.cocoachina.com/ios/20150702/12302.html)

　	2.[介绍ReactiveCocoa框架的每个组件的文章,对熟悉ReactiveCocoa API非常有帮助](http://blog.csdn.net/xdrt81y/article/details/30624469)　

　	3.[美团网官方博客之RACSignal的Subscription深入分析](http://tech.meituan.com/RACSignalSubscription.html)

　	4.[cocoaChina文章-Reactive Cocoa详解](http://www.cocoachina.com/industry/20140621/8905.html)	

　	5.[cocoaChina文章-ReactiveCocoa2实战](http://www.cocoachina.com/industry/20140609/8737.html)

　	6.[cocoaChina文章-说说ReactiveCocoa 2](http://www.cocoachina.com/industry/20140115/7702.html)

　	7.[NSHipster上的文章-Reactive​Cocoa](http://nshipster.com/reactivecocoa/)

　	8.[国外牛人的一篇文章-Data-Driven iOS Development with ReactiveCocoa](https://www.bignerdranch.com/blog/data-driven-ios-development-reactivecocoa/)

　	9.[国外牛人的一篇文章-Getting Started with ReactiveCocoa ](http://www.teehanlax.com/blog/getting-started-with-reactivecocoa/)

　	10.[NSHipster上一篇关于Recative cocoa的介绍-Reactive​Cocoa](http://nshipster.cn/reactivecocoa/)

　	11.[cocoaChina文章-【长篇高能】ReactiveCocoa 和 MVVM 入门](http://www.cocoachina.com/ios/20150526/11930.html)


　	12.[ReactiveCocoa常用语法-这样好用的ReactiveCocoa，根本停不下来](http://supermao.cn/zhe-yang-hao-yong-de-reactivecocoagen-ben-ting-bu-xia-lai/)

####书籍
　	[当然如果你比较豪，想买点书看看可以选择这个--《Functional Reactive Programming on iOS》](https://leanpub.com/iosfrp)

####源码　	
　	1.[使用ReactiveCocoa框架编写的app源码之《MVVM-IOS-Example》](https://github.com/Machx/MVVM-IOS-Example)

　	2.[使用ReactiveCocoa框架编写的app源码之《GroceryList》](https://github.com/jspahrsummers/GroceryList)

　	3.[使用ReactiveCocoa框架编写的app源码之《C-41》](https://github.com/AshFurrow/C-41)


####视频
　	[ReactiveCocoa at MobiDevDay 2013视频](https://vimeo.com/65637501)　	
　
　