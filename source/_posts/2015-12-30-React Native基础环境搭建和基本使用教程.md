---
layout: post
title: "React Native基础环境搭建和基本使用教程"
description: ""
category: 开发工具
tags: [git]
author: shaneZhang
---  


##简单介绍

[官方网站][1]
[github][2]

react native是facebook开源的一个项目，它结合了H5页面的灵活性和原生应用的高用户体验性，可以说是两者比较完美的结合。而且现在随着安卓那边资源的整合正在往跨平台的路上大步前行了。当前已经被很多国内主流的大公司改造并应用，比如阿里和携程等等。

##基础环境搭建

###nvm的安装
nvm的安装需要使用[官方教程][3]
<pre><code>
git clone https://github.com/creationix/nvm.git ~/.nvm && cd ~/.nvm && git checkout `git describe --abbrev=0 --tags` 
. ~/.nvm/nvm.sh
添加环境变量 
export NVM_DIR="$HOME/.nvm" 
激活nvm 
$NVM_DIR/nvm.sh
</code></pre>
###附加环境的搭建和使用
<pre><code>
nvm install node && nvm alias default node 
brew install watchman 
brew install flow 
更新本地的库 
brew update && brew upgrade
</code></pre>

###快速开始体验基础环境
<pre><code>
npm config set registry https://registry.npm.taobao.org
npm config set disturl https://npm.taobao.org/dist
npm install -g react-native-cli
react-native init AwesomeProject
cd AwesomeProject
</code></pre>

用xcode打开运行一下吧

##React Native初遇

对于React-Native开发，仅仅有基础前端开发的知识是不够的，你还需要了解和掌握的有：

- Node.js基础
- JSX语法基础
- Flexbox布局

PS 友情提示： 
如果不小心将电脑默认的服务器关闭后，可以通过以下方式运行：
切到项目的根目录，命令行输入npm start即可，这样即可开发终端监听。实际上也是node.js的监听服务开启而已。

React Native中文翻译资料网站：[传送门][4] 

###如何引入React Native到当前的工程中

在这里我们先来创建一个xcode工程，命名为ReactNativeDemo

前提是必须先要把React Native的基础环境搭建完毕，如果没有好得话可以参见我之前的博客来搭建基础环境，包括node 和cocoapods环境

<pre><code>
cd {workspacce}
npm install react-native --save
vim Podfile
# 取决于你的工程如何组织，你的node_modules文件夹可能会在别的地方。
# 请将:path后面的内容修改为正确的路径。
pod 'React', :path => './node_modules/react-native', :subspecs => [
'Core',
'RCTImage',
'RCTNetwork',
'RCTText',
'RCTWebSocket',
# 添加其他你想在工程中使用的依赖。
]
pod install
</code></pre>

然后在工程目录下创建index.ios.js

<pre><code>
vim index.ios.js
'use strict';

var React = require('react-native');
var {
Text,
View
} = React;

var styles = React.StyleSheet.create({
container: {
flex: 1,
backgroundColor: 'red'
}
});

class ReactNativeDemo extends React.Component {
render() {
return (
<View style={styles.container}>
<Text>This is ReactNative Demo .</Text>
</View>
)
}
}

React.AppRegistry.registerComponent('ReactNativeDemo', () => ReactNativeDemo);
</code></pre>

到这里跟官方文档教程保持一致,然后就是应用和使用了

在需要使用的地方创建一个ReactNativeView，其实React Native完全可以看做是一个UIView的子类来使用

在需要引用的地方添加以下代码

<pre><code>
NSURL *jsCodeLocation = [NSURL URLWithString:@"http://localhost:8081/index.ios.bundle"];
RCTRootView *rootView = [[RCTRootView alloc]initWithBundleURL:jsCodeLocation moduleName:@"ReactNativeDemo" initialProperties:nil launchOptions:nil];
[self addSubview:rootView];
rootView.frame = self.bounds;
</code></pre>

然后启动电脑的node服务

<pre><code>
cd node_modules/react-native
npm start
</code></pre>

下面可以尝试运行工程了

我在尝试的时候遇到了一个问题就是一直不成功，原因在于ios9限制了http的使用，需要强制使用http的方式，具体配置方式可以网上自行搜索添加。

[我的工程demo][5]


###参考学习资料
[github一些汇总][6]


[1]: http://facebook.github.io/react-native/
[2]: https://github.com/facebook/react-native
[3]: https://github.com/creationix/nvm#installation
[4]: http://react-native.cn/docs/getting-started.html
[5]: https://code.5288z.com/zhangyuqing/ReactNativeDemo
[6]: https://github.com/ele828/react-native-guide
