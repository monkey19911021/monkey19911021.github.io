---
permalink: /2016/06/27/PhotoEditingExtension
header:
  teaser: "MKPhotoExtension/img1.PNG"
categories: [Objective-C, Extension]
tags: [PhotoEditingExtension, Extension]
---
![截图1]({{ site.url }}/images/MKPhotoExtension/img1.PNG)

转载请注明出处：[http://mkapple.cn/2016/06/27/PhotoEditingExtension](http://mkapple.cn/2016/06/27/PhotoEditingExtension)

## 1. 开发流程梗概

### 1.1. 创建扩展

### 1.2. 处理开始编辑照片时的事件（startContentEditingWithInput）

### 1.3. 对预览照片进行编辑，添加滤镜、模糊，调整亮度、饱和度、对比度等

### 1.4. 处理完成编辑照片时的事件，对源图进行同样的编辑，把修改后的源图保存到某个位置。

## 2. 创建照片编辑扩展
关于如何创建一个扩展请参考[这里](http://mkapple.cn/2016/06/20/TodayExtension)的第二步骤, 我们要创建的是 Photo Editing Extension 。
<br>
![截图2]({{ site.url }}/images/MKPhotoExtension/img2.png)

创建完后，我们直接运行扩展。
<br>
![截图3]({{ site.url }}/images/MKPhotoExtension/img3.gif)

出现了一个 Hello World 界面。

## 3. 设置可编辑数据类型
在扩展的 Info.plist 文件中找到 NSExtension 属性，该属性结构如下：
<br>
![截图4]({{ site.url }}/images/MKPhotoExtension/img4.png)

## 4. 开始编辑前
`- (BOOL)canHandleAdjustmentData:` ，当照片被编辑过时调用，方法有一个参数：`PHAdjustmentData`，里面包含了上次编辑时所使用的编辑器的数据：
<br>
4.1. `formatIdentifier`，编辑器的唯一ID <br>
4.2. `formatVersion`，编辑器的版本号 <br>
4.3. `data`，编辑器保存的数据，自定义数据 <br>
<br>
返回值： <br>
4.4. `NO`, 扩展编辑的是被编辑过的效果图 <br>
4.5. `YES`, 扩展编辑的是没有任何改动的原图 <br>

例：

~~~objc
- (BOOL)canHandleAdjustmentData:(PHAdjustmentData *)adjustmentData {
    BOOL result = [formatIdentifier isEqualToString: adjustmentData.formatIdentifier] &&
    [formatVersion isEqualToString: adjustmentData.formatVersion];
    if(result){
        currentFilterName = [[NSString alloc] initWithData:adjustmentData.data encoding:NSUTF8StringEncoding];
    }
    return result;
    
}
~~~

## 5. 开始编辑
`- (void)startContentEditingWithInput:placeholderImage:`，当进入到编辑界面时调用，`contentEditingInput` 属性包含了照片的类型、地理位置、预览图和原图位置等信息。

## 6. 完成编辑
`- (void)finishContentEditingWithCompletionHandler:`，在这方法里主要需要完成两个任务：
<br>
6.1. 设置输出的 adjustmentData, 即把照片编辑的信息保存起来，放到 completionHandler 里处理。
<br>
6.2. 对原图进行相同的编辑

~~~objc
- (void)finishContentEditingWithCompletionHandler:(void (^)(PHContentEditingOutput *))completionHandler {
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        PHContentEditingOutput *output = [[PHContentEditingOutput alloc] initWithContentEditingInput:self.input];
        
        //1. 设置输出的adjustmentData
        PHAdjustmentData *adjustmentData = [[PHAdjustmentData alloc] initWithFormatIdentifier:formatIdentifier
                                                                                formatVersion:formatVersion
                                                                                         data:[currentFilterName dataUsingEncoding:NSUTF8StringEncoding]];
        output.adjustmentData = adjustmentData;
        
        //2. 对原图进行相同的编辑
        CIImage *fullSizeImage = [CIImage imageWithContentsOfURL: _input.fullSizeImageURL];
        
        UIGraphicsBeginImageContext(fullSizeImage.extent.size);
        CIImage *filterImage = [MKImageUtil filterWithOriginalImage:fullSizeImage filterName:currentFilterName]; //添加滤镜
        UIImage *drawImage = [UIImage imageWithCIImage:filterImage];
        [drawImage drawInRect:fullSizeImage.extent];
        UIImage *outputImage = UIGraphicsGetImageFromCurrentImageContext();
        NSData *jpegData = UIImageJPEGRepresentation(outputImage, 1.0);
        UIGraphicsEndImageContext();
        
        [jpegData writeToURL:output.renderedContentURL atomically:YES];
        
        completionHandler(output);
    });
}
~~~

## 7. 取消编辑
`- (BOOL)shouldShowCancelConfirmation`, 返回 YES，则取消编辑是提醒用户是否取消。

<br>
Demo: [MKPhotoEditingExtension](https://github.com/monkey19911021/MKPhotoEditingExtension.git)

<br>
演示：
<br>
![截图5]({{ site.url }}/images/MKPhotoExtension/img5.gif)
<br>

参考链接：
<br>
[照片扩展](http://objccn.io/issue-21-5/)
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
