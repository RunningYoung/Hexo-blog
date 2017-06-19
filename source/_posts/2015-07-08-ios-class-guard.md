---
layout: post
title: "iOS Class Guard github译文及使用经验总结"
description: "iOS应用的加密混淆"
category: blog
tags: [ios, security, obfuscate]
author: 窦锦坤
---

### [iOS Class Guard](https://github.com/Polidea/ios-class-guard#pro-version) github 译文

#### iOS-Class-Guard是一个用于OC类、协议、属性和方法名混淆的命令行工具。它是class-dump的扩展。这个工具会生成一个symbol table，这个table在编译期间会包含进工程中。iOS-Class-Guard能有效的隐藏绝大多数的类、协议、方法、属性和 实例变量 名。

#### iOS-Class-Guard不是应用安全的最终解决方案，但是它绝对能让攻击者更难读懂你的程序。

***

#### iOS-Class-Guard会加大代码分析和runtime检查的难度，这个工具可以认为是一个简单基础的混淆方法。由于OC的架构决定了iOS应用程序的剖析相当简单，check out一下链接就知晓了：

- [http://www.infointox.net/?p=123](http://www.infointox.net/?p=123)

- [http://www.cycript.org/](http://www.cycript.org/)

- [http://resources.infosecinstitute.com/ios-application-security-part-2-getting-class-information-of-ios-apps/](http://resources.infosecinstitute.com/ios-application-security-part-2-getting-class-information-of-ios-apps/)

- [http://timourrashed.com/decrypting-ios-app/](http://timourrashed.com/decrypting-ios-app/)

***

#### 工作原理

##### 这个工具只对应用程序的编译版本起作用（工具的脚本文件会首先编译项目源码，得到应用文件，之后使用class-dump处理应用文件）。它会读取Mach—O对象文件的OC部分（工具只对mach-o和fat类型的文件有用，如果是想混淆自定义的静态，需要稍微转换一下策略），并解析其中所有的类、属性、方法、实例变量，之后添加所有的symbols到列表中。然后它会读取所有的依赖框架，并做相同的解析OC代码结构的处理，不同的是，此时是把symbol添加到禁止列表中。之后 ***所有的并且不在禁止列表中***的symbols会被混淆处理。每一个symbol由随机生成的 子母和数字 组成。每次执行混淆操作，都会生成一个唯一的symbol map。之后这个map会格式化成一个C的宏定义 头文件，并包含到 .pch文件中。 然后，它会找出XIB和storyboard并更新里面的名字（即IB文件也会被有效的混淆掉）。 这个工具还会查找工程内的xcdatamodel文件并添加其中的类和属性名到禁止列表。 在编译期间内，所有定义在头文件内的symbol都会用对应的生成的不同的符号替换并编译。

##### iOS-Class-Guard也提供了对cocoapod库的混淆。这个工具会 根据用户提供的pods路径 自动遍历所有列出的target 并 查找 .xcconfig文件和要修改的预编译头文件路径。然后添加预先生成的头文件到库 .pch头文件，并更新target的.xcconfig文件中的头文件的search path参数。

##### iOS-Class-Guard还会生成一个json格式的symbol映射。这个映射可以用来处理crash报告是的逆向处理。注意 iOS-Class-Guard不混淆system symbol，所有如果在自定义类中的某些属性和方法与system symbol有相同的名字，则不会被混淆。

***

#### 安装

##### 如果没有安装`brew` 先安装之，在终端内执行这 `curl -LsSf http://github.com/mxcl/homebrew/tarball/master | sudo tar xvz -C/usr/local --strip 1`，若已经安装则跳过，直接执行 `sudo brew install https://raw.githubusercontent.com/Polidea/homebrew/ios-class-guard/Library/Formula/ios-class-guard.rb`. 工具的安装目录为`/usr/local/bin`。若刚想安装最新的版本可执行`brew install --HEAD https://raw.githubusercontent.com/Polidea/homebrew/ios-class-guard/Library/Formula/ios-class-guard.rb`

***

#### 用法

##### 集成iOS-Class-Guard到项目中需要以下几步：
1. 下载 `obfuscate_project` 到工程的根目录。 终端内执行 `curl -o obfuscate_project https://raw.githubusercontent.com/Polidea/ios-class-guard/master/contrib/obfuscate_project && chmod +x obfuscate_project`
2. 更新 `obfuscate_project` 内的project file、scheme 和 configuration name
3. 执行 `bash obfuscate_project`。每一次release都应该执行一次该操作。 ***保存包含symbol映射的json文件 以便于在crash时能逆向得到原来的symbol***
4. 用Xcode或其他工具 Build、test archive工程

##### 上面是基本步骤，也可以添加 额外的target，这些target会在编译期间自动 重新生成symbol map。

***
***

#### 命令行选项

`ios-class-guard 0.7 (64 bit)

Usage: ios-class-guard [options] <mach-o-file>

  where options are:
  
        -F <class>     specify class filter for symbols obfuscator (also protocol))
        
        -i <symbol>    ignore obfuscation of specific symbol)
        
        --arch <arch>  choose a specific architecture from a universal binary (ppc, ppc64, i386, x86_64, armv6, armv7, armv7s, arm64)
        
        --list-arches  list the arches in the file, then exit
        
        --sdk-ios      specify iOS SDK version (will look for /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS<version>.sdk  or /Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS<version>.sdk)
        
        --sdk-mac      specify Mac OS X version (will look for /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX<version>.sdk  or /Developer/SDKs/MacOSX<version>.sdk)
        
        --sdk-root     specify the full SDK root path (or use --sdk-ios/--sdk-mac for a shortcut)
        
        -X <directory> base directory for XIB, storyboards (will be searched recursively)
        
        -P <path>      path to project.pbxproj of Pods project (located inside Pods.xcodeproj)
        
        -O <path>      path to file where obfuscated symbols are written
        
        -m <path>      path to symbol file map (default value symbols.json)
        
        -c <path>      path to symbolicated crash dump`
        
#### 要熟悉以下几个命令

***

#### 输出头文件路径

##### ios-class-guard输出头文件路径，使用 `-O`参数， 如 `-O SWTableView/symbols.h`

***

#### class过滤

##### ios-class-guard能够过滤出 不希望混淆的类。比如，预编译的静态库。 iOS编码风格假定每个类都使用2-3个字符的前缀，可以利用这一点过滤进 或者 过滤出整个命名空间。举例，过滤出所有*APH* *MC*的命名空间 `-F '!APH*' -F '!MC*'`

***

#### 忽略symbol

##### 有些情况是我们不希望混淆，但是一些symbol仍然被混淆了，比如，使用C函数 且 OC的方法也使用了相同的名字，这会导致一个***ld 连接错误（unresolved external）***。此时，必须要找出该symbol 添加到忽略symbol list中。 举例，不混淆名为*defalte* 和 以*curl_\**开头的symbol `-i 'deflate' -i 'curl_*'`

***
***

#### CocoaPods

##### 如果项目中使用了CocoaPods，也可以混淆这些 外部库内的symbol。用户需要做的是指定Pods PBX工程文件的路径。它在.xcodeproj目录内。ios-class-guard会修改配置和预编译头文件，这样pod内的库也可以被混淆了。 用法 `-P Pods/Pods.xcodeproj/project.pbxproj`

***
***

#### 其他选线

#### Xib目录

##### 这是一个可选项。工具默认会从可执行目录（绝大多数情况下是工程的根目录）递归的搜索所有的 *XIB/Storyboard*文件。如果 *XIB/Storyboard*文件存储在其他路径，用法如下 `-X SWTableView/Xib`

***

#### symbol映射文件

##### 可以指定工具保存symbol映射的路径，默认名为 symbols.json。 用法 `-m release/symbols_1.0.0.json`

***

#### 逆向crash dump中的混淆

##### iOS Class Guard支持对自动崩溃报告工具的逆向处理，如*Crashlytics, Fabric, BugSense/Splunk Mint, Crittercism or HockeyApp*。使用`--dsym`参数，iOS Class Guard会替换提供的*dSYM*文件内的原符号和混淆符号。强烈推荐 在*Build Phases/Run script*
一开始 添加如下所示的脚本来完成*dSYM*的自动转换处理，该功能已在上述的工具中测试通过。

`if [ -f "$PROJECT_DIR/symbols.json" ]; then
/usr/local/bin/ios-class-guard -m $PROJECT_DIR/symbols.json --dsym $DWARF_DSYM_FOLDER_PATH/$DWARF_DSYM_FILE_NAME --dsym-out $DWARF_DSYM_FOLDER_PATH/$DWARF_DSYM_FILE_NAME
fi

\# Another invocations eg.: ./Crashlytics.framework/run <Crashlytics secret #1> <Crashlytics secret #2>`

##### 手动使用方法如下 `ios-class-guard -m symbols.json --dsym MyProject_obfuscated.app.dSYM --dsym-out MyProject_unobfuscated.app.dSYM`

***
***

#### 局限性，OC的工作方式决定了这个工具的局限性

#### XIB and Storyboards

##### ios-class-guard处理*XIB 和 Storyboard*文件的效果很好，但是当使用外部库，且库内包含了IB文件的bundle，***一定要忽略***这些symbol，否则你在启动app时，他们不会再有效。处理这种情况就要使用class filter了。

***

#### KVO

##### 使用混淆可能会导致KVO停止工作。大部分开发者使用硬编码字符串指定KeyPath。

{% highlight objective-c %}
- (void)registerObserver {
    [self.otherObject addObserver:self
                       forKeyPath:@"isFinished"
                          options:NSKeyValueObservingOptionNew
                          context:nil];
}

- (void)unregisterObserver {
    [otherObject removeObserver:self
                     forKeyPath:@"isFinished"
                        context:nil];
}

- (void)observeValueForKeyPath:(NSString *)keyPath
              ofObject:(id)object
                change:(NSDictionary *)change
               context:(void *)context
{
  if ([keyPath isEqualToString:@"isFinished"]) {
    // ...
  }
}
{% endhighlight %}

##### 这样写无效，属性*isFinished*将会被重新命名，而硬编码字符串不会反映这种变化。 处理方法是 移除全部的*KeyPath* 并 改为 `NSStringFromSelector(@selector(keyPath))`。
##### **修改后的代码如下**

{% highlight objective-c %}
- (void)registerObserver {
    [self.otherObject addObserver:self
                       forKeyPath:NSStringFromSelector(@selector(isFinished))
                          options:NSKeyValueObservingOptionNew
                          context:nil];
}

- (void)unregisterObserver {
    [otherObject removeObserver:self
                     forKeyPath:NSStringFromSelector(@selector(isFinished))
                        context:nil];
}

- (void)observeValueForKeyPath:(NSString *)keyPath
              ofObject:(id)object
                change:(NSDictionary *)change
               context:(void *)context
{
  if ([keyPath isEqualToString:NSStringFromSelector(@selector(isFinished))]) {
    // ...
  }
}
{% endhighlight %}

***

#### **串行化**

##### 如果使用保存在磁盘上的类 或者 用户默认使用了`NSCoding`协议，那么就不能混淆他们了。否则，再次生成symbol后，APP在启动时会crash，不能从串行化数据中读取该类。

***

#### 未定义symbol

##### 使用iOS-Class-Guard时，很可能会遇到类似与下面的问题：
`Undefined symbols for architecture i386:
  "_OBJC_CLASS_$_n9z", referenced from:
      objc-class-ref in GRAppDelegate.o`

##### 解决方法，复制`n9z`并在`symbols.h`中查找，`n9z`很可能是个类，就要把它从混淆中排除，使用`-F '!UnresolvedClassName'`参数并重新测试。

***

#### note

##### iOS-Class-Guard与[LLVM Obfuscator](https://github.com/obfuscator-llvm/obfuscator)一起使用还没有经过测试。

***

#### Pro版

##### pro版包含更多的功能：
- Encryption of strings and constants
- Tamper detection mechanism
- Anti-debug mechanism
- Methods inlining
- Assets encryption
- Control flow obfuscation
- Code virtualization with encryption
- API method execution hiding


