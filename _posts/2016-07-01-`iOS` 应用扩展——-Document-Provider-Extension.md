---
permalink: /2016/07/01/DocumentProvider
header:
  teaser: "MKDocumentProvider/img1.png"
categories: [Objective-C, Extension]
tags: [DocumentProviderExtension, Extension]
---
![截图1]({{ site.url }}/images/MKDocumentProvider/img1.png)

## 1. 开发流程梗概

### 1.1. 创建一个文件编辑器 App，具有打开、导入其它 App 的文件；移动、导出、保存文件到其它 App 的 功能。

### 1.2. 创建一个可以查看自己文件的 App，具有 `Document Provider` 扩展。

### 1.3. 在 Document Provider 扩展里编写处理打开、导入、移动、导出文件事件的代码

### 1.4. 在 File Provider 扩展里编写协调文件的代码。

## 2. 两个关键字

### 2.1 共享容器。
一个沙盒能与另一个沙盒进行文件交换只能在共享容器里进行，通过创建 App Groups 就可以获得共享容器了。

### 2.2 文件权限。
当需要访问不在 App 自身的沙盒或者自身共享容器里的资源时，需要申请权限访问，使用到 NSURL 的两个方法:
<br>
开始安全访问：[- (BOOL)startAccessingSecurityScopedResource](https://developer.apple.com/reference/foundation/nsurl/1417051-startaccessingsecurityscopedreso?language=objc)
<br>
停止安全访问：[- (void)stopAccessingSecurityScopedResource](https://developer.apple.com/reference/foundation/nsurl/1413736-stopaccessingsecurityscopedresou?language=objc)

## 3. 文件编辑器 MKTextEdit
创建一个新 App 项目，界面如此：
![截图2]({{ site.url }}/images/MKDocumentProvider/img2.png)

运行起来酱紫：
<br>
![截图3]({{ site.url }}/images/MKDocumentProvider/img3.png)

因为这个 App 需要有访问其他 App 文件的权限，所以我们得开启它的 iCloud 功能，打开 iCloud 需要提供苹果开发者账号。
<br>
![截图4]({{ site.url }}/images/MKDocumentProvider/img4.png)

### 3.1 新建
新建文件并输入标题和内容后，我们可以对文件进行移动、导入和保存。移动和导入前要先保存文件，因为我们移动和导入都要提供文件的在 App 里的地址。

### 3.2 打开文件
需要 [UIDocumentMenuViewController](https://developer.apple.com/reference/uikit/uidocumentmenuviewcontroller?language=objc) 打开一个菜单选项，里面默认提供 iCloud 具有 Document Provider 扩展的 App。

~~~objc
- (void)displayDocumentPickerWithURIs:(NSArray<NSString *> *)UTIs {
    UIDocumentMenuViewController *importMenu = [[UIDocumentMenuViewController alloc] initWithDocumentTypes:UTIs inMode:documentPickerMode];
    importMenu.delegate = self;
    [self presentViewController:importMenu animated:YES completion:nil];
}
~~~  

UTI，[Uniform Type Identifier](https://developer.apple.com/library/ios/documentation/General/Conceptual/DevPedia-CocoaCore/UniformTypeIdentifier.html#//apple_ref/doc/uid/TP40008195-CH60) 即文件类型, 在[详细列表](https://developer.apple.com/library/ios/documentation/Miscellaneous/Reference/UTIRef/Articles/System-DeclaredUniformTypeIdentifiers.html#//apple_ref/doc/uid/TP40009259)可以查询。

`documentPickerMode` 为 `UIDocumentPickerModeOpen`。

当选择了某个菜单时时会调用代理方法：

~~~objc
#pragma mark - UIDocumentMenuDelegate
-(void)documentMenu:(UIDocumentMenuViewController *)documentMenu didPickDocumentPicker:(UIDocumentPickerViewController *)documentPicker
{
    documentPicker.delegate = self;
    [self presentViewController:documentPicker animated:YES completion:nil];
}
~~~

打开一个 [UIDocumentPickerViewController](https://developer.apple.com/reference/uikit/uidocumentpickerviewcontroller?language=objc) 控制器，以 iCloud 为例，里面提供了一系列文件选择，当选择了某个文件时，调用 UIDocumentPickerViewController 的代理方法。

~~~objc
-(void)documentPicker:(UIDocumentPickerViewController *)controller didPickDocumentAtURL:(NSURL *)url
{
    [controller dismissViewControllerAnimated:YES completion:nil];
    switch (controller.documentPickerMode) {
        case UIDocumentPickerModeImport:
        {
            [self importFile: url];
        }
            break;
        case UIDocumentPickerModeOpen:
        {
            [self openFile: url];
        }
            break;
        case UIDocumentPickerModeExportToService:
        {
            NSLog(@"保存到此位置：%@", url);
        }
            break;
        case UIDocumentPickerModeMoveToService:
        {
            NSLog(@"移动到此位置：%@", url);
        }
            break;
            
        default:
            break;
    }
}
~~~

代理返回的 url 是刚选择的文件在共享容器的位置，我们可以利用这地址对该文件进行操作了。

### 3.2 读取打开的文件

~~~objc
- (void)openFile:(NSURL *)url
{
    //1.获取文件安全访问权限
    BOOL accessing = [url startAccessingSecurityScopedResource];
    
    if(accessing){
        [activityView startAnimating];
        
        //2.通过文件协调器读取文件地址
        NSFileCoordinator *fileCoordinator = [[NSFileCoordinator alloc] initWithFilePresenter:nil];
        [fileCoordinator coordinateReadingItemAtURL:url
                                            options:NSFileCoordinatorReadingWithoutChanges
                                              error:nil
                                         byAccessor:^(NSURL * _Nonnull newURL) {
                                             
                                             [activityView stopAnimating];
                                             
                                             //3.读取文件协调器提供的新地址里的数据
                                             NSString *fileName = [newURL lastPathComponent];
                                             NSString *contStr = [NSString stringWithContentsOfURL:newURL encoding:NSUTF8StringEncoding error:nil];
                                             
                                             //4.把数据保存在本地缓存
                                             [self saveLocalCachesCont:contStr fileName:fileName];
                                             
                                             //5.显示数据
                                             currentFileName = fileName;
                                             titleTextField.text = fileName;
                                             contTextView.text = contStr;
                                         }];
        
    }
    
    //6.停止安全访问权限
    [url stopAccessingSecurityScopedResource];
}
~~~

1. 文件协调器 —— [NSFileCoordinator](https://developer.apple.com/reference/foundation/nsfilecoordinator?language=objc)，它与 File Provider extension 关联，当我们使用文件协调器读取、写入文件时都会触发 File Provider extension 的方法。等下介绍。

2. 可以看看打开的地址为：<br>
`file:///Users/AllenChow/Library/Developer/CoreSimulator/Devices/F7999205-1D00-4683-A2E1-EBB8B32D67BE/data/Library/Mobile%20Documents/com~apple~CloudDocs/newFile.txt`<br>
地址是位于 iCloud 里的。
  
### 3.3 保存打开的文件
当编辑过打开的文件后，进行保存时：

~~~objc
//1.通过文件协调器写入文件
NSFileCoordinator *fileCoorDinator = [NSFileCoordinator new];
NSError *error = nil;
[fileCoorDinator coordinateWritingItemAtURL:lastURL
                                    options:NSFileCoordinatorWritingForReplacing
                                      error:&error
                                 byAccessor:^(NSURL * _Nonnull newURL) {
                                     
                                     //2.获取安全访问权限
                                     BOOL access = [newURL startAccessingSecurityScopedResource];
                                     
                                     //3.写入数据
                                     if(access && [content writeToURL:newURL atomically:YES encoding:NSUTF8StringEncoding error:nil]){
                                         NSLog(@"保存原文件成功");
                                     }
                                     
                                     //4.停止安全访问权限
                                     [newURL stopAccessingSecurityScopedResource];
                                 }];
~~~

lastURL 是之前打开文件时 `-(void)documentPicker:(UIDocumentPickerViewController *)controller didPickDocumentAtURL:(NSURL *)url` 返回的地址。我们把文件保存回这个地址。

至此，我们已经完成了`在 App 内打开 另一个 App 的文件 ——> 读取 ——> 编辑 ——> 保存`的过程了。

### 3.4 导入文件
跟打开文件一样，但是 `documentPickerMode` 为 `UIDocumentPickerModeOpen`。

回调方法为：

~~~objc
- (void)importFile:(NSURL *)url
{
    [activityView startAnimating];
    
    //1.通过文件协调工具来得到新的文件地址，以此得到文件保护功能
    NSFileCoordinator *fileCoordinator = [[NSFileCoordinator alloc] initWithFilePresenter:nil];
    [fileCoordinator coordinateReadingItemAtURL:url options:NSFileCoordinatorReadingWithoutChanges error:nil byAccessor:^(NSURL * _Nonnull newURL) {
        [activityView stopAnimating];
        
        //2.直接读取文件
        NSString *fileName = [newURL lastPathComponent];
        NSString *contStr = [NSString stringWithContentsOfURL:newURL encoding:NSUTF8StringEncoding error:nil];
        
        //3.把数据保存在本地缓存
        [self saveLocalCachesCont:contStr fileName:fileName];
        
        //4.显示数据
        currentFileName = fileName;
        titleTextField.text = fileName;
        contTextView.text = contStr;
        
    }];
}
~~~

我们看看导入的模式下打开文件的地址为：<br>
`file:///Users/AllenChow/Library/Developer/CoreSimulator/Devices/F7999205-1D00-4683-A2E1-EBB8B32D67BE/data/Containers/Data/Application/82EE4511-930E-46FB-83B9-C6099C6A91A4/tmp/com.donlinks.MKTextEdit-Inbox/newFile.txt`<br>
地址是位置 MKTextEdit App 里的临时文件夹，说明了导入模式下文件是拷贝进来 App 的，所以我们可以直接读取文件数据。

### 3.5 移动和导出文件
移动和导出的处理方式一样的，只有带给 Document Provider extension 的 documentPickerMode 不一样，我们可以根据这个参数，如果是移动模式则删除 MKTextEdit App 里的文件，如果是导出模式就拷贝文件到共享容器。

~~~objc
- (IBAction)export:(id)sender {
    //1. 保存缓存文件
    [self modify:nil];
    documentPickerMode = UIDocumentPickerModeExportToService;
    NSURL *fileURL = [NSURL fileURLWithPath: [CachesFilePath stringByAppendingPathComponent: currentFileName]];
    
    //2.打开文件选择器
    [self displayDocumentPickerWithURL:fileURL];
}

- (void)displayDocumentPickerWithURL:(NSURL *)url {
    UIDocumentMenuViewController *importMenu = [[UIDocumentMenuViewController alloc] initWithURL:url inMode:documentPickerMode];
    importMenu.delegate = self;
    [self presentViewController:importMenu animated:YES completion:nil];
}
~~~

这次打开文件选择器提供的参数是 URL 并不是 UTI 了。

## 4. 共享容器提供方 MKDocumentProvider
当你想让自己 App 里的文件能够提供给其他 App 读取和编辑，需要做两件事：<br>
1. 创建 Document Provider extension 和 File Provider extension<br> 
2. 提供共享容器，把共享的文件放到容器里。<br>

### 4.1 创建扩展
![截图5]({{ site.url }}/images/MKDocumentProvider/img5.png)


![截图6]({{ site.url }}/images/MKDocumentProvider/img6.png)


![截图7]({{ site.url }}/images/MKDocumentProvider/img7.png)<br>
记得勾选 Include a File Provider extension ![截图8]({{ site.url }}/images/MKDocumentProvider/img8.png), 命名，完成。会弹出这样一个询问你时候激活 Scheme 的框，选择激活（Activate）。

创建 Document Provider extension 之后会附带一个 File Provider extension。

修改运行扩展时执行的主应用：<br>
![截图9]({{ site.url }}/images/MKDocumentProvider/img9.png)

![截图10]({{ site.url }}/images/MKDocumentProvider/img10.png)

![截图11]({{ site.url }}/images/MKDocumentProvider/img11.png)

### 4.2 创建 App Groups 
到项目的 Capabilities 里找到 App Groups，点击开启：
![截图12]({{ site.url }}/images/MKDocumentProvider/img12.png)

Document Provider extension 和 File Provider extension 也要开启 App Groups 并且都要用同一个 group。

创建完 Document Provider 扩展需要的东西后，我们来运行看看效果：<br>
![截图]({{ site.url }}/images/MKDocumentProvider/img.gif)


### 4.3 MKDocumentProvider 展示共享容器的文件

~~~objc
#pragma mark - 获取共享容器文件夹路径
- (NSString *)storagePath {
    NSURL *groupURL = [[NSFileManager defaultManager] containerURLForSecurityApplicationGroupIdentifier:APP_GROUP_ID];
    NSString *groupPath = [groupURL path];
    NSString *_storagePath = [groupPath stringByAppendingPathComponent:APP_FILE_NAME];
    NSFileManager *fileManager = [NSFileManager defaultManager];
    if (![fileManager fileExistsAtPath:_storagePath]) {
        [fileManager createDirectoryAtPath:_storagePath withIntermediateDirectories:NO attributes:nil error:nil];
    }
    return _storagePath;
}
~~~

### 4.4 Document Provider extension 
扩展的入口是一个 [UIDocumentPickerExtensionViewController](https://developer.apple.com/reference/uikit/uidocumentpickerextensionviewcontroller?language=objc) 类。在3.2 打开文件的使用 UIDocumentPickerViewController 展示的就是这个界面了，我们可以自定义这个界面。

`- (void)dismissGrantingAccessToURL:(nullable NSURL *)url;`
调用此方法来关闭该文档选择器的视图控制器以及所提供的授权访问的网址。每种模式都有自己所要求的URL。有关其完整的详细信息，请参阅 [Dismissing the User Interface](https://developer.apple.com/reference/uikit/uidocumentpickerextensionviewcontroller/1614391-dismissgrantingaccesstourl?language=objc)

`-(void)prepareForPresentationInMode:(UIDocumentPickerMode)mode;`<br>
当开始展示界面时自动调用，可以根据 mode 来做界面定制。

`NSArray<NSString *> *validTypes`<br>
当在打开和导入模式下才有值，值为 [UTI](https://developer.apple.com/library/ios/documentation/Miscellaneous/Reference/UTIRef/Articles/System-DeclaredUniformTypeIdentifiers.html#//apple_ref/doc/uid/TP40009259)

`NSURL *documentStorageURL`<br>
共享容器地址

`NSURL *originalURL`<br>
源文件地址，只有在移动和导入模式下才有值，访问该地址需要申请安全访问权限。

#### 4.4.1 打开和导入
在打开和导入模式下，我们需要用户点击了文件就关闭文档选择器和返回文件地址 URL

~~~objc
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    [tableView deselectRowAtIndexPath:indexPath animated:YES];
    if(self.documentPickerMode == UIDocumentPickerModeExportToService ||
       self.documentPickerMode == UIDocumentPickerModeMoveToService){
        return;
    }
    NSString *filePath = [storagePath stringByAppendingPathComponent: fileNamesArray[indexPath.row]];
    NSURL *fileURL = [NSURL fileURLWithPath: filePath];
    [self dismissGrantingAccessToURL: fileURL];
}
~~~

storagePath 是共享容器文件夹路径

#### 4.4.2 移动和导出
在移动和导出模式下，我们提供个按钮让用户确定保存的路径，当点击按钮的时候保存文件到共享容器：

~~~objc
- (void)exportFile
{
    NSURL *originalURL = self.originalURL;
    NSString *fileName = [originalURL lastPathComponent];
    NSString *exportFilePath = [storagePath stringByAppendingPathComponent: fileName];
    
    //1. 获取安全访问权限
    BOOL access = [originalURL startAccessingSecurityScopedResource];
    
    if(access){
        //2. 通过文件协调器访问读取该文件
        NSFileCoordinator *fileCoordinator = [NSFileCoordinator new];
        NSError *error = nil;
        [fileCoordinator coordinateReadingItemAtURL:originalURL options:NSFileCoordinatorReadingWithoutChanges error:&error byAccessor:^(NSURL * _Nonnull newURL) {
            
            //3.保存文件到共享容器
            [self saveFileFromURL:newURL toFileURL: [NSURL fileURLWithPath:exportFilePath]];
        }];
        
    }
    
    //4. 停止安全访问权限
    [originalURL stopAccessingSecurityScopedResource];
}
~~~

### 4.4 文件提供程序 File Provider extension 
文件提供程序应用扩展允许在主应用程序的沙箱外使用打开和移动操作来访问文件。当 MKTextEdit 使用文件协调器 `NSFileCoordinator ` 打开不存在 MKDocumentProvider 的文件时，我们可以使用 `- (void)startProvidingItemAtURL:(NSURL *)url completionHandler:(void (^)(NSError *))completionHandler` 方法来为 MKTextEdit 创建新文件：

~~~objc
//文件保护，文件不存在则创建新文件
- (void)startProvidingItemAtURL:(NSURL *)url completionHandler:(void (^)(NSError *))completionHandler {
    NSError* error = nil;
    __block NSError* fileError = nil;
    
    NSFileManager *fileMgr = [NSFileManager defaultManager];
    NSString *filePath = [url path];
    if([fileMgr fileExistsAtPath:filePath]){ //1
        //文件已存在，返回
        completionHandler(error);
        return;
    }
    
    //文件不存在，创建新文件，并写入url
    NSData *fileData = [@"新建文件：" dataUsingEncoding:NSUTF8StringEncoding]; //2
    
    [self.fileCoordinator coordinateWritingItemAtURL:url options:NSFileCoordinatorWritingForReplacing error:&error byAccessor:^(NSURL *newURL) {
        [fileData writeToURL:newURL options:0 error:&fileError]; //3
    }];

    if (error!=nil) {
        completionHandler(error);
    } else {
        completionHandler(fileError);
    }
}
~~~

## 5. 搞掂
Demo: [MKDocumentProvider](https://github.com/monkey19911021/MKDocumentProvider)

演示：
<br>
![截图13]({{ site.url }}/images/MKDocumentProvider/img13.gif)
<br>

参考链接：
<br>
[CocoaChina](http://www.cocoachina.com/ios/20141007/9835.html)<br>
[简书](http://www.jianshu.com/p/3e3674630190)
<br>


P.S. 喜欢就分享或者点个赞呗  

<!-- 多说评论框 start -->
<div class="ds-thread" data-thread-key="DocumentProvider" data-title="DocumentProvider" data-url="http://mkapple.cn/2016/07/01/DocumentProvider"></div>
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