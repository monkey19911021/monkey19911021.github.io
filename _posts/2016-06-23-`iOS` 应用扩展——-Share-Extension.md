---
permalink: /2016/06/23/ShareExtension
header:
  teaser: "ShareExtension/img1.PNG"
categories:
  - Objective-C 
  - Extension
tags:
  - ShareExtension
  - Extension
---
![截图1]({{ site.url }}/images/ShareExtension/img1.PNG)

## 1. 先来介绍几个概念
**App Extension：** 扩展

**Containing app：** 提供扩展的应用称之为容器应用

**Host app：** 调用扩展的应用称之为载体应用

容器应用可以同时是载体应用，即容器容器应用也可以调用自身的扩展。
<br>
![截图6]({{ site.url }}/images/ShareExtension/img6.png)
<br>

## 2. 创建分享扩展
关于如何创建一个扩展请参考[这里](http://mkapple.cn/2016/06/20/TodayExtension)的第二步骤, 我们要创建的是 Share Extension 。
<br>	
P.S. 说实话，分享扩展这东西除了在社交应用或者一些特别应用之外没啥用处了
<br>
![截图2]({{ site.url }}/images/ShareExtension/img2.png)

创建完后，我们运行扩展。然后用 Safari 打开：http://mkapple.cn。点击 Safari 的分享按钮。发现扩展出现在分享选择栏里了。
<br>
![截图3]({{ site.url }}/images/ShareExtension/img3.png)
<br>
<br>
![截图4]({{ site.url }}/images/ShareExtension/img4.png)
<br>
可以发现苹果已经为你做好了一个基本的分享界面了。

## 3. 设置分享的数据类型
在扩展的 Info.plist 文件中找到 NSExtension 属性，该属性结构如下：
<br>
![截图5]({{ site.url }}/images/ShareExtension/img5.png)
<br>
各规则属性可以参考[官方文档](https://developer.apple.com/library/ios/documentation/General/Reference/InfoPlistKeyReference/Articles/SystemExtensionKeys.html#//apple_ref/doc/uid/TP40014212-SW11)，限制分享的图片、视频、网站地址的数量。

## 4. 验证用户输入
`SLComposeServiceViewController` —— 默认提供的标准化分享界面，提供以下属性和方法：
<br>
`NSString *contentText` —— 分享文本编辑框的内容<br>
`NSNumber *charactersRemaining` —— 设置剩下输入的内容字数<br>
`- (BOOL)isContentValid` —— 文本框内容改变时调用，返回 NO 则 Post 按钮禁用<br>

例：

~~~objc
- (BOOL)isContentValid {
    NSInteger textLength = self.contentText.length;
    self.charactersRemaining = @(maxCharactersAllowed - textLength);
    if(self.charactersRemaining.integerValue < 0){
        return NO;
    }
    
    return YES;
}
~~~

## 5. 提交分享
提交分享会调用 `- (void)didSelectPost`

~~~objc
- (void)didSelectPost {
    //上传分享数据
    [self uploadData];
    
    //提交分享请求
    [self.extensionContext completeRequestReturningItems:@[] completionHandler:nil];
    
}
~~~

## 6. 取消分享
取消分享会调用 `- (void)didSelectCancel`

~~~objc
-(void)didSelectCancel
{
    NSError *error = [NSError errorWithDomain:@"MK" code:500 userInfo:@{@"error": @"用户取消"}];
    
    //取消分享请求
    [self.extensionContext cancelRequestWithError:error];
}
~~~

## 7. 上传数据前的提取数据
介绍几个属性和方法：

* `UIViewController` 的 `extensionContext`，类是  [NSExtensionContext](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSExtensionContext_Class/index.html#//apple_ref/occ/cl/NSExtensionContext)
<br>
* `NSExtensionContext ` 的 `inputItems`，元素类型是[NSExtensionItem](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSExtensionItem_Class/index.html#//apple_ref/occ/cl/NSExtensionItem) 的数组，但实际上只有一个元素。
<br>
* `NSExtensionItem` 的 `attachments` 元素类型是 [NSItemProvider](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSItemProvider_Class/index.html#//apple_ref/occ/cl/NSItemProvider) 的数组，分享的图片、视频、URL 都封装在这个对象中。
<br>
* `NSItemProvider` 的 `registeredTypeIdentifiers` 元素类型是 NSString 的数组，但实际也只有一个类型，表示封装的数据的类型，格式为 [Uniform Type Identifier](https://developer.apple.com/library/ios/documentation/General/Conceptual/DevPedia-CocoaCore/UniformTypeIdentifier.html#//apple_ref/doc/uid/TP40008195-CH60)， 简称UTI，[详细列表](https://developer.apple.com/library/ios/documentation/Miscellaneous/Reference/UTIRef/Articles/System-DeclaredUniformTypeIdentifiers.html#//apple_ref/doc/uid/TP40009259)，UIT 类型的常量 [UTType Reference](https://developer.apple.com/library/ios/documentation/MobileCoreServices/Reference/UTTypeRef/index.html#//apple_ref/doc/constant_group/UTI_Text_Types)
<br>

例：

```objc
//提取数据
- (void)fetchItemDataAtBackground
{
//后台执行
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        
        NSArray<NSExtensionItem *> *itemArray = self.extensionContext.inputItems;
        
        //实际上只有一个 NSExtensionItem 对象
        NSExtensionItem *item = itemArray.firstObject;
        NSArray<NSItemProvider *> *providerArray = item.attachments;
        
        //输出 userInfo 或者 attachments 就可以看到 dataType 对应的字符串是啥了
        //NSLog(@"userInfo: %@", item.userInfo);
        
        for(NSItemProvider *provider in providerArray){
            //实际上一个NSItemProvider里也只有一种数据类型
            NSString *dataType = provider.registeredTypeIdentifiers.firstObject;
            if([dataType isEqualToString:@"public.image"]){
                
                [provider loadItemForTypeIdentifier:dataType options:nil completionHandler:^(UIImage *image, NSError *error) {
                    [attachImageArray addObject:image];
                }];
                
            } else if([dataType isEqualToString:@"public.plain-text"]){
                
                [provider loadItemForTypeIdentifier:dataType options:nil completionHandler:^(NSString *plainStr, NSError *error) {
                    [attachStringArray addObject: plainStr];
                }];
                
            } else if([dataType isEqualToString:@"public.url"]){
                
                [provider loadItemForTypeIdentifier:dataType options:nil completionHandler:^(NSURL *url, NSError *error) {
                    [attachURLArray addObject: url];
                }];
                
            }
        }
        
    });
}
```

## 8. 上传数据
首先，当点击 post 发布之后，扩展就被终止了，但是上传任务仍然在后台工作，它的图片、视频等数据缓存在哪呢，这时就需要容器应用提供一个缓存容器了，这时，就需要 group 了。
<br>
`TARGETS` ——> `主应用` ——> `Capabilities` ——> `App Group`
<br>

~~~objc
NSString *configName = @"com.donlinks.MKShareExtension.BackgroundSessionConfig";
NSURLSessionConfiguration *sessionConfig = [NSURLSessionConfiguration backgroundSessionConfigurationWithIdentifier: configName];
sessionConfig.sharedContainerIdentifier = @"group.MKShareExtension";
    
NSURLSession *shareSession = [NSURLSession sessionWithConfiguration: sessionConfig];
~~~

我把数据封装成 JSON 并上传到 [requestb.in](http://requestb.in/) 。该网站不是 https 的，需要在扩展的 Info.plist 加上允许所有网络请求：
<br>
![截图8]({{ site.url }}/images/ShareExtension/img8.png)
<br>

~~~xml
<key>NSAppTransportSecurity</key>
<dict>
	<key>NSAllowsArbitraryLoads</key>
	<true/>
</dict>
~~~

<br>
演示：
<br>
![截图6]({{ site.url }}/images/ShareExtension/img6.gif)
<br>

上传完成后到 requestb.in 可以看到
![截图7]({{ site.url }}/images/ShareExtension/img7.png)

## 9. 自定义分享界面思路
把 `SLComposeServiceViewController` 改为 `UIViewController` ，使用 `extensionContext` 属性的 `- (void)cancelRequestWithError:` 或者 `` 方法就可以让分享视图消失了。

Demo: [MKShareExtension](https://github.com/monkey19911021/MKShareExtension)
<br>

参考链接：
<br>
[CocoaChina](http://www.cocoachina.com/ios/20140923/9728.html)
<br>
[iOS8 分享应用扩展](http://www.devtalking.com/articles/ios8-day-by-day-day2-sharing-extension/)
<br>
[简书](http://www.jianshu.com/p/99d4ec43fd65)
<br>

P.S. 喜欢就分享或者点个赞呗

<!-- 多说评论框 start -->
<div class="ds-thread" data-thread-key="ShareExtension" data-title="ShareExtension" data-url="http://mkapple.cn/2016/06/23/ShareExtension"></div>
<!-- 多说评论框 end -->
<!-- 多说公共JS代码 start (一个网页只需插入一次) -->
<script type="text/javascript">
var duoshuoQuery = {short_name:"mkapple"};
	(function() {
		var ds = document.createElement('script');
		ds.type = 'text/javascript';ds.async = true;
		ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
		ds.charset = 'UTF-8';
		(document.getElementsByTagName('head')[0] 
		 || document.getElementsByTagName('body')[0]).appendChild(ds);
	})();
	</script>
<!-- 多说公共JS代码 end -->