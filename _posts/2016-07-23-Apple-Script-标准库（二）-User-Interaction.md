---
permalink: /2016/07/23/ACUserInteraction2
header:
  teaser: ""
categories: [AppleScript]
tags: [AppleScript]
---


注意！转载请注明[出处](http://mkapple.cn/2016/07/23/ACUserInteraction2)和[作者](http://mkapple.cn)，谢谢
{: .notice--danger}

{% include toc icon="file-text" title="目录" %}

## 一、 File Commands

### 1. info for 获取文件信息
```applescript
info for file : 根据文件路径获取文件信息
[size boolean] : 如果值为 false，返回的 size 的值为 missing value
→ file information : 字典信息

-- file information
name (text, r/o) : 文件名
displayed name (text, r/o) : the user-visible name of the item
short name (text, r/o) : the short name (CFBundleName) of the item (if the item is an application)
name extension (text, r/o) : 扩展名
bundle identifier (text, r/o) : the item’s bundle identifier (if the item is a package)
type identifier (text, r/o) : the item’s type identifier
kind (text, r/o) : the kind of the item
default application (alias, r/o) : 默认此文件的打开程序
creation date (date, r/o) : 创建日期
modification date (date, r/o) : 修改日期
file type (text, r/o) : the file type of the item
file creator (text, r/o) : the creator type of the item
short version (text, r/o) : the item’s short version string (from the Finder’s ‘Get Info’ box)
long version (text, r/o) : the item’s long version string (from the Finder’s ‘Get Info’ box)
size (integer, r/o) : 文件大小
alias (boolean, r/o) : Is the item an alias file?
folder (boolean, r/o) : Is the item a folder?
package folder (boolean, r/o) : Is the item a package (a folder treated as a file?)
extension hidden (boolean, r/o) : 是否隐藏扩展名
visible (boolean, r/o) : 是否隐藏
locked (boolean, r/o) : 是否被锁定?
busy status (boolean, r/o) : Is the item currently in use?
folder window (bounding rectangle, r/o) : the coordinates of the folder’s window (if the item is a folder)
```

```applescript
(* 例子： *)
-- 查看选择文件的名称
set fileInfo to info for (choose file)get name of result
```

### 2. list disks 罗列磁盘
```applescript
list disks
→ list of text : 返回磁盘名称的数组
```

```applescript
(* 例子： *)
list disks
-- {"Macintosh HD", "home", "net"}
```

### 3. list folder 罗列文件夹下的所有文件和文件夹
```applescript
list folder file : 根据地址罗列
[invisibles boolean] : 罗列中是否包含隐藏文件，默认为真
→ list of text : 文件和文件夹名称数组
```

```applescript
(* 例子： *)
list folder (choose folder) without invisibles
-- {"Applications", "Library", "System", "Users", "用户信息"}
```

### 4. mount volume 连接ftp、smb等服务器
```applescript
mount volume text : 要加载的名字或者地址 (e.g. ‘afp://server/volume/’)
on server text : the server on which the volume resides; omit if URL path provided
[in AppleTalk zone text] : the AppleTalk zone in which the server resides; omit if URL path provided
[as user name text] : 连接服务的用户名，如果是客人用户则忽略
[with password text] : 连接服务的密码，如果是客人用户则忽略
→ file : 返回文件名
```

```applescript
(* 例子： *)
mount volume "ftp://192.168.5.71" as user name "st" with password "123"
-- file "st@192.168.5.71:"
```

### 5. path to 返回指定规定应用或者脚本的地址
```applescript
path to n: 参数见截图，还有：current application, frontmost application, application “AppName”, me, it
[as type class] : 转换返回数据的类型， alias 或者 string (默认 alias)
→ text or alias : 规定项目的路径
```

![截图1]({{ site.url }}/images/AppleScriptAddition/img.png){: .align-center}

```applescript
(* 例子： *)
path to application "Terminal"
-- alias "Macintosh HD:Applications:Utilities:Terminal.app:"
```

### 6. path to 返回指定规定的文件夹的路径
```applescript
path to application support/‌applications folder/‌desktop/‌desktop pictures folder/‌documents folder/‌downloads folder/‌favorites folder/‌Folder Action scripts/‌fonts/‌help/‌home folder/‌internet plugins/‌keychain folder/‌library folder/‌modem scripts/‌movies folder/‌music folder/‌pictures folder/‌preferences/‌printer descriptions/‌public folder/‌scripting additions folder/‌scripts folder/‌services folder/‌shared documents/‌shared libraries/‌sites folder/‌startup disk/‌startup items/‌system folder/‌system preferences/‌temporary items/‌trash/‌users folder/‌utilities folder/‌workflows folder/‌voices/‌apple menu/‌control panels/‌control strip modules/‌extensions/‌launcher items folder/‌printer drivers/‌printmonitor/‌shutdown folder/‌speakable items/‌stationery : 文件夹名称
[from system domain/‌local domain/‌network domain/‌user domain/‌Classic domain] : 在哪路寻找指定的文件夹
[as type class] : 转换返回数据的类型， alias 或者 string (默认是 alias)
[folder creation boolean] :如果该文件夹不存在是否创建 (默认 true)
→ alias : 指定文件夹的路径
```

```applescript
(* 例子： *)
path to desktop
-- alias "Macintosh HD:Users:AllenChow:Desktop:"
```

### 7. path to resource 返回该应用的资源文件夹（Resources）下的文件的绝对路径
```applescript
path to resource text : 指定资源的名称
[in bundle file] : bundle 的名称 (默认是当前应用的 bundle)
[in directory text] : Resources 文件夹下的子文件夹
→ alias : 返回的路径
```

```applescript
(* 例子： *)
path to resource "applet.rsrc"
-- alias "Macintosh HD:Users:AllenChow:Desktop:未命名.app:Contents:Resources:applet.rsrc"
```

## 二、 Clipboard Commands 粘贴板命令

### 1. set the clipboard to any 把数据放到粘贴板里
```applescript
(* 例子： *)
set the clipboard to "xxxxx"
-- 按粘贴键就可以粘贴数据了
``` 

### 2. the clipboard 获取粘贴板里的数据
```applescript
the clipboard
[as type class] : 转换类型
→ any : 数据
```

```applescript
(* 例子： *)
the clipboard as string
-- 粘贴板里的数据
```

### 3. clipboard info 粘贴板信息
```applescript
clipboard info
[for type class] : 约束数据类型
→ list of list :类型为 {data type, size} 的数组
```

```applescript
(* 例子： *)
clipboard info for string
-- 拷贝 “xxx” 返回 {{string, 3}}
```

## 三、 File Read/Write 文件读写命令

### 1. open for access file 打开文件通道
```applescript
open for access file : 根据文件路径打开文件，如果文件不存在则自动创建
[write permission boolean] : 是否给写入权限
→ integer : 通道标记，整型，配合 ‘read’, ‘write’ 和 ‘close access’ 命令使用
```

```applescript
(* 例子： *)
set accessNum to open for access (choose file) with write permission
-- xxx
```

### 2. close access 关闭通道
```applescript
close access any : 根据通道标记关闭通道
```

```applescript
(* 例子： *)
close access accessNum
```

### 3. read 读取文件信息
```applescript
read any : 根据通道标记、文件别名、文件路径名来读取文件
[from integer] : 读取开始位置，默认为上次开始读取位置
[for integer] : 读取长度，默认读取到结尾
[to integer] : 读取结束位置
[before text] : …or read up to but not including this character…
[until text] : …or read up to and including this character
[using delimiter text] : the value that separates items to read…
[using delimiters list of text] : …or a list of values that separate items to read
[as type class] : 转换类型
→ any : 读取的数据
```

```applescript
(* 例子： *)
set accessNum to open for access (choose file) with write permissionset content to read accessNum as stringclose access accessNumget content
-- 读取的数据
```

### 4. write 写入文件数据
```applescript
write any : 写入的数据
to any : 根据通道标记、文件别名、文件路径名写入文件
[starting at integer] : 从指定位置开始写入，默认从开头写入
[for integer] : the number of bytes to write; if not specified, write all the data provided
[as type class] : 写入的数据类型: text, data, list等
```

```applescript
(* 例子： *)
set accessNum to open for access (choose file) with write permissionwrite "\nxxxx\n" to accessNum starting at (get eof accessNum) as string -- 在文件结尾写入close access accessNum
```

### 5. get eof 获取文件字节数
```applescript
get eof any : 参数类型包括：通道标记、文件别名、文件路径名
→ integer : 文件字节数
```

```applescript
(* 例子： *)
set accessNum to open for access (choose file) with write permissionset ii to get eof accessNumclose access accessNumget ii
-- 字节数
```

### 6. set eof 设置文件字节数
```applescript
set eof any : 参数类型包括：通道标记、文件别名、文件路径名
to integer : 设置的新长度，任何数据超过这长度都会丢失
```

```applescript
(* 例子： *)
set accessNum to open for access (choose file) with write permissionset eof accessNum to 180close access accessNum
```

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

