---
layout: post
title: "BDD行为驱动开发-学习指南"
description: ""
category: 
tags: [BDD, test, TDD]
author: 窦锦坤
---


#### 对iOS开发而言测试方式的大概路线是单元测试（XCTest）、TDD、BDD。单元测试就是一系列的断言，网上很多资料，可以看一下，测试大型项目或者复杂逻辑时，力不从心。TDD（测试驱动开发）是敏捷开发的核心实践，基本思路就是通过测试来推动整个开发的进行，先把需求分析，设计，质量控制量化，根据需求排除模棱两可的需求，编写测试用例代码，然后根据测试用例编写产品代码，这就是所谓的TDD。

#### 但是当我们使用TDD时，马上会出现一个问题，我要测试什么，只知道TDD测试驱动开发，却不知道测试什么。而BDD（行为驱动测试）从***名字***上就明确的告诉了我们要测试什么——行为。

#### ***什么叫行为：一个类对象的接口定义了其方法和依赖关系。这些方法和依赖关系决定了类对象如何与应用的其他部分交互 以及该类对象的功能，这些就是对象的行为。***

#### 那么“行为”怎么理解？
 
我的理解（纯粹是一家之言，每个人的理解不一样）：

- 第一步，需求分析，先明确要实现的类在业务需求上要实现的功能。依据“功能”，我们就能确定这个类的接口定义和依赖关系，这样我们就明确了该类的行为。
- 开发者在编写测试代码的时候，要明确自己的定位，开发者是观察者，而类对象是被观察者，开发者要观察类对象的行为，这个行为包含两个方面：非互动行为 和 互动行为。非互动行为 就是不需要观察者与之互动就可以观察到的行为，可以理解为对象对自己施加的行为（可以理解为我们看到的它的长相，这是他自己打扮的结果）；互动行为 就是观察者给被观察者一个信号，被观察者回复一个应答。 以UI控件的测试为例，先测试非互动行为，这个UI控件应该长成什么样子（即根据需求设计的UI，如title color等）；然后是互动行为（如Button被点击的反应等）。
- 行为定义包括两个方面，一是 接口定义的方法，二是 与其他类或者部分的依赖关系。测试的时候不要漏掉对依赖关系的测试。一些必须要处理的*操作*，但是属于内部实现，在行为上体现不出来，应该怎么避免处理上的遗漏？这就是属于依赖关系，因为这些*操作*在当前的测试对象的行为上体现不出来，但是其他地方肯定要依赖这些*操作*（但这似乎又和不要测试内部实现有点矛盾）。 

***

#### 在iOS开发中，可用的BDD测试框架

OC版本的BDD框架： 基于宏定义和block实现

- [Specta](https://github.com/specta/specta)
- [Kiwi](https://github.com/kiwi-bdd/Kiwi/wiki)及其[wiki](https://github.com/kiwi-bdd/Kiwi/wiki)
- [Cedar](https://github.com/pivotal/cedar) 

swift版本的BDD框架

- [Sleipnir](https://github.com/railsware/Sleipnir) 
- [Quick](https://github.com/Quick/Quick)

##### 在这三个OC的框架中，前两个最受欢迎，而从github上的star数量来说，`Kiwi`大约是`Specta`的两倍，从侧面说明了使用`Kiwi`的人最多。但是也有好多反应`Specta`比`Kiwi`好用的，这主要是看个人的习惯和喜好。他们都是对XCTest的封装，所以不能同时集成到项目中使用。这两者简单使用过程来说，最直观的区别就是`Kiwi`的书写格式是消息发送，而`Specta`是点语法的链式表达。

`Kiwi`与`Specta`的主要区别：

- `Specta`由github的`RAC`的那帮人维护，使用较多的黑魔法，一旦apple改变了测试的底层实现，可能会出现很多问题
- 功能上来说，[`Kiwi`功能要多一些](http://blog.csdn.net/u012496940/article/details/47404359)，`Specta`就是没有`mock`和验证功能的`Kiwi`。
- 集成的方便程度，`Kiwi`只需要使用Pod集成一个`Kiwi`，而`Specta`还需要其他的三方库支持，如`Expecta`，因为`Specta`是轻量级的。
- 使用的方便程度，这个因人而异，前者是OC的消息发送，后者是点语法的链式表达，个人倾向于点语法，`Specta`的expectation语法有一个比Kiwi好的地方：每个变量都隐式boxing 如 expect(items.count).to.*equal(5)*。不需要像Kiwi那样将5包装成NSNumber，*theValue(5)*，`Specta`和Expecta搭配使用效果更好。

***

#### 以`Kiwi`为例，学习BDD的过程

喵神的两篇文章，从测试的基础讲起，XCTest到TDD再到BDD再到`Kiwi`的学习使用，先学习这两篇，基本上就了解了BDD的`Kiwi`测试

- [TDD的iOS开发初步以及Kiwi使用入门](http://onevcat.com/2014/02/ios-test-with-kiwi/)
- [Kiwi 使用进阶 Mock, Stub, 参数捕获和异步测试](http://onevcat.com/2014/05/kiwi-mock-stub-test/)

接下来这一篇blog，很有价值，以为它用一个实际项目讲解了 `RAC` `MVVM`和`Kiwi`在项目中的实际使用。`RAC`和`MVVM`是我们下一步要用到的框架。

- [行为驱动开发iOS](http://www.jianshu.com/p/73f9d719cee4)

看完这三篇基本上就了解BDD，还有如下几篇可以看一下：

- 使用`Specta`的[Getting Started With BDD With Calabash and Specta (Part 1)](http://www.developerdave.co.uk/2015/05/getting-started-with-bdd-with-calabash-and-specta-part-1/)
- [ios测试框架的理解](http://blog.csdn.net/u012496940/article/details/47404359)
- [关于TDD、BDD和DDD的一些看法](http://www.cnblogs.com/wangshenhe/archive/2013/02/16/2913431.html)
- [行为驱动开发](http://objccn.io/issue-15-1/)

****

#### 行为驱动开发大概三个步骤，这三个步骤要循环往复：

- 选择最重要的行为，并编写行为的测试文件。此时，由于测试对象的类还没编写，所以编译失败。创建测试对象的类并编写类的伪实现，让编译通过。
- 实现被测试类的行为，让测试通过。
- 如果发现代码中有重复代码，重构被测试类来消除重复

***

#### 测试中的原则，讲几个重要的，可测试 可重用之类的是必须的

- 注重测试对象的行为，而不是内部实现，就是只测试接口和依赖关系，不测试内部实现
- 单一性，一个测试文件只测试一个类
- 小步前进，不要一次写好多测试代码，一起去实现业务，在一起测试
- 及时重构，对结构不合理或者重复的代码，在测试通过后，应及时进行重构，并再次测试

***

#### 学习BDD时遇到的问题，大家在学习的时候可以思考一下，提供一下思路

##### 知乎上有个问题[TDD 与 BDD 仅仅是语言描述上的区别么？](http://www.zhihu.com/question/20161970)的一个回答如下：

##### BDD的核心价值是体现在正确的对系统行为进行设计，所以它并非一种行之有效的测试方法。它强调的是系统最终的实现与用户期望的行为是一致的、验证代码实现是否符合设计目标。但是它本身并不强调对系统功能、性能以及边界值等的健全性做保证，无法像完整的测试一样发现系统的各种问题。但BDD倡导的用简洁的自然语言描述系统行为的理念，可以明确的根据设计产生测试，并保障测试用例的质量。

##### 这个回答提到的，也是我在学习BDD过程中遇到的问题，BDD怎么去测试我们代码的性能和边界问题？
边界测试可以反复的写大量的`then`(BDD的测试都是在讲一个故事，三段式的，`Given..When..Then`结构，看了前面的提供的blog就懂了)覆盖可能的各种边界？有没有更有效的方法，能在我们修改这个被测试的代码后 仍然能使用原先的测试代码进行测试；或者能驱动我们能够覆盖到各种可能边界。 至于性能测试，BDD是不是直接不能做，还要使用其他的手段。










