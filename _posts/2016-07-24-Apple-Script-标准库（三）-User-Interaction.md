---
permalink: /2016/07/24/ACUserInteraction3
header:
  teaser: ""
categories: [AppleScript]
tags: [AppleScript]
---


注意！转载请注明[出处](http://mkapple.cn/2016/07/24/ACUserInteraction3)和[作者](http://mkapple.cn)，谢谢
{: .notice--danger}

{% include toc icon="file-text" title="目录" %}

## 一、 脚本命令

### 1. load script 加载脚本文件
```applescript
load script file : 根据文件别名和文件路径加载脚本
→ script : 返回脚本对象
```

### 2. store script 保存脚本到文件
```applescript
store script [script] : 保存脚本
[in file] : 位置
[replacing ask/‌yes/‌no] : 是否替换已有文件
```

### 3. run script 运行脚本文件
```applescript
scripting components
→ list of text : 脚本组件数组
```

```applescript
(* 例子： *)
-- 外部脚本文件：
on run {para1, para2} -- 外部参数	display alert para1 & para2end run

-- 内部脚本代码：
set target to load script (choose file)run script target with parameters {"hello", ", world"}scripting components
```

### 4. scripting components 脚本组件
```applescript
run script script : 运行脚本的文件或者对象
[with parameters list of any] : 参数数组
[in text] : the scripting component to use; default is the current scripting component
→ any : 运行结果
```

### 3. do shell script 运行 shell 脚本命令
```applescript
do shell script text : shell 脚本命令
[as type class] : 运行结果的预期类型
[administrator privileges boolean] : 是否需要用户鉴权
[user name text] : 用户鉴权的用户名，如果指定了用户名，必须指定用户密码
[password text] : 用户鉴权的用户密码
[with prompt text] : 用户鉴权提示框的提示
[altering line endings boolean] : change all line endings to Mac-style and trim a trailing one (default true)
→ text : 命令输出
```

```applescript
(* 例子： *)
do shell script "echo hello" user name "DONLINKS" password "11" with administrator privileges
-- hello
```

## 二、 各种命令

### 1. current date 当前日期
```applescript
current date
→ date : 当前日期
```

```applescript
(* 例子： *)
current date
-- date "2016年8月23日 星期二 上午11:45:21"

year of (current date)
-- 2016

month of (current date)
-- August

day of (current date)
-- 23

weekday of (current date)
-- Tuesday

time of (current date)
-- 42321 (11*3600 + 45*60 + 21)
```

### 2. 系统音量设置
```applescript
-- 获取系统音量设置
get volume settings 
→ volume settings : 当前系统音量设置

-- 设置系统音量
set volume [number] : 设置输出音量，0~7，参数可以忽略，如果指定则其余所有参数忽略
[output volume integer] : 输出音量 0~100
[input volume integer] : 输入音量 0~100
[alert volume integer] : 警告音音量 0~100
[output muted boolean] : 是否柔和音量

-- 音量属性
volume settings n : 音量属性
output volume (integer, r/o) : 输出音量
input volume (integer, r/o) : 输入音量
alert volume (integer, r/o) : 警告音音量
output muted (boolean, r/o) : 输出是否柔和

```

### 3. random number 随机数
```applescript
random number [number] : 随机数最大值，默认为1.0，如果此参数被指定则 from 和 to 参数会被忽略
[from number] : 随机数最小值 (默认为 0.0)
[to number] : 随机数最大值 (默认为 1.0)
[with seed integer] : a starting point for a repeatable sequence of random numbers. A value of 0 will use a random seed.
→ number : 随机数，如果参数都是整型，则结果是整型，否则为浮点型。
```

```applescript
(* 例子： *)
random number from 2.2 to 3.5
-- 2.926985691293
```

### 4. round 四舍五入
```applescript
round real : 取舍数字
[rounding up/‌down/‌toward zero/‌to nearest/‌as taught in school] : 取舍方向，默认为 to nearest，as taught in school为真的四舍五入。
→ integer : 结果
```

```applescript
(* 例子： *)
round 2.5 rounding as taught in school
-- 3
```

### 5. 系统环境变量
```applescript
system attribute [any] : 参数为 system attribute 数组结果的元素
[has integer] : test specific bits of response (ignored for environment variables)
→ any : 数组
```

```applescript
(* 例子： *)
system attribute
-- {"TMPDIR", "__CF_USER_TEXT_ENCODING", "SHELL", "HOME", "Apple_PubSub_Socket_Render", "SSH_AUTH_SOCK", "PATH", "LOGNAME", "XPC_SERVICE_NAME", "USER", "XPC_FLAGS"}

system attribute "SHELL"
-- "/bin/bash"
```

### 6. 系统信息
```applescript
system info
→ system information : 系统信息数组

system information n : system info 命令的结果
AppleScript version (text, r/o) : AppleScript 版本
AppleScript Studio version (text, r/o) : the AppleScript Studio version
system version (text, r/o) : 系统版本
short user name (text, r/o) : 系统用户名缩写
long user name (text, r/o) : 详细系统用户名
user ID (integer, r/o) : 当前用户 ID
user locale (text, r/o) : 用户地区
home directory (alias, r/o) : 用户 home 文件夹
boot volume (text, r/o) : 硬盘主分区
computer name (text, r/o) : 电脑名
host name (text, r/o) : 端口名
IPv4 address (text, r/o) : IPv4 地址
primary Ethernet address (text, r/o) : 物理地址
CPU type (text, r/o) : CPU 型号
CPU speed (integer, r/o) : CPU 频率，单位 MHz
physical memory (integer, r/o) : 内存，单位 MB
```

```applescript
(* 例子： *)
system attribute
-- {"TMPDIR", "__CF_USER_TEXT_ENCODING", "SHELL", "HOME", "Apple_PubSub_Socket_Render", "SSH_AUTH_SOCK", "PATH", "LOGNAME", "XPC_SERVICE_NAME", "USER", "XPC_FLAGS"}

system attribute "SHELL"
-- "/bin/bash"
```

### 7. time to GMT 
```applescript
time to GMT
→ integer : 当地时间与格林威治标准时间差
```

### 8. POSIX file 可移植操作系统接口
```applescript
POSIX path (text, r/o) : 把文件别名（alias）或者文件转换成带斜线的路径
→ String : 结果路径
```

```applescript
(* 例子： *)
POSIX path of (path to me)
--"/Users/AllenChow/Desktop/未命名.app/"
```

### 9. open location 打开URL
```applescript
open location [text] : 要打开的URL
[error reporting boolean] : 是否显示错误提示框
```

```applescript
(* 例子： *)
-- 打开谷歌浏览器里所有书签的地址
tell application "Google Chrome"	reopen	activate	set tag to bookmarks bar	set otherTag to other bookmarks		set marks to ((get bookmark items of otherTag) & (get bookmark items of tag))		repeat with temp in marks		open location (URL of temp as string)	end repeat	end tell
```



<!-- 多说评论框 start -->
<div class="ds-thread" data-thread-key="ACUserInteraction3" data-title="ACUserInteraction3" data-url="http://mkapple.cn/2016/07/24/ACUserInteraction3"></div>
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
