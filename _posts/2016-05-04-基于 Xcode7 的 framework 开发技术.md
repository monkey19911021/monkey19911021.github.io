---
permalink: /160504Framework
categories: [Objective-C]
tags: [Framework]
---

{% include toc icon="gears" title="目录" %}
## 1.  静态库和动态库*	Static Library （.a）*	Dynamic Library (.dylib)*	Static Framework (.framework)*	Dynamic Framework (.framework)### 1.1  什么是静态库程序编译一般需经预处理、编译、汇编和链接几个步骤。在我们的应用中，有一些公共代码是需要反复使用，就把这些代码编译为“库”文件；在链接步骤中，连接器将从库文件取得所需的代码，复制到生成的可执行文件中。这种库称为静态库，其特点是可执行文件中包含了库代码的一份完整拷贝；缺点就是被多次使用就会有多份冗余拷贝。### 1.2  什么是动态库动态库功能跟静态库类似，也是为了代码复用。静态库在程序的链接阶段被复制到了程序中，和程序运行的时候没有关系；动态库在链接阶段没有被复制到程序中，而是程序在运行时由系统动态加载到内存中供程序调用。使用动态库的优点是系统只需载入一次动态库，不同的程序可以得到内存中相同的动态库的复本，因此节省了很多内存。### 1.3  选择哪种库由于Appstore不支持上传包含动态库的app（其原因百度之），所以我们在开发app之前我们需要考虑好，app上架到哪里，动态更新功能是否必不可少。如果只上架到AppStore则只能是用静态库；如果只上架到本地服务器则可以使用动态库和静态库；如果不但要上架到而且还要上架到本地服务器让它具有动态更新功能的，则这个库则需要做得想静态库一样的动态库，我们只需要为静态库做一点小改动则可以变成动态库。## 2.  制作动态库
### 2.1  选择framework项目framework一般用于开发动态库
![截图2.1.1]({{ site.url }}/images/MKExceptionImg/MKExceptionImg1.png)命名为：FrameworkTest![截图2.1.2]({{ site.url }}/images/MKExceptionImg/MKExceptionImg2.png)### 2.2  创建通用集合![截图2.2.1]({{ site.url }}/images/MKExceptionImg/MKExceptionImg3.png)命名为项目名+Aggregate：FrameworkTestAggregate![截图2.2.2]({{ site.url }}/images/MKExceptionImg/MKExceptionImg4.png)### 2.3  添加项目依赖Build Phases ——> Target Dependencies ——> +![截图2.3.1]({{ site.url }}/images/MKExceptionImg/MKExceptionImg5.png)选择本项目![截图2.3.2]({{ site.url }}/images/MKExceptionImg/MKExceptionImg6.png)### 2.4  添加运行脚本Build Phases ——> + ——> New Run Script Phase![截图2.4.1]({{ site.url }}/images/MKExceptionImg/MKExceptionImg7.png)![截图2.4.2]({{ site.url }}/images/MKExceptionImg/MKExceptionImg8.png)添加如下脚本代码：

~~~sh# Sets the target folders and the final framework product.FMK_NAME=${PROJECT_NAME}# Install dir will be the final output to the framework.# The following line create it in the root folder of the current project.INSTALL_DIR=${SRCROOT}/Products/${FMK_NAME}.framework# Working dir will be deleted after the framework creation.WRK_DIR=buildDEVICE_DIR=${WRK_DIR}/Release-iphoneos/${FMK_NAME}.frameworkSIMULATOR_DIR=${WRK_DIR}/Release-iphonesimulator/${FMK_NAME}.framework# -configuration ${CONFIGURATION}# Clean and Building both architectures.xcodebuild -configuration "Release" -target "${FMK_NAME}" -sdk iphoneos clean buildxcodebuild -configuration "Release" -target "${FMK_NAME}" -sdk iphonesimulator clean build# Cleaning the oldest.if [ -d "${INSTALL_DIR}" ]thenrm -rf "${INSTALL_DIR}"fimkdir -p "${INSTALL_DIR}"cp -R "${DEVICE_DIR}/" "${INSTALL_DIR}/"# Uses the Lipo Tool to merge both binary files (i386 + armv6/armv7) into one Universal final product.lipo -create "${DEVICE_DIR}/${FMK_NAME}" "${SIMULATOR_DIR}/${FMK_NAME}" -output "${INSTALL_DIR}/${FMK_NAME}"rm -r "${WRK_DIR}"~~~![截图2.4.3]({{ site.url }}/images/MKExceptionImg/MKExceptionImg9.png)
作用是把编译后的库代码的模拟器版本和真机版本合并成一个通用版本，使此库能运行在模拟器和真机上。并且把生成的库放在项目文件夹里的Products文件夹。### 2.5  设置证书为了此动态库能运行在普通的iPhone上，需要为动态库像app一样设置开发证书和发布证书。
![截图2.5.1]({{ site.url }}/images/MKExceptionImg/MKExceptionImg10.png) ### 2.6  编译![截图2.6.1]({{ site.url }}/images/MKExceptionImg/MKExceptionImg11.png)选择通用库编译。
Build：![截图2.6.2]({{ site.url }}/images/MKExceptionImg/MKExceptionImg12.png)
## 3.  动态库工程联编
### 3.1  导入动态库项目我们新建一个普通app项目来测试此动态库,命名为：`TestDynamicFramework`。引入我们刚才的动态库项目
![截图3.1.1]({{ site.url }}/images/MKExceptionImg/MKExceptionImg13.png)——>![截图3.1.2]({{ site.url }}/images/MKExceptionImg/MKExceptionImg14.png)  ### 3.2  设置项目依赖导入动态库后可以在添加项目依赖处找到动态库通用集合![截图3.2.1]({{ site.url }}/images/MKExceptionImg/MKExceptionImg15.png)把动态库生成的framework文件拖进项目
![截图3.2.2]({{ site.url }}/images/MKExceptionImg/MKExceptionImg16.png)注意不要选上Copy items if needed### 3.3  项目设置删除Build Phases ——> Link Binary With Libraries下因为刚才拉进来的framework而产生的库连接![截图3.3.1]({{ site.url }}/images/MKExceptionImg/MKExceptionImg17.png)在Build Phases ——> Copy Bundle Resources ——> + ；把动态库添加进来，让app编译的时候把动态复制进main bundle。![截图3.3.2]({{ site.url }}/images/MKExceptionImg/MKExceptionImg18.png)这样子每次运行app项目的时候都会重新编译动态库，达到可以快速查看动态库的修改。但是每次这样运行app就要重新编译动态库会很耗费时间，所以在不需要修改动态库的时候把项目的动态库依赖给删除。*关于如何动态加载动态库以及动态库如何定位资源文件请看Demo。## 4.  制作静态库静态库与动态库的制作方法前5个步骤跟第2节1~6点所讲述的一样，但是静态库需要多执行以下步骤：### 4.1  项目设置打开Build Settings ——> 搜索mach， 在Mach-O Type一栏中选择Static Library
![截图4.1.1]({{ site.url }}/images/MKExceptionImg/MKExceptionImg19.png) ### 4.2  设置开放头文件打开Build Phases ——> Headers，把Project中的需要对外开放的头文件拖到Public中。![截图4.2.1]({{ site.url }}/images/MKExceptionImg/MKExceptionImg20.png)在与项目名相同的头文件中设置开放的头文件![截图4.2.2]({{ site.url }}/images/MKExceptionImg/MKExceptionImg21.png)### 4.3  制作资源包由于静态库属于静态连接，资源文件都需要放到bundle中，让其他项目主动加载它，静态库中的对象才能找到这些资源文件。
![截图4.3.1]({{ site.url }}/images/MKExceptionImg/MKExceptionImg22.png)

![截图4.3.2]({{ site.url }}/images/MKExceptionImg/MKExceptionImg23.png)设置bundle的bitcode为No。![截图4.3.3]({{ site.url }}/images/MKExceptionImg/MKExceptionImg24.png)Base SDK 设置为iOS的。![截图4.3.4]({{ site.url }}/images/MKExceptionImg/MKExceptionImg25.png)
把资源文件添加进Build Phases ——> Copy Bundle Resources![截图4.3.5]({{ site.url }}/images/MKExceptionImg/MKExceptionImg26.png)

![截图4.3.6]({{ site.url }}/images/MKExceptionImg/MKExceptionImg27.png) ~~~objc//获取该bundle对象代码：NSBundle *bundle = [NSBundle bundleWithURL:[[NSBundle mainBundle] URLForResource:@"StaticFrameworkTestBundle" withExtension:@"bundle"]];~~~
## 5.  静态库工程联编静态库工程联编方式跟动态库工程联编方式基本一样，参考第3节1~2点，但需要加上以下步骤：
### 5.1  把资源包放到项目中使用真机把资源包编译一下![截图5.1]({{ site.url }}/images/MKExceptionImg/MKExceptionImg28.png)

![截图5.2]({{ site.url }}/images/MKExceptionImg/MKExceptionImg29.png)把这资源包拖进项目但不要复制。### 5.2  设置连接方式Build Setting ——> 搜索other linker ——> 在Other Linker Flags 中添加-all_load。目的是为了把静态库中所有的类包括没有直接使用的类创建连接符，让项目可以加载所有的类。![截图5.2.1]({{ site.url }}/images/MKExceptionImg/MKExceptionImg30.png)OK, Run！。	## 6.  静态库一秒变动态库
### 6.1  基本设置在Build Settings ——> 搜索Preprocessor Macros ——> Debug和Release两项中分别都添加useStaticFramework=1。意思是useStaticFramework=1时使用静态库，useStaticFramework=0时使用动态库。![截图6.1.1]({{ site.url }}/images/MKExceptionImg/MKExceptionImg31.png)### 6.2  设置静态库的编译类型为动态![截图6.2.1]({{ site.url }}/images/MKExceptionImg/MKExceptionImg32.png)注意：动静并存时：![截图6.2.2]({{ site.url }}/images/MKExceptionImg/MKExceptionImg33.png)OK，现在只要修改库的编译类型以及app的useStaticFramework就可以让app项目加载动态库或者静态库了。
*具体代码参考Demo<!-- 多说评论框 start -->
<div class="ds-thread" data-thread-key="160504Framework" data-title="基于Xcode7的Framework开发技术" data-url="http://mkapple.cn/160504Framework"></div>
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