---
permalink: /2017/03/30/MagicalRecord_basic
header:
  teaser: ""
categories: [Objective-C, Swift]
tags: [单例, 扩展]
---

注意！转载请注明[出处](http://mkapple.cn/2017/03/30/MagicalRecord_basic)和[作者](http://mkapple.cn)，谢谢
{: .notice--danger}

感谢 [yerl](http://yerl.cn) 的指导。
{: .notice--info}

## 1. OC篇


断点运行看看：

![截图]({{ site.url }}/images/InstanceByExtension/img.png)

static let 是使用 dispatch_once 函数执行的，所以 sharedInstance 是一个单例对象。


<!-- 多说评论框 start -->
<div class="ds-thread" data-thread-key="MagicalRecord_basic" data-title="MagicalRecord_basic" data-url="http://mkapple.cn/2017/03/30/MagicalRecord_basic"></div>
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
