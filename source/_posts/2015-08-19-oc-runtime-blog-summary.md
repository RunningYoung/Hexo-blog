---
layout: post
title: "收集的Objective-C runtime博客及知识点总结"
description: ""
category: 
tags: [runtime, OC, Objective-C]
author: 窦锦坤
---

#### 这是收集到的一些关于OC runtime的blog。文中的技术要点为各个blog的技术点的结论总结，抛却文中源码和解析过程，不想看原文的可以直接看要点干货。

### 关于category

#### [深入理解Objective-C：Category](http://tech.meituan.com/DiveIntoCategory.html) 技术要点：

#### 一 category是Objective-C 2.0之后添加的语言特性，category的主要作用：
1. 为已经存在的类添加方法
2. 把类的实现分开在几个不同的category文件里面，好处：a)可以减少单个文件的体积 b)可以把不同的功能组织到不同的category里 c)可以由多个开发者共同完成一个类 d)可以按需加载想要的category 等等
3. 声明私有方法
4. 模拟多继承
5. 把framework的私有方法公开


#### 二 与extension对比
- extension 在编译器决议，一般用来隐藏类的私有信息，你必须有一个类的源码才能为一个类添加extension，所以你无法为系统的类 比如NSString添加extension。而category在运行期决议，可以向任何类添加方法。
- extension可以添加实例变量，而category是无法添加实例变量的（因为在运行期，对象的内存布局已经确定，如果添加实例变量就会破坏类的内部布局，这对编译型语言来说是灾难性的）。

#### 三 被category覆盖掉的 类中的original method
- category的方法没有“完全替换掉”原来类已经有的方法，而是将它放到了category的方法的后面。selector与方法的实现IMP是一个map结构的，map中的key就是selector，value就是IMP，可以理解有多个同名方法的map，在消息派发时，一旦匹配到就不在向后查找，所以从表现上来说，category中方法好像覆盖了类中的original method。 正因为这样，我们仍然可以调用到类中的original method，只要顺着方法列表找到最后一个对应名字的方法，就可以调用原来类的方法（这是根据blog中的Code封装的一个方法）：

{% highlight objective-c %}
- (IMP)ed_getOriginalMethodInClass:(Class)targetClass withSelector:(SEL)aSeclector
{
    if (targetClass) {
        unsigned int methodCount;
        Method *methodList = class_copyMethodList(targetClass, &methodCount);
        IMP lastImp = NULL;
        SEL lastSel = NULL;
        NSString *targetSELString = NSStringFromSelector(aSelector);
        for (NSUInteger i = 0; i < methodCount; i++) {
            Method method = methodList[i];
            NSString *methodName = [NSString stringWithCString:sel_getName(method_getName(method)) 
                                        encoding:NSUTF8StringEncoding];
            if ([targetSELString isEqualToString:methodName]) {
                lastImp = method_getImplementation(method);
                lastSel = method_getName(method);
            }
        }
    
        free(methodList);
        
        return lastIMP;
    }
    
    return NULL;
}
{% endhighlight %}

#### 四 category和+load方法
- 类中和category中均可以实现+load方法
- 执行顺序： 先 类中，后 category中，多个category都实现了同名方法，那么会按照编译顺序执行。

小技巧：在Xcode中点击`Edit Scheme`，添加如下两个环境变量（可以在执行load方法以及加载category的时候打印log信息，更多的环境变量选项可参见objc-private.h）:
![设置环境变量](http://tech.meituan.com/img/diveintocategory/environment_vars.png)
![文件结构](http://tech.meituan.com/img/diveintocategory/project.png)
![编译顺序1](http://tech.meituan.com/img/diveintocategory/compile1.png)
![编译顺序2](http://tech.meituan.com/img/diveintocategory/compile2.png)

设置如图一所示的环境变量就可以在启动应用的时候打印加载信息，文件结构如图二所示，分别有图三和图四两种文件顺序，那么编译使调用+load方法的顺序就不同，打印信息就不上了。

#### 五 category和关联对象
- 现在在category里面是无法为category添加实例变量的，虽然从原文blog的源码解析中可以看出apple有在category中实现添加property的意图，但是还没有实现。
- 向category中添加属性，只能通过关联属性来做。举例代码如下：
{% highlight objective-c %}
@implementation MyClass (Category1)

+ (void)load
{
    NSLog(@"%@",@"load in Category1");
}

- (void)setName:(NSString *)name
{
    objc_setAssociatedObject(self,
                             "name",
                             name,
                             OBJC_ASSOCIATION_COPY);
}

- (NSString*)name
{
    NSString *nameObject = objc_getAssociatedObject(self, "name");
    return nameObject;
}

@end
{% endhighlight %}

- 所有的关联对象都由AssociationsManager管理，而在对象的销毁逻辑里面，runtime的销毁对象函数objc_destructInstance里面会判断这个对象有没有关联对象，如果有，会调用_object_remove_assocations做关联对象的清理工作。


### 关于方法缓存

#### [深入理解Objective-C：方法缓存](http://tech.meituan.com/DiveIntoMethodCache.html) 技术要点：

#### 一 方法缓存
- 类class的底层定义都是struct，类的定义里就有cache字段，***类的所有缓存都存在metaclass上，所以每个类都只有一份方法缓存，而不是每一个类的object都保存一份***。
{% highlight objective-c %}
    struct _class_t {
      struct _class_t *isa;
      struct _class_t *superclass;
      void *cache;
      void *vtable;
      struct _class_ro_t *ro;
    };
{% endhighlight %}

- 从父类取到的方法，也会存在类本身的方法缓存里。而当用一个父类对象去调用那个方法的时候，也会在父类的metaclass里缓存一份。
- 类的方法缓存大小没有限制，为了防止快速无限增大，apple的实现方法会使缓存的大小增速慢一点，但是确实是没有上限的（具体可见原文）。
- 为什么类的方法列表不直接做成散列表，而是使用list+单独缓存？

原文给出的可能原因：

- 散列表是没有顺序的，Objective-C的方法列表是一个list，是有顺序的；Objective-C在查找方法的时候会顺着list依次寻找，并且category的方法在原始方法list的前面，需要先被找到，如果直接用hash存方法，方法的顺序就没法保证。
- list的方法还保存了除了selector和imp之外其他很多属性
- 散列表是有空槽的，会浪费空间


### 关于runtime

#### [Objective-C特性：Runtime](http://www.jianshu.com/p/25a319aee33d), 这篇没有技术要点，要自己通读下来。推荐。
