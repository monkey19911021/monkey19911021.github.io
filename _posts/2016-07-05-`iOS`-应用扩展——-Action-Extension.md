---
permalink: /2016/07/05/ActionExtension
header:
  teaser: "MKActionExtension/img1.gif"
categories: [Objective-C, Extension]
tags: [ActionExtension, Extension]
---
![截图1]({{ site.url }}/images/MKActionExtension/img1.gif)

转载请注明出处：[http://mkapple.cn/2016/06/20/ActionExtension](http://mkapple.cn/2016/06/20/ActionExtension)

## 1.干啥
任何场合下选择了一段英文，点击分享，使用 MKAction 打开可以翻译该文本。

## 2. 新建一个 iOS 项目，MKActionExtension

## 3. 新建 Action Extension
![截图2]({{ site.url }}/images/MKActionExtension/img2.png)

![截图3]({{ site.url }}/images/MKActionExtension/img3.png)

命名，完成。

修改运行时执行的主应用
<br>
![截图4]({{ site.url }}/images/MKActionExtension/img4.png)
<br>

在 Run 一项里选择运行时执行的主应用：<br>
![截图5]({{ site.url }}/images/MKActionExtension/img5.png)
<br>
勾上 Debug executable <br> 
![截图6]({{ site.url }}/images/MKActionExtension/img6.png)
<br>

运行扩展看看：
<br>
![截图7]({{ site.url }}/images/MKActionExtension/img7.gif)
<br>

默认提供了选取图片，并且显示图片的功能。

## 4. 获取输入数据

```objc
- (void)loadInputItems
{
    //1. 从扩展上下文获取 NSExtensionItem 数组
    NSArray<NSExtensionItem *> *itemArray = self.extensionContext.inputItems;
    
    //2. 从 NSExtensionItem 获取 NSItemProvider 数组
    NSExtensionItem *item = itemArray.firstObject;
    NSArray<NSItemProvider *> *providerArray = item.attachments;
    
    //3. 加载、获取数据
    NSItemProvider *itemProvider = providerArray.firstObject;
    if([itemProvider hasItemConformingToTypeIdentifier:(NSString *)kUTTypePlainText]){
        [itemProvider loadItemForTypeIdentifier:(NSString *)kUTTypePlainText options:nil completionHandler:^(NSString *text, NSError *error) {
            if(text) {
                [[NSOperationQueue mainQueue] addOperationWithBlock:^{
                    originalTextView.text = text;
                    
                    //4. 翻译
                    [self youdaoTranslate:text complate:^(NSString *translateText) {
                        translateTextView.text = translateText;
                    }];
                }];
            }
        }];
    }
}
```

介绍几个属性和方法：

* [NSExtensionContext](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSExtensionContext_Class/index.html#//apple_ref/occ/cl/NSExtensionContext)， 扩展上下文，可以获取进入扩展时的数据 ——inputItems， 元素类型为 NSExtensionItem 的数组。
<br>
* [NSExtensionItem](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSExtensionItem_Class/index.html#//apple_ref/occ/cl/NSExtensionItem)， 扩展数据项，包含附件数组 —— attachments，元素类型为 NSItemProvider 的数组。
<br>
* [NSItemProvider](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSItemProvider_Class/index.html#//apple_ref/occ/cl/NSItemProvider)，数据项的附件都封装在里面，要获取数据就要根据数据的 [UTI](https://developer.apple.com/library/ios/documentation/MobileCoreServices/Reference/UTTypeRef/index.html#//apple_ref/doc/constant_group/UTI_Text_Types) 类型来加载获取附件。
<br>
* `NSItemProvider` 的 `- (void)loadItemForTypeIdentifier:(NSString *)typeIdentifier options:(NSDictionary *)options completionHandler:(NSItemProviderCompletionHandler)completionHandler`， 根据 [UTI](https://developer.apple.com/library/ios/documentation/MobileCoreServices/Reference/UTTypeRef/index.html#//apple_ref/doc/constant_group/UTI_Text_Types) 类型来加载获取附件，

## 5. 翻译

利用 [有道 API](http://fanyi.youdao.com/openapi?path=data-mode) 提供的接口。

~~~objc
- (void)youdaoTranslate:(NSString *)text complate:(void (^)(NSString *))complate{
    NSURLSession *shareSession = [NSURLSession sharedSession];
    text = [text stringByReplacingOccurrencesOfString:@" " withString:@"%20"];
    NSString *urlStr = [NSString stringWithFormat:@"http://fanyi.youdao.com/openapi.do?keyfrom=%@&key=%@&type=data&doctype=json&version=1.1&q=%@", Keyfrom, YouDaoAPIkey, text];
    
    NSURLSessionDataTask *task = [shareSession dataTaskWithURL:[NSURL URLWithString: urlStr] completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        NSDictionary *dic = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingAllowFragments error:nil];
        NSArray *resultArray = dic[@"translation"];
        
        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
            complate(resultArray[0]);
        }];
    }];
    [task resume];
}
~~~

演示：
滚回最顶看图 -.-

Demo：[MKActionExtension](https://github.com/monkey19911021/MKActionExtension)
<br>

P.S. 喜欢就分享或者点个赞呗

<!-- 多说评论框 start -->
<div class="ds-thread" data-thread-key="ActionExtension" data-title="ActionExtension" data-url="http://mkapple.cn/2016/07/05/ActionExtension"></div>
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