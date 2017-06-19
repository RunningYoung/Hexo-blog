---
layout: post
title: "phabricator实战使用"
description: ""
category: 其它
tags: [phabricator]
author: 张毓庆
---  

#phabricator实战使用
根据我们上一节我们队phabricator的认识和基本环境的搭建以后，我们在本讲中更全面的了解到这个工具的使用。

note:在使用这个工具的时候，要求我们对vim有一个基本的使用和了解,因为我们发送code review请求都是通过命令行来发送的。

##Arcanist命令使用

 - 通过arc help 可以查看arc所支持的所有命令
 - 详细的帮助文档，arc help --full
 - arc diff ￼￼￼￼￼￼￼￼￼发送代码差异到codereview系统
 - arc list 显示未提交修改的代码信息
 - arc cover 可以找到某个代码修改的提交人
 - arc patch 适应某个修改，并在这个修改上进行工作
 - arc export 通过Differential功能下载/导出一个补丁文件
 - arc amend 审核git更新提交后的信息
 - arc commit svn提交代码库的更改
 - arc land 向服务器推送git代码库的更改
 - arc branch 可以看到跟过的有关git分支

##在项目中是怎样使用arc的,我在这里简单做一个demo

 1 查看分支的情况

<pre><code>
➜  shane git:(developer) git branch 
 - developer
  master
</code></pre>

 2 在要开发得分支上新开一个分支 
<pre><code>
git branch zhangyuqing developer
git checkout zhangyuqing
</code></pre>

3 进行修改代码工作并提交审核
<pre><code>
vim zhangyuqing.txt
git add zhangyuqing.txt 
git commit -m 'add zhangyuqing.txt'
arc diff --create  develop或者arc diff develop 一般都会强制创建新的审核请求
note: 此处要添加对比的branch名字，否则可能会不知道从何处开始对比
 </code></pre>
arc diff详细信息展示如下:
<pre><code>
fix some bugs

Summary:  fix some bugs
fix some bugs

add zhangyuqing.txt

Test Plan: fix some bugs  @shane

Reviewers: test

Reviewed By: test

Subscribers:

Differential Revision: http://10.100.16.251:8080/D2
</code></pre>
保存后会显示这样一个ID号码
<pre><code>
Created a new Differential revision:
        Revision URI: http://10.100.16.251:8080/D2

Included changes:
  A       fix.txt
  A       zhangyuqing.txt
</code></pre>

这个时候我们使用arc list可以查看到该审核请求的问题信息,并且等待你的leader或者小组其他成员对代码进行review
<pre><code>
➜  shane git:(zhangyuqing) arc list
* Needs Review D2: fix some bugs
</code></pre>
<pre><code>
➜  shane git:(zhangyuqing) arc list
* Accepted D2: fix some bugs
</code></pre>
当你的代码被其他人同意之后，最后一步执行向代码库中提交代码
<pre><code>
➜  shane git:(zhangyuqing) arc land zhangyuqing --onto developer
Switched to branch developer. Updating branch...
The following commit(s) will be landed:

a4715df fix some bugs
12b2e32 add zhangyuqing.txt

Switched to branch zhangyuqing. Identifying and merging...
Landing revision 'D2: fix some bugs'...
Merging zhangyuqing into developer
Already up-to-date.
Pushing change...

Counting objects: 3, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 420 bytes | 0 bytes/s, done.
Total 3 (delta 1), reused 0 (delta 0)
To http://10.3.4.127:8888/yq.zhang08/shane.git
   6ade274..204c773  developer -> developer

Cleaning up feature branch...
(Use `git checkout -b 'zhangyuqing' 'a4715dfa9fabb6a986bb1f14dde8cd36fe958ba3'` if you want it back.)
Done.
</code></pre>

此时再去查看,D2这个问题已经没了，并且系统会自动关闭该审核请求
<pre><code>
➜  shane git:(developer) arc list
You have no open Differential revisions.
</code></pre>
ps: 整个流程到此就结束了，在最后一步如果有冲突的话就会push失败，那么解决方案就跟我们之前解决冲突一样的了，执行arc land它会自动删除你创建的临时分支，并且切回开发得分支中。整个过程中全程邮件发送。

