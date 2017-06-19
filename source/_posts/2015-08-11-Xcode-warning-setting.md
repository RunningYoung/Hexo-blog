---
layout: post
title: Xcode中的warning设置
description: ""
category: 
tags: [Xcode, warning, setting]
author: 窦锦坤
---

### Xcode中的warning警告设置

#### 作为一名有节操的程序猿，应该有严谨的coding态度，写文章也不能说屎上最牛。Xcode的warning设置，能给我们很好的帮助，强制约束我们严谨coding：0 warning 0 error。下面直接上干货。

***
***

#### 扯外篇，本blog来自[这里](http://boredzo.org/blog/archives/2009-11-07/warnings)并与时俱进了一下下。 以下几条在常规的iOS开发中，几乎没用，所以关闭。
 - ###### 没用的，如 “‘Effective C++’“ Violations 等，不写C++代码的话，用不到，所以关闭。
 
 - ###### 没有帮助的，如 “Four Character Literals” 等，这个选项是涉及四字节编码类型 如OSType，弊大于利，所以关闭。
 
 - ##### 不可修复的，如 “Multiple Definition Types for Selector” 等，Cocoa抛出这类警告会遍布它所有的头文件，我们不能fix这种情况。
 
***
***
 
#### 正篇，我们开启一下警告，会使我们的项目更清爽，并帮我们排除潜在的bug。
***
***
 
#### ***Check Switch Statements***
 
##### `当一个switch语句的index索引值为一个枚举类型时，如果这时缺少了一个或者几个枚举值，会抛出此类警告。`
 
##### solution：使用default语句可以fix it。 但是在编程规范我中，我们处理枚举类型的switch，我们推荐**不使用**default。 这样可以提示我们漏掉的情况，并且以后对枚举值做扩展时，也会提示我们处理新的枚举值。所以推荐的解决方案是：写全每个枚举值的处理，并且*不使用*default。
***
 
#### ***Hidden Local Variables***
 
##### `当一个局部变量shadow另外一局部变量、参数、全局变量或者内建的函数名时，会抛出次警告。`
 
##### solution：命名时注意规范，这种情况，局部变量会屏蔽掉上一级的同名变量，这可能不是我们想要的。还有一种情况就是命名一个变量名为index，index还是标准C库函数，这种代码`myArray[index]`不要不要的。一旦index变量声明失败，运行结果就全凌乱了。so，never name a variable *index*。
***
 
#### ***Implicit Conversion to 32 Bit Type***
 
##### `当一个数据值隐式地从64位转换为32位时，会抛出此警告`
 
##### solution：当想要将一份旧代码转到一个64位的平台上并正确运行，开启这警告很有用。它会提示我们潜在的指针错误。
***
 
#### ***Initializer Not Fully Bracketed***
 
##### `当初始化不对等时，会抛出这个警告。`
{% highlight objective-c %}
   int a[2][2] = { 0, 1, 2, 3 }; // wrong way
   
   int b[2][2] = { { 0, 1 }, { 2, 3 } }; // right way
{% endhighlight %}
 
##### solution：这个警告也适用于结构体，如NSRect：
{% highlight objective-c %}
   NSRect warns = { 0.0f, 0.0f, 640.0f, 480.0f };
 
   NSRect doesNotWarn = { {0.0f, 0.0f}, {640.0f, 480.0f} };
{% endhighlight %}
 
##### tips：推荐使用`NSZeroPoint`替代` {0.0f, 0.0f}`
***
 
#### ***Mismatched Return Type***
 
##### `当返回值类型与函数声明的返回值类型不匹配时，会抛出警告`
 
##### solution：没啥好说的
***
 
#### ***Missing Braces and Parentheses***
 
##### `当缺少必要的大括号或者圆括号时，会使代码结构和层级混乱，会抛出此警告`
 
##### solution：使用if else一定要加大括号，不要吝啬使用大括号和圆括号。
***
 
#### ***Missing Fields in Structure Initializers***
 
##### `当结构体初始化 缺少了若干域时，会抛出此警告`
 
##### solution：举例说明：
 
{% highlight objective-c %}
    struct s { int f, g, h; }; // bad way
    struct s x = { 3, 4 }; 

    struct s { int f, g, h; };  // right way
    struct s x = { .f = 3, .g = 4 }; // or
    struct s x = {3, 4, 5};
{% endhighlight %}
***

#### ***Missing Newline At End Of File***

##### `与diff文件的清洁度有关，在编译时分隔不同的头文件。`

##### solution：每个头文件末尾加上空行。
***

#### ***Sign Comparison***

##### `当符号数和无符号数直接比较时，可能会出现错误的结果。`

##### solution：避免这种情况，不能避免时，把符号书转化成无符号数。
***

#### ***Typecheck Calls to printf/scanf***	  

##### `打开这个警告会检查printf/scanf的入参合法性，如NSArray *array = [NSArray arrayWithObjects:@"foo", @"bar"]; 就会抛出警告。`

##### solution：没啥好说的，开启这个警告 printf、NSLog 和 stringWithFormat都会检查入参。
***

#### ***Undeclared Selector***

##### `当"@selector(...)"表达式引用一个未声明的selector时，会抛出此异常。 未声明就是指没有对应的该方法名 在@interface 或 @protocol中显式声明 或者在 @implementation中隐式声明。`

##### solution：没啥好说的，开启这个警告可以避免KVC, KVO, KVV 或者 Bindings method传入一个错误的key。
***

#### ***Unused Functions***

##### `当一个静态函数只有声明 没有定义 或者 一个non-inline静态函数没有被使用时，会抛出这个警告。`

##### solution：建议将模块内的 对外不可见的函数声明为static。
***

#### ***Unused Labels***

#### ***Unused Values***

#### ***Unused Variables***

##### 以上三条遵循规范原则：“code you don't have is code you don't have to debug”，可以使代码简洁。
***

#### ***Treat Warnings as Errors***

##### 警告意味着潜在的bug，开启这个code的困难模式，能有效的降低这个风险，实在不能避免的warning，使用#pragma吧。
***

#### ***Analyze during 'build'***

##### 在build时打开静态分析，Clang Static Analyzer能查找bug，记得要排除静态分析的警告。除非某些文件分析的时间很长，会影响编译和开发速度，否则默认打开这个选项。
***


