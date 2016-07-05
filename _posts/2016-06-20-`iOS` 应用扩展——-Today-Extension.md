---
permalink: /2016/06/20/TodayExtension
header:
  teaser: "MKTodayExtension/img0.PNG"
categories: [Objective-C, Extension]
tags: [TodayExtension, Extension]
---
![截图1]({{ site.url }}/images/MKTodayExtension/img0.PNG)

转载请注明出处：[http://mkapple.cn/2016/06/20/TodayExtension](http://mkapple.cn/2016/06/20/TodayExtension)

## 今日面板
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;今日面板属于一个应用的扩展，但是该扩展跟该应用是两个独立的程序。OK，来做一个任务列表（ToDoList）来熟悉 Today Extension 的开发吧。

### 1. 创建一个任务列表：
![截图2]({{ site.url }}/images/MKTodayExtension/img1.png)
<br>
一个基本的 tableView ，具有点击右上角添加新元素、左滑 tableViewCell 删除元素的，so easy。

### 2. 创建 Today Extension 扩展
![截图3]({{ site.url }}/images/MKTodayExtension/img2.png)
<br>
<br>
![截图4]({{ site.url }}/images/MKTodayExtension/img3.png)
<br>
下一步，命名，完成。
<br>

会弹出这样一个框：
<br>
![截图5]({{ site.url }}/images/MKTodayExtension/img4.png)
<br>
选择激活（Activate）
<br>

修改运行时执行的主应用
<br>
![截图6]({{ site.url }}/images/MKTodayExtension/img5.png)
<br>

在 Run 一项里选择运行时执行的主应用：<br>
![截图7]({{ site.url }}/images/MKTodayExtension/img6.png)
<br>
勾上 Debug executable <br> 
![截图8]({{ site.url }}/images/MKTodayExtension/img7.png)
<br>

修改在今日面板显示的标题：
<br>
![截图9]({{ site.url }}/images/MKTodayExtension/img8.png)
<br>

运行扩展看看：
<br>
![截图10]({{ site.url }}/images/MKTodayExtension/img9.png)
<br>


### 3. 在扩展中创建 tableView
![截图11]({{ site.url }}/images/MKTodayExtension/img10.png)
<br>
如同平常一样，塞一个 tableView 到 sb 里面，接着注意了，在扩展想要修改显示的视图大小需要在 viewControll 里设置 `preferredContentSize` 属性：

~~~objc
self.preferredContentSize = CGSizeMake(0, todoList.count * 44-1);
~~~

### 4. 扩展使用主应用的共享数据
由于扩展和主应用是相互独立的程序，所以需要主应用共享出数据给扩展使用，使用`App Group`来解决问题。
<br>
`TARGETS` ——> `主应用` ——> `Capabilities` ——> `App Group`
<br>
点击开启
<br>
![截图12]({{ site.url }}/images/MKTodayExtension/img11.png)
<br>
输入：group.BundleID
<br>
<br>
主应用同步数据到 group

~~~objc
- (void)updateTodoSnapshot
{
    NSUserDefaults *infoDic = [[NSUserDefaults alloc] initWithSuiteName: GROUP_ID];
    [infoDic setObject:todoList forKey:TODO_LIST_ID];
    [infoDic synchronize];
    
    //更新今日面板信息，NotificationCenter framework
    [[NCWidgetController widgetController] setHasContent:YES forWidgetWithBundleIdentifier:@"com.donlinks.MKTodyExtension.MKTodayTarget"];
}
~~~

当 group 数据更新了，扩展会调用 `NCWidgetProviding` 协议，实现该协议的两个方法

~~~objc
- (void)widgetPerformUpdateWithCompletionHandler:(void (^)(NCUpdateResult))completionHandler {
    [self loadContents];
    completionHandler(NCUpdateResultNewData);
}

- (UIEdgeInsets)widgetMarginInsetsForProposedMarginInsets:(UIEdgeInsets)defaultMarginInsets{
    return UIEdgeInsetsMake(0, 27, 0, 0);
}
~~~
第一个方法是系统通知扩展要更新时，扩展调用的方法；第二个方法返回一个内补大小，如果不实现，默认情况视图左侧会有一定的缩进。
<br>

扩展获取同步数据

~~~objc
NSUserDefaults *infoDic = [[NSUserDefaults alloc] initWithSuiteName: GROUP_ID];
todoList = [infoDic objectForKey: TODO_LIST_ID];
~~~

OK, 演示一遍：
<br>
![截图13]({{ site.url }}/images/MKTodayExtension/img12.gif)
<br>

### 5. 扩展调起主应用
首先，给主应用注册一个 URL Scheme
![截图14]({{ site.url }}/images/MKTodayExtension/img13.png)
<br>

点击今日面板时打开主应用

~~~objc
-(void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    [tableView deselectRowAtIndexPath:indexPath animated:YES];
    
    NSString *text = todoList[indexPath.row];
    if([text isEqualToString:@"添加ToDo"]){
        text = @"new_item";
    }else{
        NSUserDefaults *infoDic = [[NSUserDefaults alloc] initWithSuiteName: GROUP_ID];
        NSArray *array = [infoDic objectForKey: TODO_LIST_ID];
        
        text = [NSString stringWithFormat:@"%@", @([array indexOfObject:text])];
    }
    
    NSString *urlStr = [@"MKToday://" stringByAppendingString: text];
    
    //打开主应用
    [self.extensionContext openURL:[NSURL URLWithString:urlStr] completionHandler:nil];
}
~~~

演示：
<br>
![截图15]({{ site.url }}/images/MKTodayExtension/img14.gif)
<br>

OK, 打完收工，Demo在此：[MKTodyExtension](https://github.com/monkey19911021/MKTodyExtension)
<br>

参考链接：
<br>
[苹果官方文档](https://developer.apple.com/library/ios/documentation/NotificationCenter/Reference/NotificationCenter_Framework/index.html)
<br>
[App Extension编程指南](http://www.cocoachina.com/ios/20140904/9527.html)
<br>

P.S. 喜欢就分享或者点个赞呗

<!-- 多说评论框 start -->
<div class="ds-thread" data-thread-key="TodayExtension" data-title="TodayExtension" data-url="http://mkapple.cn/2016/06/20/TodayExtension"></div>
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