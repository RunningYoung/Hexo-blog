---
layout: post
title: "ReactiveCocoa 常用操作示例"
description: ""
category: ios开发
tags: []
author: 饭小团
--- 
#ReactiveCocoa 常用操作示例

这里的内容更多一些⬇️

 * [行为驱动开发iOS](http://www.jianshu.com/p/73f9d719cee4)
 * [这样好用的ReactiveCocoa，根本停不下来](http://supermao.cn/zhe-yang-hao-yong-de-reactivecocoagen-ben-ting-bu-xia-lai/)
 
###what is a signal?

![Alt text](/attachment/fanshen/signal1.png)

一个signal是一个对象，可以发出一连串的数值。如果没有任何subscribers(订阅者)的话，这个signal不会做任何事。只有在一个subscriber监听他（或者说订阅他（subscribed）的情况下才会发送信息。signal可以发出给subscriber空值或者更多的包含“value”值的“next”事件，随后可以跟随complete事件或者error事件。

![Alt text](/attachment/fanshen/signal2.png)

你可以过滤，转换，分裂，合并这些值，不同的订阅者（subscribers）会以不同的方式使用这些signal。

![Alt text](/attachment/fanshen/signal3.png)

###RACSignal
signal为了控制通过app内部的的所有信息流，可以把所有异步函数都统一起来。包括delegates, callback blocks, notifications, KVO, target/action event observers, etc。 统一到一个接口下面。

![Alt text](/attachment/fanshen/signal0.png)

###Where do signals get the values they are sending along?

Signals 是一些异步代码，等待一些事情发生后才发送结果值给他的subscribers。 你可以自己创建signal使用createSignal：函数

###What is a subscriber?

简单来说，一个subscriber是一段等待signal发送值后才执行的代码（“complete”和“error”事件也一样）

这是一个KVO的例子🌰：

	RACSignal *usernameValidSignal = RACObserve(self.viewModel, usernameIsValid);

![](/attachment/fanshen/signal4.png)

观察者在注意viewModel中的usernameIsValid值的变化，一旦发生变化就会产生signal。这时如果有subscriber订阅的话，signal的值就会传给subscriber的block执行。



###CreateSignal

    RACSignal * sig1 = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        
        [subscriber sendNext:@"Hello, "];
        [subscriber sendNext:@"world!"];
        [subscriber sendCompleted];
        return nil;
    }];
    
    [sig1 subscribeNext:^(id x) {
        NSLog(@"%@",x);
    }];

createSignal创建了一个signal，每当这个sig1得到一个新的subscriber时候。这个新的subscriber就被传入到这个block里了。
这里我们向subscriber里传了两个值 hello 和 world，完后sendCompleted结束传值。这样，在sig1 subscribeNext: 里就连续执行了两次，分别打印了hello和world。

####TextField

    RACSignal *signal = [[textField1 rac_textSignal]
					    //map用来处理传入的参数，可以返回任何对象。但这个对象要能对应下面这个backgroundColor值。否则会崩溃
                            map:^id(NSString * value) {                                    
                            	NSLog(@"%@",value);
                             return [UIColor yellowColor];
                         }];
    
    RAC(butn,backgroundColor) = signal;

####Button

    @weakify(self);
    [[[[goWalkbut rac_signalForControlEvents:UIControlEventTouchUpInside]
       doNext:^(id x) {     			//附加操作，我用来更新UI
           @strongify(self);
           [ARProgressHub show];
           self.tableView.hidden = YES;
           otherMapButton.hidden = YES;
           
       } ]
      filter:^BOOL(id value) {       			 //过滤，用来判断一些条件
           @strongify(self);
           if (inCity == NO) {			    	  //这里最好不要改变外部的值，最好把几个值绑定成信号传进来
               noInSameCityLable.text = @"未获取到步行路径";
               [ARProgressHub dismiss];
               return NO;
           }else{
               return YES;
           }
       }]
     subscribeNext:^(id x) {		//订阅下一步执行内容。上一步如果是NO，则此block不执行
           
           naviRequest.searchType = AMapSearchType_NaviWalking;
           [searchAPI AMapNavigationSearch:naviRequest];
       }];
    
    }];
    
    
####基本操作
		
	  1.//数组处理成序列Sequence，对Sequence里的每个值都平方一次
		NSArray*array=@[@1,@2,@4,@5,@6,@7,@8];
        RACSequence * stream= [array rac_sequence];
    	RACSequence * stream2 = [stream map:^id(NSNumber * value){
        			return @(pow([value integerValue], 2));
	    }];
    	NSLog(@"%@",[stream2 array]);
    	
    	
      2.//数组[1,2,3,4,5,6,7,8,]处理为string "aa12345678"
		RACSequence * stream3 = [[stream map:^id(id value){
        		return [value stringValue]; //对数组内的值转为string
	    }] foldLeftWithStart:@"aa" reduce:^id(id accumulator, id value) {
	    	//把“aa”加入到string前面
		    //这里最好打断点来了解.accumulator是上一次值的结果
	        return [accumulator stringByAppendingString:value];
    	}];
    	
    	 NSLog(@"%@",stream3);

    
      3.//数组打包成tuple
    	RACTuple *tuple = RACTuplePack(@"aaaa",@111);
    
		//解包tuple，这里把tuple里的值@“aaaa”赋给string，@111赋值给num
		RACTupleUnpack(NSString *string,NSNumber *num) = tuple;
    	NSLog(@"string = %@ , num = %@ ",string,num);
    
####比较有用的属性

* distinctUntilChanged 如果第二次的输入的值没有变 后续的操作不会执行

		[textField1.rac_textSignal.distinctUntilChanged subscribeNext:^(id x) {
        NSLog(@"1 = %@",x);
    	}];    

####NSNotification 
   
	//用来替换通知，监听UITextFieldTextDidBeginEditingNotification的变化，发生变化执行订阅block里的操作
		[[[NSNotificationCenter defaultCenter]				rac_addObserverForName:@"UITextFieldTextDidBeginEditingNotification"
							object:nil]
	         subscribeNext:^(id x) {
    	        if (!self.bankDict) {
        	        [self showAlert:@"请选择银行"];
            	}
        }];
        
####Delegate

	//可以替换代理函数
	@weakify(self)
    [[self rac_signalForSelector:@selector(actionSheet:clickedButtonAtIndex:)  //替换的代理函数
                    fromProtocol:@protocol(UIActionSheetDelegate)]             //指出是哪个代理
        subscribeNext:^(RACTuple *tuple) {                                    //参数是个RACTuple，里面包含着代理参数
            
            @strongify(self);
            
            UIActionSheet *sheet = tuple.first;
            NSInteger buttonIndex = [tuple.second integerValue];
            ASSERT_Class(sheet, UIActionSheet);
            
            NSString *desAddressStr = [self.model.storeDict objectForKey:@"name"];
            
            NSString *mapType = [sheet buttonTitleAtIndex:buttonIndex]; // 地图类型
            NSString *urlStr; // 调起接口
            
           bala.....bala....bala...            
        }
     ];
     

####KVO
	
	一：
	[RACObserve(self, self.testLabel.text) subscribeNext:^(id x) {
        NSLog(@"%@",@"testlabel");
    }];

	等同于：
	
	RACSignal *testLabelSignal = RACObserve(self, self.testLabel.text);
	
	[usernameValidSignal subscribeNext:^(id x) {
        NSLog(@"%@",@"testlabel");
    }];


	二：
    RACSignal *signal = RACObserve(textField,text);
    [signal subscribeNext:^(id x) {
        NSLog(@"%@",x);
    }];
    
    RAC(lable,text) = signal;
	
	不加subscribeNext，等同于：
	
	RAC(lable,text) = RACObserve(textField,text); 
	
	textField的text可以同步到lable的text。不过textField需要离开它才会同步
	
	
####Merge

		RACSignal * sig = [RACSignal combineLatest:@[textField1.rac_textSignal,
                                                     textField2.rac_textSignal]
                     	 					reduce:^id(id x,id y){
        
                          				    return @[x,y];
	 						}];
    
	    [sig subscribeNext:^(id x) {
        
    	}];
    	
    //combineLatest合并的两个textField信号的最新值
    //reduce可以对传入textField值做处理.而subscribeNext里的参数就是reduce的返回值。
	    [[[RACSignal merge:@[RACObserve(textField1, text),
                             RACObserve(textField2, text)]]
      		bufferWithTime:5 
      		   onScheduler:[RACScheduler mainThreadScheduler]]
    	     subscribeNext:^(RACTuple *value) {

       		   NSLog(@"%@",value);
    	 }];
     //merge 可以把两个信号合起来
     //bufferWithTime 指定缓冲时间.如果你在5秒内输入完两个textField。那么subscribeNext会打印两个值。否则就只有一个值
     //onScheduler 指定线程
	
####Switching

-switchToLatest 被应用于多个信号中的信号(signal-of-signals),他总会从最新的信号中提出值

	RACSubject *letters = [RACSubject subject];
	RACSubject *numbers = [RACSubject subject];
	RACSubject *signalOfSignals = [RACSubject subject];

	RACSignal *switched = [signalOfSignals switchToLatest];

	// Outputs: A B 1 D
	[switched subscribeNext:^(NSString *x) {
    	NSLog(@"%@", x);
	}];

	[signalOfSignals sendNext:letters];
	[letters sendNext:@"A"];
	[letters sendNext:@"B"];

	[signalOfSignals sendNext:numbers];
	[letters sendNext:@"C"];
	[numbers sendNext:@"1"];

	[signalOfSignals sendNext:letters];
	[numbers sendNext:@"2"];
	[letters sendNext:@"D"];
	
文章参考链接：

 * [ReactiveCocoa and MVVM, an Introduction](http://www.sprynthesis.com/2014/12/06/reactivecocoa-mvvm-introduction/)
 * [ReactiveCocoa for a better world](https://github.com/blog/1107-reactivecocoa-for-a-better-world)
