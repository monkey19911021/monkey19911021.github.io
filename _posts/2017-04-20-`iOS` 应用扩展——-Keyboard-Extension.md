---
permalink: /2017/04/20/KeyboardExtension
header:
  teaser: "KeyboardExtension/img0.PNG"
tags: [Extension]
---
![截图1]({{ site.url }}/images/KeyboardExtension/img0.PNG)

注意！转载请注明[出处](http://mkapple.cn/2017/04/20/KeyboardExtension)和[作者](http://mkapple.cn)，谢谢
{: .notice--danger}

## 自定义键盘扩展
我们来做一个可以进行科学运算的计算器键盘吧。

### 2. 创建 Keyboard Extension 扩展
![截图3]({{ site.url }}/images/KeyboardExtension/img2.png)
<br>
<br>
![截图4]({{ site.url }}/images/KeyboardExtension/img3.png)
<br>
下一步，命名，完成。
<br>


修改运行时执行的主应用
<br>
![截图6]({{ site.url }}/images/KeyboardExtension/img5.png)
<br>

在 Run 一项里选择运行时执行的主应用, 勾上 Debug executable：<br>
![截图7]({{ site.url }}/images/KeyboardExtension/img6.png)
<br>

在 info.plist 文件修改自定义键盘的标题：
<br>
![截图9]({{ site.url }}/images/KeyboardExtension/img8.png)
<br>

以及其他的默认设置：
<br>
![截图10]({{ site.url }}/images/KeyboardExtension/img9.png)
<br>

<font color="orange">IsASCIICapable</font>--布尔值，默认为NO,表示第三方输入法是否可以向文档中插入ASCII字符串。如果您的第三方输入法专门提供[UIKeyboardTypeASCIICapable](https://developer.apple.com/documentation/uikit/uitextinputtraits#//apple_ref/c/econst/UIKeyboardTypeASCIICapable)输入法特性，那么将这个选项设置为YES。

<font color="orange">PrefersRightToLeft</font>--布尔值，默认为NO,表示第三方输入法使用的是否是一个从右到左的语言。如果您的输入法主语言是从右到左书写的，那么讲这个选项设置为YES。

<font color="orange">PrimaryLanguage</font>--字符串值，默认为en-US（美国英语），用以表示您输入法的主语言，使用模式为<语言>-<地区>。您可以在[这个页面](https://opensource.apple.com/source/CF/CF-476.14/CFLocaleIdentifier.c)找到某一语言和地区所对应的字符串。

<font color="orange">RequestsOpenAccess</font>--布尔值，默认为NO，表示第三方输入法是否要扩展到已满足基本输入法需求的沙箱之外。如果开放存取功能，您的输入法将获得如下特性，但每一个都有相应的责任：

1.访问位置服务和地址本数据库，每一个都需在第一次访问时获得授权。

2.可选择与容纳该输入法的应用的共享容器，使得在应用中可以定制词汇表。

3.能够将输入的字符和其他输入事件上传至服务器进行处理。

4.访问iCloud，例如，能够确保输入法的设置以及您的自动修正词汇能够在所有用户设备上同步。

5.通过包含还输入法的应用，能够访问Game Center和应用内购买。

6.能够和受控应用进行协同，如果您使用来设计该输入法以支持移动设备管理(MDM)。


### 3. 开始写代码
刚创建项目的时候系统会帮你创建一个继承于 **UIInputViewController** 的类：KeyboardViewController。我们直接在里面添加视图就可以构建键盘页面了。

#### 3.1. 构建键盘
我还是喜欢用 xib 来创建视图。
<br>
![截图9]({{ site.url }}/images/KeyboardExtension/img10.png)
<br>

添加到视图控制器：
~~~objc
keyBoardView = [[UINib nibWithNibName: @"Keyboard" bundle: nil]  instantiateWithOwner: nil options: nil][0];
keyBoardView.frame = CGRectMake(0, 0, [UIScreen mainScreen].bounds.size.width, 220); //控制键盘的尺寸
[self.view addSubview: keyBoardView]; //直接添加到视图控制器
~~~

为每个按钮设置方法
~~~objc
for(UIButton *btn in [keyBoardView subviews]){
    if([btn isKindOfClass: [UIButton class]]){
        [btn addTarget:self action:@selector(keyDidClick:) forControlEvents:UIControlEventTouchUpInside];
    }
}

-(void)keyDidClick:(UIButton *)btn {
    switch (btn.tag) {
        case 116:
            [self advanceToNextInputMode]; //切换输入法
            break;
        case 115:
            [self dismissKeyboard];//键盘隐藏
            break;
        case 119:{
            [self.textDocumentProxy deleteBackward];//删除上个输入
        }
            break;
        case 120:{
            [self.textDocumentProxy insertText: @"\n"]; //按下发送键，即 returnKey
        }
            break;
            
        default:
        {
            [self.textDocumentProxy insertText: [btn titleForState: UIControlStateNormal]]; //插入文本
            if(btn.tag == 114){
                //TODO: 计算
                
            }
        }
            break;
    }
}
~~~

#### 3.2. 运行
运行之后，我们到手机的设置 -> 通用 -> 键盘 -> 添加第三方键盘，然后打开微信，切换输入法到我们创建的键盘。
<br>
![截图6]({{ site.url }}/images/KeyboardExtension/img.gif)


Demo：[https://github.com/monkey19911021/MKCustomKeyboard](https://github.com/monkey19911021/MKCustomKeyboard)
<br>


P.S. 喜欢就分享或者点个赞呗

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
