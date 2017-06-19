---
layout: post
title: "phabricator简单使用技巧"
description: ""
category: 其它
tags: [phabricator]
author: 张毓庆
---  

#phabricator
phabricator是faceBook开源的一套code Review工具，功能很多也很强大，我们现在团队中目前仅用到了code Review这一个核心的功能,[官方网站:]http://phabricator.org

##Code Review
code review在这里分为了两种一种是向代码仓库提交前进行审核，另一种是向代码仓库中提交后进行审核，我们团队中使用push前进行提交。我这里也只介绍push前进行审核，另外一种感兴趣的童鞋可以查看我的博客。[Audit用户指南][1]
##提交前审核
###环境配置
安装Arcanist很简单，从github上拉两个代码库到本地的同一个文件夹就可以了：
<pre><code>
git clone https://github.com/phacility/libphutil.git
git clone https://github.com/phacility/arcanist.git
cd ~
vim .profile
export PATH="$PATH:/Users/Shixiong/Workspace/arcanist/bin"
source ~/.profile
</code></pre>
###在项目中添加引用
<pre><code>
cd  yourproject
vim .arcconfig
{
"phabricator.uri": "http://your.phabricator.site",
"editor": "vim"
}
</code></pre>
###提交Differential

 - 首先安装证书，运行arc
install-certificate，它会提示你用浏览器打开一个链接，获取一个Token，然后粘贴获得的Token按回车即可。
 - 修复项目的Bug(也就是对你的项目做一些改变)。
 - 运行git commit -am “修复了 XX BUG” ，commit你的改动
 - 运行arc diff，提交Differential，它会提醒你填写一些信息：
   <pre><code>
Test Plan – 必填，详细说明你的测试计划；
Reviewers – 必填，审查人的账户，多个使用”,”隔开；
Subscribers – 非必填订阅人，多个使用”,”隔开。
  </code></pre>
 - 提交成功后，审查人就能在Differential收到通知。
 - 审查人将状态修改为Accept Revision表示通过，通过后，作者就可以将代码push到代码库里面去了。

  [1]: https://www.5288z.com/?p=1475
