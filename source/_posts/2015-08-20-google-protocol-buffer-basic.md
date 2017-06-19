---
layout: post
title: "Google Protocol Buffers总结-基础篇"
description: ""
category: 
tags: [protobuf，protocol buffers]
author: 窦锦坤
---


### protobuf

#### Protocol Buffers简称protobuf或者pb，是由Google开发维护的夸语言、跨平台的 可扩展的结构化数据串行机制，官方地址[Protocol Buffers](https://developers.google.com/protocol-buffers/)。

#### 现在常用的网络数据传输的结构化方式有：XML、JSON和Protobuf等。有很多篇介绍他们的对比和不同：blog[protobuf,json,xml,binary,Thrift之间的对比](http://blog.sina.com.cn/s/blog_406127500102uy6e.html) 和 [使用 Protocol Buffers 代替 JSON 的五个原因](http://www.oschina.net/translate/choose-protocol-buffers)。网上还有很多其他对比的帖子。不过简言之，protobuf的优缺点如下：
- 效率高：用protobuf序列化后的数据大小是json的10+分之一，xml格式的20+分之一，是二进制序列化的10+分之一。protobuf虽然会需要一定的本地编解码成本，但是数据压缩体积变小，网络传输速度加快，在效率和体验上的提升非常可观。
- 语法简答易学：保证做一次就会。
- 夸语言、跨平台、可扩展：不收平台的限制，并且无偿向后兼容。
- 缺点：唯一的缺点就是可读性差，因为数据都是编码后的，不像XML和JSON可读。

********

### 安装

#### Google的官方版本，只提供了JAVA C++ C 和 Python版本的pb，但是在github的开源项目上有OC和最新的SWIFT语言的版本。git还在维护的版本有[qzix/protobuf-objc](https://github.com/qzix/protobuf-objc) 和 [alexeyxo/protobuf-objc](https://github.com/alexeyxo/protobuf-objc)，其中后者还[支持swift](http://protobuf.io/#swift)，推荐使用后者，同时安装步骤也在该页。借助homebrew安装更方便，如果已安转homebrew请跳过第一步，摘录如下：

做操作以前先获取系统权限sudo一下

1. `ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"`
2. `brew install automake`
3. `brew install libtool`
4. `brew install protobuf`
5. `ln -s /usr/local/Cellar/protobuf/2.6.0/bin/protoc /usr/local/bin (optional)`
6. `git clone git@github.com:alexeyxo/protobuf-objc.git`
7. `./build.sh`
8. `Add /src/runtime/ProtocolBuffers.xcodeproj in your project.`

******

### 使用-编写`.proto`源文件

#### 使用pb的第一步就是编写`.proto`源文件，使用任何一款自己习惯的编辑器即可。对应OC中的类，`protobuf`称为`message`消息，其实整个语法结构，细想下来与常见语言的语法结构差不多，所以很容易学。举例代码入下：
{% highlight objective-c %}
message Person {
    required int32 age = 1;
    repeated string name = 2;
    optional string email = 3;
}
{% endhighlight %}

#### `.proto`源文件格式从上面的源码中可以猜测一个大概，举例并解析如下：
{% highlight objective-c %}
message msgName(消息名称) {
    required/repeated/optional(限定符) dataType(数据类型) variableName(变量名) =(赋值号) serialNumber(序号，必须从1开始，不能跳跃);(结束的分号)
}
{% endhighlight %}

#### ***关于msgName***
##### `.proto`中可以有多个message，dateType也可以是自定义message，但是源文件最好与主message同名，即`mainMsgName.proto`,否则在使用的时候会出现一些名称不一致的问题，虽然仍能影响使用，但是会增加使用难度。

##### tips：一些公共的协议类的`.proto`源文件，可以独立成一个文件，在其他使用到的该消息的文件的开头位置用import引入即可，如`import "myproject/CommonMessages.proto"`

#### ***关于required/repeated/optional(限定符)***
1. 在每个消息中必须至少留有一个required类型的字段（在实际使用中，可以全部是optional的），这样的变量在相应类的初始化时必须初始化，否则报错。 
2. 每个消息中可以包含0个或多个optional类型的字段，从名字可以看出，可选的，可以不初始化或者不使用该字段。推荐尽可能使用optional限定符，即使协议文档规定为required，两者仍可以兼容，并且避免以后协议修改造成的重写和重新编译。
3. repeated表示的字段可以包含0个或多个数据。需要说明的是，这一点有别于C++/Java中的数组，因为后两者中的数组必须包含至少一个元素，可以理解为repeated也是一种可选类型。
4. 如果打算在原有消息协议中添加新的字段，同时还要保证老版本的程序能够正常读取或写入，那么对于新添加的字段必须是optional或repeated。道理非常简单，老版本程序无法读取或写入新增的required限定符的字段。

#### ***关于dataType(数据类型)***

#### 这个很简单，数据类型名字也很好记，有表格如下：
<table style="width: 760px; height: 323px;" border="0" align="center">
<tbody>
<tr>
<td style="text-align: center;"><span style="color: #0000ff;"><strong><span style="font-size: 15px;">.proto Type</span></strong></span></td>
<td style="text-align: center;"><span style="color: #0000ff;"><strong><span style="font-size: 15px;">Notes</span></strong></span></td>
<td style="text-align: center;"><span style="color: #0000ff;"><strong><span style="font-size: 15px;">C++ Type</span></strong></span></td>
<td style="text-align: center;"><span style="color: #0000ff;"><strong><span style="font-size: 15px;">Java Type</span></strong></span></td>
</tr>
<tr>
<td style="text-align: center;"><span style="font-size: 15px;">double</span></td>
<td>&nbsp;</td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;double</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;double</span></td>
</tr>
<tr>
<td style="text-align: center;"><span style="font-size: 15px;">float</span></td>
<td>&nbsp;</td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;float</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;float</span></td>
</tr>
<tr>
<td style="text-align: center;"><span style="font-size: 15px;">int32</span></td>
<td><span style="font-size: 15px;">Uses variable-length encoding. Inefficient for encoding negative numbers &ndash; if your field is likely to have negative values, use sint32 instead.</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;int32</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;int</span></td>
</tr>
<tr>
<td style="text-align: center;"><span style="font-size: 15px;">int64</span></td>
<td><span style="font-size: 15px;">Uses variable-length encoding. Inefficient for encoding negative numbers &ndash; if your field is likely to have negative values, use sint64 instead.</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;int64</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;long</span></td>
</tr>
<tr>
<td style="text-align: center;"><span style="font-size: 15px;">uint32</span></td>
<td><span style="font-size: 15px;">Uses variable-length encoding.</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;uint32</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;int</span></td>
</tr>
<tr>
<td style="text-align: center;"><span style="font-size: 15px;">uint64</span></td>
<td><span style="font-size: 15px;">Uses variable-length encoding.</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;uint64</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;long</span></td>
</tr>
<tr>
<td style="text-align: center;"><span style="font-size: 15px;">sint32</span></td>
<td><span style="font-size: 15px;">Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int32s.</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;int32</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;int</span></td>
</tr>
<tr>
<td style="text-align: center;"><span style="font-size: 15px;">sint64</span></td>
<td><span style="font-size: 15px;">Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int64s.&nbsp;</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;int64</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;long</span></td>
</tr>
<tr>
<td style="text-align: center;"><span style="font-size: 15px;">fixed32</span></td>
<td><span style="font-size: 15px;">Always four bytes. More efficient than uint32 if values are often greater than 2<sup>28</sup>.&nbsp;</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;uint32</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;int</span></td>
</tr>
<tr>
<td style="text-align: center;"><span style="font-size: 15px;">fixed64</span></td>
<td><span style="font-size: 15px;">Always eight bytes. More efficient than uint64 if values are often greater than 2<sup>56</sup>.</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;uint64</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;long</span></td>
</tr>
<tr>
<td style="text-align: center;"><span style="font-size: 15px;">sfixed32</span></td>
<td><span style="font-size: 15px;">Always four bytes.</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;int32</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;int</span></td>
</tr>
<tr>
<td style="text-align: center;"><span style="font-size: 15px;">sfixed64</span></td>
<td><span style="font-size: 15px;">Always eight bytes.</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;int64</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;long</span></td>
</tr>
<tr>
<td style="text-align: center;"><span style="font-size: 15px;">bool</span></td>
<td><span style="font-size: 15px;">&nbsp;</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;bool</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;boolean</span></td>
</tr>
<tr>
<td style="text-align: center;"><span style="font-size: 15px;">string</span></td>
<td><span style="font-size: 15px;">A string must always contain UTF-8 encoded or 7-bit ASCII text.</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;string</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">&nbsp;String</span></td>
</tr>
<tr>
<td style="text-align: center;"><span style="font-size: 15px;">bytes</span></td>
<td><span style="font-size: 15px;">May contain any arbitrary sequence of bytes.</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">string</span></td>
<td style="text-align: center;"><span style="font-size: 15px;">ByteString</span></td>
</tr>
</tbody>
</table>

#### pb中的数据类型参考C++版本使用即可，除此之外pb还支持枚举类型的数据定义，举例如下：
{% highlight objective-c %}
enum UserStatus {
     OFFLINE = 0;
     ONLINE = 1;
}
message UserInfo {
    required int64 acctID = 1;
    required string name = 2;
    required UserStatus status = 3;
}
message LogonRespMessage {
    required LoginResult logonResult = 1;
    required UserInfo userInfo = 2;
}
{% endhighlight %}

*******

### 使用-编译`.proto`源文件

#### 完成源文件的编写之后，就是要编译该源文件，编译的环境和命令在安装时就已经配置好了，我们只需要调用指令编译即可。编译得到的结果对于OC来说会生成一对`.h .m`文件，即一个类，类名即为`.proto`文件的名字；对于swift语言因该是生成一个`.swift`文件。编译指令以如下：

#### `protoc .proto源文件路径 --objc_out="输出文件路径"`

#### `protoc .proto源文件路径 --swift_out="输出文件路径"`

***

### 使用-项目中引用

#### 将上一步生成的目标文件引入到项目中，就可以当做普通的类来使用了。

#### tips：不同的oc版本的pb的开源项目略有不同，不过大同小异，如类名的命名方式等，一个较大的区别是有的支持ARC，有的不支持ARC 需要禁用，所以推荐用cocopods引入。

#### 实际使用举例，以前面的Person.proto为例，*常规*的使用步骤如下（当然要导入必要的头文件）：
{% highlight objective-c %}
- (void)viewDidLoad 
{
    // 串行化一个,Person结构的数据
    NSData *personData = [self makePersonDataWithAge:21
                                               names:@[@"测试", @"test"]
                                               email:@"test@zuche.com"];
    NSLog(@"data : %@", personData); // 用于数据传输
    
    // 反串行化得到的串行化数据
    Person *aPerson = [Person parseFromData:personData];
    NSLog(@"person age : %d\n person names : %@\n person email : %@\n", aPerson.age, aPerson.names, aPerson.email);  // 接收到的数据的还原
}


- (NSData *)makePersonDataWithAge:(int32_t)age names:(NSArray *)names email:(NSString *)email
{
    PersonBuilder *builder = [[PersonBuilder alloc] init]; // 有的版本是这样的Person_Builder
    builder.age = age;
    builder.email = email;
    [builder setNamesArray:names];
    
    Person *args = [builder build];
    
    return [args data];
}
{% endhighlight %}

### 总结

#### 这只是pb使用的基础用法，看源码能得到很多收获。pb的精髓在于其编码方式，有兴趣的可以看一下。这一篇只是讲了pb的基本使用，再其使用过程中，还有很多需要注意的点，这一篇中就不讲了。

> 补充延伸阅读
>
> [Google Protocol Buffers 概述](http://shitouer.cn/2013/04/google-protocol-buffers-overview/)
>
> [Protobuf语言指南](http://www.cnblogs.com/dkblog/archive/2012/03/27/2419010.html)
>
> [Google Protocol Buffers 编码(Encoding)](http://www.cnblogs.com/shitouer/archive/2013/04/12/3017381.html)
> 







