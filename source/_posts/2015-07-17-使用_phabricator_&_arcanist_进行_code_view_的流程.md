---
layout: post
title: "使用 Phabricator & Arcanist 进行 Code Review 的流程"
description: ""
category: 开发工具
tags: [git, Phabricator, Arcanist]
author: 陈灿
---  

#使用 Phabricator & Arcanist 进行 Code Review 的流程

之前我们讲过[ Git 使用规范与注意事项](http://10.100.16.251:4000/开发工具/2015/01/19/git.html)，这次我们需要在 git 基础上加入 code review 机制。  
下面 git 命令的简写，请参考上面链接中的内容。

##Before Using Phabricator & Arcanist

###Minor Change
如果只是修改小部分代码，不需要开 feature branch，比如修复某个崩溃，开发工作的流程为：

1. git co develop; git pull
2. 修改代码
3. git commit -a -m "xxxx"
4. 重复2和3
5. git pull --rebase，如果有冲突，解决冲突
6. git push
7. 重复1到6


###Feature Development
如果进行大改动，需要开 feature branch，比如业务改造，开发流程会多增加几步：

1. git co develop; git pull
2. **git co -b new_feature_branch; git push -u origin new_feature_branch**
3. 修改代码
4. git commit -a -m "xxxx"
5. 重复3和4
6. git pull --rebase，如果有冲突，解决冲突
7. git push
8. **重复上面3～7步**
9. **git co develop; git merge --no-ff new_feature_branch**
10. **git branch -d new_feature_branch**
11. 重复1到10

也就是说，在 new_feature_branch 开发完成前，修改都是在 new_feature_branch 上进行 commit，最后new feature 完成后，再 merge 到 develop。

##After Using Phabricator & Arcanist
[phabricator实战使用](http://10.100.16.251:4000/其它/2015/07/06/phabricator实战使用.html)，描述了一次 arc diff 以及 arc land 的过程。  

phabricator的code review流程，比较灵活，但需要掌握的配置也多，这里是它的[官方文档](https://secure.phabricator.com/book/phabricator/)。

code review 大致的流程为：  
1. 修改代码  
2. arc diff，生成 revision，它的状态为 Need Review  
3. 邮件通知 reviewer 进行审核。如果审核不过，它会被置为 Need Revision，再次 diff 后，会重新回到 Need Review 状态；如果审核通过，它会被置为 Accepted  
4. arc land 后，revision 的状态会变为 Closed  

真实的开发情况会比上面描述的流程复杂一些，比如我们不会等待一次 code review request 被Accepted 后，才继续开发，而是等待过程中继续开发。

为了适应真实的开发情况，制定了一个流程和规范。

先说一下流程图中的一些概念： 
  
* code review 分支  
	如果我们将之前的 develop 称为** develop分支**，new_feature_branch 称为** feature 分支**，我们现在引入一个新的概念，叫做 **code review 分支**。

	Minor change 时，会关系到** develop分支**和 **code review 分支**，执行的 land 命令为 **arc land code_review_branch --onto develop**

	Feature Development 时，会关系到** develop分支**、** feature 分支**、 **code review 分支**，执行的 land 命令为 **arc land code_review_branch --onto new_feature_branch**。之后会在 develop 上执行 git merge --no-ff new_feature_branch。

* 依赖性 revision 和 独立性 revision  
	在 crb1(code review branch 1) 上建立一个 revision 之后，
	* 如果接下来的开发依赖这个 revision 的修改，需要在 crb1 基础上，再 git co -b crb2。然后在 crb2 上执行 arc  diff --create 建立 revision，这个叫依赖性 revision。  
		land 依赖性 revsion 时，需要按依赖顺序，依次land，不能跳跃。可以用 revision id 来辨别顺序。如果跳跃 land 一个revision id更大的依赖性 revsion，会将它所依赖的 revision 也push上去。
	* 如果接下来的开发不依赖这个 revision 的修改，从最新的 develop 或者 feature 分支上，co 新的code review branch即可。
	

[Code_Review_Flow](/attachment/code_review/Code_Review_Flow.pdf)

![](/attachment/code_review/Code_Review_Flow.jpg)


##配置文件及命令详解

上面这个流程里面的命令的行为，依赖一些 .arcconfig 中的一些配置。我们目前的配置文件为：

<pre>
{
  "phabricator.uri" : "http://10.100.16.251:8080",
  "editor": "vim",
  "base": "git:HEAD^",
  "arc.land.onto.default": "develop",
  "arc.land.update.default": "rebase",
  "history.immutable": false
}
</pre>

还有其他的一些配置，可以用arc get-config --verbose查看。

这里的配置是什么意思，后面我会结合下面命令讲一下。

###arc diff --create
它是一个比较重要的命令，这个是关于它的[官方文档](https://secure.phabricator.com/book/phabricator/article/arcanist_diff/)。  

--create，不要省略，diff默认是create、还是update，是依情况而定，不好判断。  

由于我们的配置文件，这个命令实际执行了 git commit -a;arc diff --create HEAD^。

省略git commit -a，是因为arc diff会帮你运行这个命令，之后才真正运行arc diff，diff的rang，也是在commit之后才确定。

省略HEAD^，是因为 .arcconfig 中的 **"base": "git:HEAD^""**，"base"的值是一些 rule，其他的 rule 还有"base": "git:merge-base(origin/develop), arc:prompt"等。

这个参数或者设置的意思是，HEAD^ 为arc diff 的 **the beginning of the range**。
>
In Git and Mercurial, many arc commands (notably, arc diff) operate on a range of commits beginning with some commit you specify and ending with the working copy state.  
Since **the end of the range is fixed (the working copy state), you only need to specify the beginning of the range**. 

###arc land code_view_branch
arc land code_view_branch实际执行了 arc land --onto develop --squash --revision DXX --update-with-rebase。

省略 --onto develop，是因为 .arcconfig 中的 "arc.land.onto.default": "develop"。

省略 --update-with-rebase，是因为 .arcconfig 中的 "arc.land.update.default": "rebase"。

省略 --squash 是因为 
.arcconfig 中的"history.immutable": false。如果为 true，对应 --merge。

这些参数和配置的含义是什么呢？这就需要描述一下 arc land 这个命令执行的内容。大家可以看看某次 arc land 的输出:
<pre>

Can@bogon:~/Workspace/zuche another_code_view_branch$ arc land code_view_branch --onto develop
Switched to branch develop. Updating branch...
The following commit(s) will be landed:

996e636 remove wechat sdk

Switched to branch deletefile. Identifying and merging...
Landing revision 'D46: remove wechat sdk'...
Rebasing code_view_branch onto develop
Current branch code_view_branch is up to date.
Pushing change...

Counting objects: 4, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 585 bytes | 0 bytes/s, done.
Total 4 (delta 2), reused 0 (delta 0)
To git@10.3.4.127:fz.rong/zuche.git
   a25fb28..481e168  develop -> develop

Cleaning up feature branch...
(Use `git checkout -b 'code_view_branch' '996e636bf77b84e3f9795eadc6e241006c6639a9'` if you want it back.)
Switched back to branch testontest.
Done.

</pre>
我是在 another_code_view_branch 上执行的 arc land code_view_branch，也就是是说，arc land 的对象是 branch，如果不传入，默认为当前branch，因为我在another_code_view_branch上面，所以必须传入。

从输出可以看到这个命令的内容为：

1. 首先 switch 到 develop，然后 git pull 或者 git pull --rebase。  
  如果不带--onto develop，会读取 .arcconfig 中的 arc.land.onto.default 配置项。  
  是 pull 还是 pull --rebase，参见 arc get-config --verbose 中 arc.land.update.default，或者 arc help land 中 --update-with-merge、--update-with-rebase 解释。
2. 然后 git rebase 或 merge code_review_branch 到 develop 分支。  
  参见 arc get-config --verbose 中 history.immutable，或者 arc help land 中 --merge、--squash 解释。squash 是 rebase 的一种，详细可以参考这个[文章](http://blog.chinaunix.net/uid-27714502-id-3436706.html)。
3. git push origin develop。
4. git branch -D code_review_branch。


###arc patch revision-id
这个命令是 code reviewer 用来 review 代码的，网页端 review 代码只能静态看，而 patch 代码后，可以在IDE里面 reivew 代码，并且可以运行起来动态调试。

运行这个命令前，要保证本地的develop或feature，包含revision所在分支的起始commit，最好先在develop或者feature执行git pull --rebase。


##要点和规范
* 一个rivision只对应一次commit，并且只对应一个code review branch。
* 操作方面：
	* 用arc diff --create替代git commit 
	* 用arc land替代git push
* 当你发现自己diff的代码有比较大的问题，修改需要review：  
	* 如果还没有被accepted，应该通知reviewer，避免他已经开始 review 了。然后进行arc diff —update。
	* 如果已经被accepted，应走新建依赖性revision。因为update 一个 Accepted 的 revision，它的状态不会变成 Need Review。为什么会这样，下面有说明。
* 当一个你review代码时发现一些比较小的问题，小到即使改了，你不想再review了，应该accept，并添加对这比较小的问题的comment。  
	这样做是避免不必要的打回，给作者带来气馁的情绪。    
	所以才有这样的设定：update 一个 Accepted 的 revision，它的状态不会变成 Need Review。而一个accepted的revision，作者还是可以进行minor change 的，并且是不需要review 的。
	官方有[解释](https://secure.phabricator.com/book/phabricator/article/differential_faq/)。
	>
	Although this behavior is configurable, we think stickiness is a good behavior: stickiness encourage authors to update revisions when they make minor changes after a revision is accepted. For example, a reviewer may accept a change with a comment like this:  
	<pre>Looks great, but can you add some documentation for the foo() function before you land it? I also caught a couple typos, see inlines.</pre>  
	If updating the revision reverted the status to "Needs Review", the author is discouraged from updating the revision when they make minor changes because they'll have to wait for their reviewer to have a chance to look at it again.
* 一般情况下，所有改动都是需要review的，即使比较小，也怕不小心修改了其他内容。

##经验技巧

###arc cover 命令
arc cover，能列出当前working tree到diff的beginning of the range，之间所涉及到的文件的之前的其他改动者。有点拗口，简单来说，就是推荐给你一些reviewer。  
我们配置diff的beginning of the range为HEAD^，修改代码后，直接运行arc diff，首先会协助你commit，然后才真正diff，此时diff就仅仅包含你的这次commit的代码。

###Commandeer的用法
Commandeer乃code review神器，但需要对git和arc比较熟多情况下才能用。下面我描述下它的威力：

1. 刚才fanshen提了D121给我review，然后我arc patch  D121，然后git  reset HEAD^，用xcode看看修改。
2. 发现一些有些问题，心想着这些问题其实我直接改了更快些，然后update D121，再让fanshen看看。
3. 修改好后我在arc diff —update D121，提示：You don't **own** revision D121 '去掉几个不必要的IfDEBUG，修改DDLog'. You can only update revisions you own. You can '**Commandeer**' this revision from the web interface if you want to become the owner.
4. 按照第3步提示，我Commandeer了D121，发现我和fanshen的角色互换了，在执行arc diff --update D121成功。
5. 然后我发现有些地方我改错了，继续修改并arc diff --update D121。
6. fanshen通过后，我来land。

###查看revision每次的update

我们有个规范是一个rivision只对应一次commit。为什么呢，这里解释一下，rivision可以多次update，phabricator的code review界面能显示每个update的修改，但如果rivision有多次commit，首先，代码粒度有些大，其次，phabricator并不提供查看每次commit修改的功能。

![](/attachment/code_review/update.jpg)

###关掉已经review过到文件的diff界面
已经review过的，可以通过view options，选Collapse File，关掉这个文件的diff界面。
![](/attachment/code_review/view_options.jpg)

###Inline comment
按住鼠标左键拖动选择一或多行：
![](/attachment/code_review/in_line_comment.jpg)

###提交develop或者feature分支上的修改
有时候因为某些原因，忘记创建code review branch直接在develop上尝试修改代码，修改好后想diff，可直接git co -b crb创建，因为没git add。
