---
layout: post
title: "ios提交appStore 审核失败被拒原因"
description: ""
category: 模板
tags: appStore
author: 范珅
--- 


###ios提交appStore 审核失败被拒原因

####版本3.0.8

#####被拒原因：

	上传screenshot有越狱信息
	
######解决方案：
	
	上传screenshot不能有越狱信息

####版本3.0.9

#####被拒原因

	原因不明：
		可能原因是：
		1.焦点图的广告，有专车优惠信息
		2.专车wap页面跳转专车app（可能是苹果认为跳转到不是一个账号的app）

######解决方案：
	
	1.审核时去掉焦点图广告，上线后再打开
	2.审核时关闭，上线后再允许跳转
	

####2015年11月25日 版本3.1.0

#####被拒原因：

	 App的名字，描述，截图，预览图与app内部的内容和功能描述不一致。
	 
######以下原文：

	From Apple
	3.3 - Apps with names, descriptions, screenshots, or previews not relevant to the content and functionality of the App will be rejected
	3.3 Details
	We noticed that your marketing screenshot(s) do not sufficiently reflect your app in use.

	We've attached screenshot(s) for your reference.

	Next Steps
	Please revise your screenshots to demonstrate the app functionality in use.

	Since your iTunes Connect Application State is Metadata Rejected, we do NOT require a new binary. To revise the metadata, visit iTunes Connect to select your app and revise the desired metadata values. Once you’ve completed all changes, click the “Submit for Review” button at the top of the App Information page.

	NOTE
	Please be sure to make any metadata changes to all App Localizations by selecting each specific localization and making appropriate changes.
	
######解决方案：

	iOS 3.1.0 审核被拒，原因确认是app store的程序截图不符合苹果审核规范。

	为了避免这些问题，提交审核会由两个人一起做，并且是对苹果审核规范非常熟悉的人。

