---
permalink: /2017/03/30/MagicalRecord_basic
header:
  teaser: ""
categories: [Objective-C]
tags: [CoreData]
---

注意！转载请注明[出处](http://mkapple.cn/2017/03/30/MagicalRecord_basic)和[作者](http://mkapple.cn)，谢谢
{: .notice--danger}


## 1. 安装
源代码地址：[MagicalRecord](https://github.com/magicalpanda/MagicalRecord)

通过 CocoaPods 安装：
```ruby
pod "MagicalRecord"
```

## 2. 设置 CoreData 堆栈
1.导入 CoreData.framework 框架。

2.打开 AppDelegate.m 文件，增加以下方法：

```objc
#import "MagicalRecord.h" //导入头文件

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [MagicalRecord setupAutoMigratingCoreDataStack]; //设置堆栈
    return YES;
}

- (void)applicationWillTerminate:(UIApplication *)application {
    [MagicalRecord cleanUp]; //app退出时的清理工作
}
```

3.建立数据模型

![截图]({{ site.url }}/images/MagicalRecord/01.png)

我增加了一个 Person 类，在使用到该实体的类里导入 Model+CoreDataModel.h 文件，因为系统会帮你生成 Person 类文件，如果没自动生成则需要手动创建；选中模型文件 ——> Editor ——> Create NSManagedobject Subclass；com + b 编译一下，发现报文件重复错误，则把刚自动生成的文件删除，再使用 __#import "Model+CoreDataModel.h"__ 就应该不会报错了，点进去也可以看到系统为你生成的类和属性。

## 3. 增删改查

### 3.1 添加数据
```objc
NSManagedObjectContext *context = [NSManagedObjectContext MR_defaultContext];
Person *per = [Person MR_createEntityInContext: context];
per.name = @"aaa";
per.age = 18;
per.headImg = UIImagePNGRepresentation([UIImage imageNamed: @"delete"]);
per.date = [NSDate date];
[context MR_saveToPersistentStoreAndWait];
```

### 3.2 删除数据
```objc
Person *person = [muArray objectAtIndex: indexPath.row];
[person MR_deleteEntity];//删除实体
[[NSManagedObjectContext MR_defaultContext] MR_saveToPersistentStoreAndWait];

//或者清空某个类的所有数据
[Person MR_truncateAll];
```

### 3.3 修改数据
```objc
Person *person = [muArray objectAtIndex: indexPath.row];
person.name = @"bbb";
[[NSManagedObjectContext MR_defaultContext] MR_saveToPersistentStoreAndWait];
```

### 3.4 查询数据
```objc
//从持久化存储（PersistantStore）中查询出所有的Person实体
NSArray *people = [Person MR_findAll];

//查询出所有的Person实体并按照 lastName 升序（ascending）排列
NSArray *peopleSorted = [Person MR_findAllSortedBy:@"lastName" ascending:YES];

//查询出所有的Person实体并按照 lastName 和 firstName 升序（ascending）排列
NSArray *peopleSorted = [Person MR_findAllSortedBy:@"lastName,firstName" ascending:YES];

//查询出所有的Person实体并按照 lastName 降序，firstName 升序（ascending）排列
NSArray *peopleSorted = [Person MR_findAllSortedBy:@"lastName:NO,firstName" ascending:YES];
//或者
NSArray *peopleSorted = [Person MR_findAllSortedBy:@"lastName,firstName:YES" ascending:NO];

//查询出所有的Person实体 firstName 为 Forrest 的实体
Person *person = [Person MR_findFirstByAttribute:@"firstName" withValue:@"Forrest"];
```

### 3.5 高级查询
```objc
//使用 NSPredicate 来实现高级查询。
NSPredicate *peopleFilter = [NSPredicate predicateWithFormat:@"department IN %@", @[dept1, dept2]];
NSArray *people = [Person MR_findAllWithPredicate:peopleFilter];

//NSFetchRequest
NSPredicate *peopleFilter = [NSPredicate predicateWithFormat:@"department IN %@", departments];
NSFetchRequest *people = [Person MR_requestAllWithPredicate:peopleFilter];

//自定义 NSFetchRequest
NSPredicate *peopleFilter = [NSPredicate predicateWithFormat:@"department IN %@", departments];
NSFetchRequest *peopleRequest = [Person MR_requestAllWithPredicate:peopleFilter];
[peopleRequest setReturnsDistinctResults:NO];
[peopleRequest setReturnPropertiesNamed:@[@"firstName", @"lastName"]];
NSArray *people = [Person MR_executeFetchRequest:peopleRequest];


//查询实体个数
//返回的是 NSNumber 类型
NSNumber *count = [Person MR_numberOfEntities];

//基于 NSPredicate 查询条件过滤后的实体个数
NSNumber *count = [Person MR_numberOfEntitiesWithPredicate:...];

//返回的是 NSUInteger 类型
+ (NSUInteger) MR_countOfEntities;
+ (NSUInteger) MR_countOfEntitiesWithContext:(NSManagedObjectContext *)context;
+ (NSUInteger) MR_countOfEntitiesWithPredicate:(NSPredicate *)searchFilter;
+ (NSUInteger) MR_countOfEntitiesWithPredicate:(NSPredicate *)searchFilter inContext:(NSManagedObjectContext *)context;


//合计操作
NSInteger totalFat = [[CTFoodDiaryEntry MR_aggregateOperation:@"sum:" onAttribute:@"fatCalories" withPredicate:predicate] integerValue];

NSInteger fattest  = [[CTFoodDiaryEntry MR_aggregateOperation:@"max:" onAttribute:@"fatCalories" withPredicate:predicate] integerValue];

NSArray *caloriesByMonth = [CTFoodDiaryEntry MR_aggregateOperation:@"sum:" onAttribute:@"fatCalories" withPredicate:predicate groupBy:@"month"];
```

### 3.6 Demo地址
[MagicalRecord_basic](https://github.com/monkey19911021/MagicalRecord_basic)

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
