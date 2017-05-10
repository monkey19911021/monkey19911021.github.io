---
permalink: /2016/06/17/CoreSpotlight
header:
  teaser: "CoreSpotlight/corespotlight.png"
categories: [Objective-C, iOS API]
tags: [CoreSpotlight]
---
![截图1]({{ site.url }}/images/CoreSpotlight/corespotlight.png)

转载请注明出处：[http://mkapple.cn/2016/06/17/CoreSpotlight](http://mkapple.cn/2016/06/17/CoreSpotlight)

### 1. 先将framework导入：<br>
工程 -> Build Phases -> Link Binary With Libraries->![截图2]({{ site.url }}/images/CoreSpotlight/image2.png)
<br>

### 2. 创建搜索项

#### 2.1. 导入头文件

~~~objc
#import <CoreSpotlight/CoreSpotlight.h>
~~~

#### 2.2. 创建搜索项

~~~objc
NSMutableArray<CSSearchableItem *> *array = @[].mutableCopy;
    
//创建 SearchableItemAttributeSet
CSSearchableItemAttributeSet *set = [[CSSearchableItemAttributeSet alloc] initWithItemContentType: @"views"];
set.title = @"打开MKApple";
set.contentDescription = @"在应用里打开MKApple的网站";
set.thumbnailData = UIImagePNGRepresentation([UIImage imageNamed:@"fielIcon"]);
    
//创建 SearchableItem
/*
 uniqueIdentifier: 搜索项唯一标识符
 domainIdentifier: 搜索项的域标识，用于批量操作搜索项
*/
CSSearchableItem *item = [[CSSearchableItem alloc] initWithUniqueIdentifier:@"MKUnique" domainIdentifier:@"MKDomain" attributeSet:set];
[array addObject:item];
    
//更新搜索项
[[CSSearchableIndex defaultSearchableIndex] indexSearchableItems:array completionHandler:^(NSError * _Nullable error) {
        
}];
~~~


### 3. AppDelegate 中实现 spotlight 点击进来的事件

~~~objc
-(BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray * _Nullable))restorationHandler
{
    NSString *identifier = userActivity.userInfo[CSSearchableItemActivityIdentifier];
    
    if([identifier isEqualToString:@"MKUnique"]){
        
        UIViewController *rootViewCtrl = self.window.rootViewController;
        
        __block UIViewController *ctrl = [UIViewController new];
        
        [rootViewCtrl presentViewController:ctrl animated:YES completion:^{
            
            UIWebView *webView = [[UIWebView alloc] initWithFrame: ctrl.view.bounds];
            [webView sizeToFit];
            webView.scalesPageToFit = YES;
            [webView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@"http://mkapple.cn"]]];
            [ctrl.view addSubview:webView];
            
        }];
        
    }
    
    return YES;
}
~~~

### 4. 删除搜索项
~~~objc
//通过 uniqueIdentifier 删除
[[CSSearchableIndex defaultSearchableIndex] deleteSearchableItemsWithIdentifiers:@[@"MKUnique"] completionHandler:^(NSError * _Nullable error) {
        
}];
    
//通过 domainIdentifier 删除
[[CSSearchableIndex defaultSearchableIndex] deleteSearchableItemsWithDomainIdentifiers:@[@"MKDomain"] completionHandler:^(NSError * _Nullable error) {
        
}];
    
//删除本应用所有的索引
[[CSSearchableIndex defaultSearchableIndex] deleteAllSearchableItemsWithCompletionHandler:^(NSError * _Nullable error) {
        
}];
~~~


<!-- 网易云跟帖 -->
<div id="cloud-tie-wrapper" class="cloud-tie-wrapper"></div>
<script>
  var cloudTieConfig = {
    url: document.location.href, 
    sourceId: "",
    productKey: "ed9b8d43dc944e809d5c088decaffc0a",
    target: "cloud-tie-wrapper"
  };
</script>
<script src="https://img1.cache.netease.com/f2e/tie/yun/sdk/loader.js"></script>
