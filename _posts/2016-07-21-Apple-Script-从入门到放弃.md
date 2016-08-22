---
permalink: /2016/07/21/AppleScriptStart
header:
  teaser: "AppleScriptStart/img0.png"
categories: [AppleScript]
tags: [AppleScript]
---

## 1. hello, world
```applescript
display dialog "hello, world" (* 1 *)
display alert "hello, world" (* 2 *)
say "hello, world" (* 3 *)
```

## 2. 变量
```applescript
-- 声明，可以直接 set someValue to "" 来声明 someValue
property someValue : ""

-- 设值 
set someValue to "MKApple"

-- 取值
get someValue
```

## 3. 变量类型
---

### 3.1 number 数字
```applescript
set x to 10

-- 字符串转整型
set x to "10" as number 
```

### 3.2 string 字符串
```applescript
set str to "hello, world"

-- 使用 the length of 获取字符串长度 
set strLength to the length of "hello" 

-- 使用 & 连接字符串 
set str to "hello," & " world" 

-- 使用 \ 来转义 
set str to "\"hello\"" 

-- 整型转字符串 
set str to 10 as string 

-- 使用 every character of  把字符串转成字符数组 
set strList to every character of "String"

-- 根据指定字符分割字符串 
set defaultDelimiters to AppleScript's text item delimiters (* 先保存默认字符分割符 *)
set AppleScript's text item delimiters to "," (* 使用 AppleScript's text item delimiters 来指定分割符 *)
set strList to every text item of "1,12,321,cdsa"
set AppleScript's text item delimiters to defaultDelimiters (* 分割完之后需要还原分割符 *)
get strList

-- 将数组按照指定分割符组成一段字符串 
set defaultDelimiters to AppleScript's text item delimiters
set AppleScript's text item delimiters to ","
set restoredStr to {"hello", "world"} as string
set AppleScript's text item delimiters to defaultDelimiters
get restoredStr

-- 数字转 ASCII码
ASCII character 65
返回：A

-- ASCII 码转数字
ASCII number "A"
返回：65

-- 取出一段字符串在另一段字符串的位置
offset of "el" in "hello"
返回：2
```

### 3.3 list 集合
```applescript
set list1 to {10, "value1", "value2"}

-- 使用 the length of 或者 the count of 获取集合长度 
the length of list1

-- 使用 & 连接集合 
set list2 to list1 & {"value4"} 

-- 使用 & 连接集合和其他类型变量，& 左边的值的类型决定连接后的值的类型 
get "valueX" & list1

(* 使用 item index of 获取 list 第 index 个元素，
若 index 为正数则从第一个向后第 index 个元素，
若 index 为负数则从最后一个向前第 index 个元素 *)
item 2 of list2 

-- 更改集合元素 
set item 2 of list2 to "value3" 

-- 使用 the end of 为集合从末尾添加元素 
set the end of list2 to "value5"

-- 使用 some item of 来随机获取 list2 中元素 
set randomItem to some item of list2 

-- 使用 the last item of 获取 list2 最后的元素, 同理可以使用 first、second 等 
set lastItem to the last item of list2

-- 使用 items begin through end of 获取 list2 的子集合
set shotList to items 2 through 3 of list2

-- 使用 reverse of 反转集合 
set reverseList to reverse of list2

-- 将变量转换成集合 
set list3 to "value6" as list 
```

### 3.4 record 字典
```applescript
set record1 to {key1: "value1", key2: "value2"}

-- 使用 the length/count 获取字典 key/value 対数目
get the count of record1

-- 使用 the key of record 为字典改变值
set the key1 of record1 to "changedValue"

-- 使用 the key of record 根据 key 获取字典值
get the key1 of record1

-- 使用 the items of 获取由字典所有值组成的数组
get the items of record1
```

### 3.5 copy
```applescript
set record1 to {key1:"value1", key2:"value2"}
copy the result as list to {text_returned, button_pressed}
get text_returned
```

---

## 4. 捕捉错误
```applescript
try
    set x to 1 / 0
on error the error_message number the error_number
    (*
      error_message记录了出错信息，是一个String类型。
      error_number记录了错误代号，是一个number类型。
    *)
    display alert "出错: " & the error_number & "   详细：" & the error_message
end try
```

## 5. if ... else

### 5.1. 一般形式
```applescript
if 条件 then  
    ...  
else if 条件 then   
    ...  
else  
    ...  
end if  
```

### 5.2. 至简形式
```applescript
-- 都写在一行
if 条件 then ...  
```

### 5.3. 数字比较
- = 等于
- \> 大于
- < 小于
- \>= 大于等于
- <= 小于等于
- /= 不等于

### 5.4. 字符串比较
```applescript
-- str1 是否以 str2 开头
str1 begins with (或者 starts with) str2 

-- str1 是否不以 str2 开头
str1 does not start with str2 

-- str2 是否以 str2 结尾
str1 ends with str2

-- str2 是否不以 str2 结尾
str1 does not end with str2

-- 是否一致
str1 is equal to str2

-- str1 是否在 str2 之前
str1 is comes before str2 

-- str1 是否在 str2 之后
str1 is comes after str2

-- str1 是否在 str2 里
str1 is in str2

-- str1 是否不在 str2 里
str1 is not in str2

-- str1 是否包含 str2
str1 contains str2

-- str1 是否不包含 str2
str1 does not contain str2
```

## 6. 循环语句
```applescript
-- repeat x times
set num to 0
repeat 3 times
	set num to num + 1
end repeat

-- repeat with 计数变量 from 起始值 to 结束值 by 增长值
-- by 增长值 缺省为1
set num to 0
set startValue to 0
set endValue to 10
set stepValue to 2
repeat with temp from startValue to endValue by stepValue
	set num to num + temp
end repeat

-- repeat while 循环条件
set num to 0
repeat while num /= 10
	set num to num + 1
end repeat

-- repeat until 循环结束条件
set num to 0
repeat until num = 10
	set num to num + 1
end repeat

-- repeat with 单个元素 in 集合
set num to 0
repeat with temp in {1, 2, 3}
	set num to num + temp
end repeat
```

## 7. Handler 函数
```applescript
-- 带参数和返回值的函数
on charactersOfStr(str)	return every character of strend charactersOfStrget charactersOfStr("hello,world")
```

```applescript
-- 如果要在其他应用内调用本文件中的 Handler，要在调用后加 of me 来指定该 Handler 所在文件
on warnFunc()	display dialog "将要清空废纸篓" buttons {"OK"} default button "OK"	return button returned of result as stringend warnFunctell application "Finder"	if warnFunc() of me is equal to "OK" then empty the trashend tell
```

```applescript
-- 调用文件外部的脚本使用 load script
tell application "Finder"	try		set scriptPath to choose file		set warnScript to (load script scriptPath)		tell warnScript						try				set resultWarn to warnFunc()								try					if resultWarn is equal to "OK" then empty the trash				on error					display alert "废纸篓没东西"				end try							on error				display alert "没有找到该Handler"			end try					end tell	on error		display alert "请选择文件"	end try	end tell
```

P.S. 喜欢就分享或者点个赞呗

<!-- 多说评论框 start -->
<div class="ds-thread" data-thread-key="AppleScriptStart" data-title="AppleScriptStart" data-url="http://mkapple.cn/2016/07/21/AppleScriptStart"></div>
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
