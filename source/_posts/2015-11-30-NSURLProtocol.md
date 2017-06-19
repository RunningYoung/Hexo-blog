---
layout: post
title: "NSURLProtocol使用"
description: ""
category: ios
tags: net
author: 窦锦坤
--- 

## NSURLProtocol的使用讲解

#### 开发iOS的应用一定离不开网络。iOS的常用网络框架主要分为两种：基于NSURLConnection和基于CFNetwork。基于NSURLConnection的有apple原生和AFNetworking等，基于CFNetwork框架有ASIHttpRequest等。而NSURLProtocol能够拦截处理基于NSURLConnection的一切网络请求和***UIWebView***，但是对于基于CFNetwork的网络请求没有作用。

### 什么是NSURLProtocol？
`NSURLProtocol is an abstract class which provides the basic structure for performing protocol-specific loading of URL data.
它是一个抽象类，为载入URL的data的一些特定协议提供基础的结构。`

#### NSURLProtocol是URL加载系统中的的协议支持部分，它功能强大却晦涩。它是一个抽象类，在使用的使用必须子类化来实现新的或者定制已经存在的URL加载行为，实现自己业务的需求。

### NSURLProtocol的使用举例
- 拦截图片加载请求，转为从本地文件加载
- 为了测试对HTTP返回内容进行mock和stub
- 对发出请求的header进行格式化
- 对发出的媒体请求进行签名
- 创建本地代理服务，用于数据变化时对URL请求的更改
- 故意制造畸形或非法返回数据来测试程序的鲁棒性
- 过滤请求和返回中的敏感信息
- 在既有协议基础上完成对 NSURLConnection 的实现且与原逻辑不产生矛盾

#### NSURLProtocol的核心思想是 通过使用它，不用改变网络调用的其他部分，就可以改变URL加载行为的全部细节。简言之，NSURLProtocol就是一个苹果允许的网络调用的中间人。

### NSURLProtocol的使用方法

#### 使用NSURLProtocol必须先子类化，否则不能使用。

#### 注册自定义的子类，代码如下
{% highlight objective-c %}
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    //注册protocol
    [NSURLProtocol registerClass:[CustomURLProtocol class]];
    return YES;
}
{% endhighlight %}

#### 子类化时要override以下方法，逻辑顺序如下：
- +canInitWithRequest: 这个方法是告诉protocol要控制什么样的网络请求。这个方法返回YES 该请求会被控制，否则以下的委托方法不会在被调用，直接进入下一步的处理。
- +canonicalRequestForRequest: 这个方法是对请求的规范处理，这个规范就是自己想要的处理规范，如替换URL、修改URL的参数等。
- -startLoading 和 -stopLoading 前者对应于loading开始从web抓取数据，后者是抓取数据完毕的回调。这两个方法的方法体处理的都是针对protocol客户端进行操作。在这两个方法内要做初始化`self.connection`等操作。
- 每个`NSURLProtocol`的子类实例都有一个`client`属性，该属性对URL加载系统进行相关操作。它不是`NSURLConnection`，但看起来和实现了`NSURLConnectionDelegate`协议的对象非常相似。`<NSURLProtocolClient>`要实现的方法如下：
    - -URLProtocol:cachedResponseIsValid:
    - -URLProtocol:didCancelAuthenticationChallenge:
    - -URLProtocol:didFailWithError:
    - -URLProtocol:didLoadData:
    - -URLProtocol:didReceiveAuthenticationChallenge:
    - -URLProtocol:didReceiveResponse:cacheStoragePolicy:
    - -URLProtocol:wasRedirectedToRequest:redirectResponse:
    - -URLProtocolDidFinishLoading:
    
    这几个委托方法内要做的就是在恰当的时候让`client`调用每一个委托方法。

#### 以上方法并不是全部可以overide方法，还有很多其他的方法，可以实现更加强大的功能。

#### 具体的使用实例就在赘述的了，参看扩展阅读的 [使用NSURLProtocol拦截替换http请求](http://ksnowlv.github.io/blog/2014/08/05/shi-yong-nsurlprotocol-lan-jie-ti-huan-httpqing-qiu/) 和 [iOS开发之--- NSURLProtocol](http://www.jianshu.com/p/7c89b8c5482a) 就会明白，其实很简单，就是按照使用方法 根据自己的需求重写若干个方法而已。

### 几点注意：
- URLProtocol会被以注册顺序的反序访问，所以自己写的protocol比其他内建的protocol拥有更高的优先级。
- 无论对webView还是native的,只要是基于NSURLConnection的网络请求的,可以根据相关特征进行URL拦截并处理。


> 扩展阅读

> [NSURLProtocol](http://nshipster.cn/nsurlprotocol/)

> [NSURLProtocol与NSURLProtocolClient简介](http://www.cnblogs.com/ioslps/p/4269201.html)

> [iOS开发之--- NSURLProtocol](http://www.jianshu.com/p/7c89b8c5482a)

> [使用NSURLProtocol拦截替换http请求](http://ksnowlv.github.io/blog/2014/08/05/shi-yong-nsurlprotocol-lan-jie-ti-huan-httpqing-qiu/)

> [NSURLProtocol Tutorial](http://www.raywenderlich.com/59982/nsurlprotocol-tutorial)
