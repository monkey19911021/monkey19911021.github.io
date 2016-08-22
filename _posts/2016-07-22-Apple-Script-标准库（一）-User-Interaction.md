---
permalink: /2016/07/22/ACUserInteraction
header:
  teaser: ""
categories: [AppleScript]
tags: [AppleScript]
---

不知道为啥，一看到 AppleScipt 标准库的 API 我就忍不住把他们都整理出来了。
{: .notice--info}

注意！转载请注明[出处](http://mkapple.cn/2016/07/22/ACUserInteraction)和[作者](http://mkapple.cn)，谢谢
{: .notice--danger}

### 1. beep 咚！
```applescript
beep [integer] : 咚的 n 声！
```

```applescript
(* 例子： *)
-- 咚10声
beep 10
```

### 2. choose application 选择应用程序
```applescript
choose application
[with title text] : 选择框标题
[with prompt text] : 选择框提示
[multiple selections allowed boolean] : 是否允许多选? (默认 false)
[as type class] : 把返回的结果转换成指定类型，可以是 application 或者 alias.
→ application : 返回 application "程序名"
```

```applescript
(* 例子： *)
-- 把选择的应用程序转换成字符串
set appName to choose application with title "选择应用程序" with prompt "随便选吧" as string-- 打开所选的应用程序tell application appName	reopen -- 打开	activate -- 放到窗口顶层end tell
```

### 3. choose remote  application 选择远程应用程序
```applescript
choose remote application
[with title text] : 标题
[with prompt text] : 提示
→ application : 返回 application "程序名"
```

### 4. choose color 选择颜色
```applescript
choose color
[default color RGB color] : 默认颜色
→ RGB color : 返回选择的颜色，类型为组合
```

### 5. choose file 选择文件
```applescript
choose file
[with prompt text] : 选择框提示
[of type list of text] : 选择文件的类型的集合
[default location alias] : 默认位置
[invisibles boolean] : 是否显示隐藏文件? (默认 false)
[multiple selections allowed boolean] : 能否多选? (默认 false)
[showing package contents boolean] : 能否显示包内容 (包会被当做文件夹. 默认 false.)
→ alias : {alias "Macintosh HD:Users:MK:百度云同步盘:92e8647ajw1f3zd77ku5yg20b407tk27.gif"}
```

```applescript
(* 例子： *)
choose file of type {"gif", "png", "app"} with prompt "选择文件" with invisibles, multiple selections allowed and showing package contents
get result as list
```

### 5. choose file name 选择文件保存的路径
```applescript
choose file name
[with prompt text] : 路径选择框路径
[default name text] : 默认文件名
[default location alias] : 默认选择路径
→ file : file "Macintosh HD:Users:MK:百度云同步盘:新建文件"
```

```applescript
(* 例子： *)
choose file name default name "新建文件"
get result as string
```

### 6. choose file folder 选择文件夹
```applescript
choose folder
[with prompt text] : 文件夹选择框标题
[default location alias] : 默认路径
[invisibles boolean] : 是否显示隐藏文件夹? (默认 false)
[multiple selections allowed boolean] : 是否多选? (默认 false)
[showing package contents boolean] : 是否显示包内容? (包会被当做文件夹. 默认 false.)
→ alias : 选择的文件夹的别名
```

### 7. choose from list 选择框
```applescript
choose from list list of text or number : 进行选择的文本或者数字集合
[with title text] : 选择框标题
[with prompt text] : 选择框提示
[default items list of text or number] : 默认选中的文本或者数字
[OK button name text] : 确定按钮文本
[cancel button name text] : 取消按钮文本
[multiple selections allowed boolean] : 是否允许多选
[empty selection allowed boolean] : 是否允许不选择
→ list of number or text : 返回选择的文本或者数字的集合
```

```applescript
(* 例子： *)
choose from list {"A", "B", "C", "D"} with title "选择吧" with prompt "多项选择题" default items {"A", "B"} OK button name "选好了" cancel button name "我选不了" with multiple selections allowed and empty selection allowed
get result as list
```

### 8. display notification 系统消息框
```applescript
display notification [text] : 消息内容
[with title text] : 消息标题（默认是应用的名称）
[subtitle text] : 子标题
[sound name text] : 声音名称（系统偏好设置 -> 声音 -> 声音效果）
```

```applescript
(* 例子： *)
display notification "消息来了" with title "消息标题" subtitle "子消息标题" sound name "Glass"
```

### 9. choose URL 选择 URL
```applescript
choose URL
[showing list of Web servers/‌FTP Servers/‌Telnet hosts/‌File servers/‌News servers/‌Directory services/‌Media servers/‌Remote applications] : which network services to show
[editable URL boolean] : Allow user to type in a URL?
→ URL : the chosen URL
```

### 10. delay 延迟执行下一个命令
```applescript
delay [number] : 延迟 n 秒
```

```applescript
(* 例子： *)
tell application "Terminal"	activate	do script "echo 将在3秒内睡眠"		delay 1	do script "echo 将在2秒内睡眠" in window 1		delay 1	do script "echo 将在1秒内睡眠" in window 1		delay 1end telltell application "Finder"	sleepend tell
```

### 11. display alert 警告框
```applescript
display alert text : 标题
[message text] : 内容
[as critical/‌informational/‌warning] : 警告框类型
[buttons list of text] : 按钮文本集合
[default button text or integer] : 默认选中按钮文本或者索引
[cancel button text or integer] : 取消按钮文本或索引
[giving up after integer] : 几秒后自动消失
→ alert reply : {button returned:"点击按钮文本", gave up: 是否自动消失了(true/false)}
```
```applescript
(* 例子： *)
display alert "警告框" message "警告框消息" as warning buttons {"按钮1", "按钮2"} default button 1 cancel button 2 giving up after 5if gave up of result then display alert "警告框自动消失了"
```

### 12. display dialog 对话框
```applescript
display dialog text : 对话内容
[default answer text] : 默认文本框内容
[hidden answer boolean] : 是否把文本加密? (默认 false)
[buttons list of text] : 按钮文本集合
[default button text or integer] : 默认选中按钮文本或索引
[cancel button text or integer] : 取消按钮文本或索引
[with title text] : 对话框标题
[with icon text or integer] : 图标路径
[with icon stop/‌note/‌caution] : 系统图标
[with icon file] : 图标路径
[giving up after integer] : 几秒后消失
→ dialog reply : {button returned:"点击按钮的文本", text returned:"文本框内容", gave up:是否自动消失了(true/false)}
```
```applescript
(* 例子： *)
set iconFile to choose file of type {"icns", ""} with showing package contentsset btnList to {"说完了", "以后再说", "..."}display dialog "你想说啥" default answer "不想说啥" buttons btnList default button 1 cancel button 2 with title "说一说" with icon iconFile giving up after 3 with hidden answerif gave up of result then display alert text returned of result
```

### 13. say 说
```applescript
say text : 说话内容
[displaying text] : 把说话的内容反馈到窗口
[using text] : 用什么嗓音，嗓音可以到系统偏好设置 -> 听写与语音 -> 文本至语音 -> 系统嗓音 查找
[speaking rate number] : 阅读速率，人类平均阅读速率为每分钟180~220个字
[pitch number] : the base pitch frequency, a real number from 0 to 127. Values correspond to MIDI note values, where 60 is equal to middle C. Typical pitches range from around 30 to 40 for a low-pitched male voice to perhaps 55 to 65 for a high-pitched child’s voice.
[modulation number] : the pitch modulation, a real number from 0 to 127. A value of 0 corresponds to a monotone in which all speech is at the base speech pitch. Given a pitch value of 46, a modulation of 2 means the widest range of pitches would be 44 to 48.
[volume number] : 音量，0~1
[stopping current speech boolean] : stop any current speech before starting (default is false). When false, “say” waits for previous speech commands to complete before beginning to speak.
[waiting until completion boolean] : wait for speech to complete before returning (default is true).
[saving to any] : 保存到文件，建议用AIFF格式
```
```applescript
(* 例子： *)
display dialog "输入你想说的话" default answer "I don't know what to say!"set sayContent to text returned of resultchoose from list {"Ting-Ting", "Alex", "Zarvox", "Agnes", "Victoria"} with title "选择说话的声音" with prompt "Ting-Ting是说中文的但是需要另外下载" default items {"Zarvox"} OK button name "选好了" cancel button name "用默认的吧"set sayVoice to result as stringif sayVoice is equal to "false" then set sayVoice to "Victoria"say sayContent using sayVoicesay sayContent using sayVoice saving to choose file name default name "什么鬼声音.AIFF"
```

<!-- 多说评论框 start -->
<div class="ds-thread" data-thread-key="ACUserInteraction" data-title="ACUserInteraction" data-url="http://mkapple.cn/2016/07/22/ACUserInteraction"></div>
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
