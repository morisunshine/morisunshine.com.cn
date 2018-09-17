---
date: '2014-04-22'
title: "iOS编码规范参考"
slug: ios_code_standards
category: iOS
description: "经常说没有最好的风格，只有统一风格，所以编码规范就是为了统一我们的编码风格。而在这里我对我平时使用的编码规范进行了一个总结，希望能给大家提供参考"
tags: [iOS]
comments: true 
share: true

---

经常说没有最好的风格，只有统一风格，所以编码规范就是为了统一我们的编码风格。而在这里我对我平时使用的编码规范进行了一个总结，希望能给大家提供参考

## 注释

建议使用[VVDocumenter](https://github.com/onevcat/VVDocumenter-Xcode)插件

### 多行注释

格式:

```objc
/**

              注释内容
*/
```

### 单行注释

格式:

```objc
///在对文件、类、函数进行注释时推荐使用多行注释，在函数体内对代码块进行注释时，使用单行注释
```
### 函数的注释

函数注释的格式为

```objc
/**
 *  @brief  
 *  @param
 *  @return
 **/
 在brief中需要写明函数的主要功能、注意事项
 在param中需要写明函数的变量类型、变量的作用
 在return中需要写明函数的返回类型、返回值的作用
 如有其他需要说明的地方，可以在@return后面添加新项。如
 /**
 *  @brief  
 *  @param
 *  @return
 *  @warning
 **/

/**
 *  @brief      上传图片。上传成功后会自动将图片放入缓存，缓存的key为图片的url
 *  @param      UIImage，需要上传的图片
 *  @return     void
 *  @blocks
 *              success:返回的NSDictionary中包含服务器的response信息，包括图片id（id）,url(url),宽度(width),高度(height)。使用括号中的名称从NSDictionary中获取。
 *              failed:返回error
 **/
```

## 命名

### 常量的命名

(如宏、枚举、静态局部变量等)应该以小写字母k开头，使用混合大小写的格式来分隔单词。

### 函数的命名

函数名应该已小写字母开头，并混合大小写，其中的每个参数都应该是小写字母开头，读起来应该像句子一样，访问器方法应该与他们getting的成员变量的名字一样，但不应该以get作为前缀，如：

```objc
  - (id)getDelegate;	//avoid
  - (id)delegate;   	//good
```

### 变量的命名

成员变量应该已小写字母开头，并以下划线作为后缀，如usernameTextField_,使用KVO/KVC绑定成员变量时，可以以一个下划线为前缀。

>公共变量命名:
小写字母开头。如:imageView;

>实例变量命名:

>私有变量:
应该以下划线开头。如:_addButton

>常量命名:
以小写k开头，混合大小写。如:kInvalidHandle, kWritePerm


### 图片的命名

应该已“模块+功能+作用+样式”的形式

```objc
如:message_private_at_button_bg_normal.png
```

### 类的命名

类名、分类名、协议名应该以大写字母开始，并混合小写字母来分隔单词，应该已“模块+功能+子功能”的方式：

```objc
如:MessagePrivateAtsomebody
```
应用级的类，应避免不用前缀，跨应用级的类，应使用前缀，
```objc
如:GTMSendMessage
```

## 分类名

类别名应该有两三个字母作为前缀已表示为某项目的一部分，并且包含所扩展的类的名字，如我们要创建一个NSString的类别以解析，我们将类别放在一个名为GTMNSString+Parsing.h的文件中。类别名本身为GTMStringParsingAdditions(类别名与文件名不同是为了让文件可以包含更多相同功能的扩展)类名与包含类别名之间应一个空格分隔。

```objc
如:
//NSString 为要扩展的类名, GTMStringParsingAdditions为类别名
@interface NSString (GTMStringParsingAdditions)
 	    - (NSString *)gtm_foobarString;
 	    @end

 	    @interface FoobarViewController ()
 	    @property(nonatomic, retain) NSView *dongleView;
 	    - (void)performLayout;
 	    @end
```

## 注意事项

### 条件语句

为避免错误，条件语句体必须使用大括号，即便语句体中的语句可以不必使用大括号（比如只有一行语句）。常见的错误包括在不使用大括号的情况下添加第二行语句，以为它属于if语句的一部分。此外，更可怕的事情是，如果条件语句中的代码行被注释，则本不术语条件语句的下一行代码将变成条件语句的一部分。此外，这种编码风格和所有其它条件语句均保持一致。

```objc
如:
if (!error) {
    return success;
}
```

### 变量

变量的命名应尽可能具有自解释性。除了在for()循环语句中，应避免使用单个字母变量名称。
除非是常量，星号应紧贴变量名称表示指向变量的指针，
```objc
如:
NSString *text;
```

应尽可能使用属性定义替代单一的实例变量。避免在初始化方法,dealloc方法和自定义的setter和getter方法中直接读取实例变量参数（init,initWithCoder:，等等）

```objc
如:
@interface NYTSection: NSObject
 
@property (nonatomic) NSString *headline;
 
@end

应尽量避免下面的方式:

@interface NYTSection : NSObject {
    NSString *headline;
}
```

### 下划线
当使用属性变量时，应通过self.来获取和更改实例变量。这就意味着所有的属性将是独特的，因为它们的名称前会加上self。本地变量名称中不应包含下划线

```objc
如:
self.imgView.backgroundColor = [UIColor black];

而尽量避免:

_imgView.backgroundColor = [UIColor black];
```

### Immutable实例初始化
在创建NSString,NSDictionary,NSArray和NSNumber等对象的immutable实例时，应使用字面量。需要注意的是，不应将***nil***传递给NSArray和NSDictionary字面量，否则会引起程序崩溃。所以在存入数据到NSArray和NSDictionary时也要判断一下数据是否是nil。

```objc
NSArray *names = @[@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul"];
NSDictionary *productManagers = @{@"iPhone" : @"Kate", @"iPad" : @"Kamal", @"Mobile Web" : @"Bill"};
NSNumber *shouldUseLiterals = @YES;
NSNumber *buildingZIPCode = @10018;

不恰当:
NSArray *names = [NSArray arrayWithObjects:@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul", nil];
NSDictionary *productManagers = [NSDictionary dictionaryWithObjectsAndKeys: @"Kate", @"iPhone", @"Kamal", @"iPad", @"Bill", @"Mobile Web", nil];
NSNumber *shouldUseLiterals = [NSNumber numberWithBool:YES];
NSNumber *ZIPCode = [NSNumber numberWithInteger:10018];
```
### 类型

最好用NSInteger和NSUInteger而不是用int，long。其他的float同理。


### CGRect函数
当需要获取一个CGRect矩形的x,y,width,height属性时，应使用CGGeometry函数，而非直接访问结构体成员。

```objc
如：
CGRect frame = self.view.frame;
 
CGFloat x = CGRectGetMinX(frame);
CGFloat y = CGRectGetMinY(frame);
CGFloat width = CGRectGetWidth(frame);
CGFloat height = CGRectGetHeight(frame);
 
不恰当：
CGRect frame = self.view.frame;
 
CGFloat x = frame.origin.x;
CGFloat y = frame.origin.y;
CGFloat width = frame.size.width;
CGFloat height = frame.size.height;
```

### 常量

相对字符串字面量或数字，我们更推荐适用常量。应使用static方式声明常量，而非使用#define的方式来定义宏。

```objc
例如：
static NSString * const NYTAboutViewControllerCompanyName = @"The New York Times Company";  
static const CGFloat NYTImageThumbnailHeight = 50.0;
 
不恰当:
//#define CompanyName @"The New York Times Company"
//#define thumbnailHeight 2
```

### 枚举类型

在使用enum的时候，推荐适用最新的fixed underlying type(WWDC 2012 session 405- Modern Objective-C)规范，因为它具备更强的类型检查和代码完成功能。

```objc
如：
typedef NS_ENUM(NSInteger, NYTAdRequestState) {
    NYTAdRequestStateInactive,
    NYTAdRequestStateLoading
};
```

### 布尔变量

因为nil将被解析为NO，因此没有必要在条件语句中进行比较。永远不要将任何东西和YES进行直接比较，因为YES被定义为1，而一个BOOL变量可以有8个字节。

```objc
如：
if (!someObject) {
}

if (isAwesome)

if (![someObject boolValue])
 
不恰当：
if (someObject == nil) {
}

if ([someObject boolValue] == NO)

if (isAwesome == YES) // Never do this.
```

如果一个BOOL属性使用形容词来表达，属性将忽略’is’前缀，但会强调惯用名称。
例如：

```objc
@property (assign, getter=isEditable) BOOL editable;
```

### 单例
在创建单例对象的共享实例时，应使用线程安全模式。

```objc
如：
(instancetype)sharedInstance {
   static id sharedInstance = nil;
 
   static dispatch_once_t onceToken;
   dispatch_once(&onceToken, ^{
      sharedInstance = [[self alloc] init];
   });
 
   return sharedInstance;
}
```

### 数字

尽量避免数字固定为某个类型如:写5而不写5.0，5.3而不写5.3f)

### 代码组织

用 `#pragma mark`来将方法分类，这个非常有效，当你用快捷键`Control+6`可以高效的寻找到自己想要跳转到的方法

```objc
//#pragma mark Properties

@dynamic someProperty;

- (void)setCustomProperty:(id)value {}

#pragma mark Lifecycle

+ (id)objectWithThing:(id)thing {}
- (id)init {}

#pragma mark Drawing

#pragma mark UItableView delegate

#pragma mark UITableView datasource

- (void)drawRect:(CGRect) {}

#pragma mark Another functional grouping

#pragma mark GHSuperclass

- (void)someOverriddenMethod {}

#pragma mark NSCopying

- (id)copyWithZone:(NSZone *)zone {}

#pragma mark NSObject

- (NSString *)description {}
```

### 判断语句

if和else应该和左大括号在同一行

```objc
如:
if (button.enabled) {
    // Stuff
} else if (otherButton.enabled) {
    // Other stuff
} else {
    // More stuf
}
```

Switch 也是一样

```objc
如:
switch (something.state) {
    case 0: {
        // Something
        break;
    }

    case 1: {
        // Something
        break;
    }

    case 2:
    case 3: {
        // Something
        break;
    }

    default: {
        // Something
        break;
    }
}
```

### Import

在类的头文件中需要引用其他的类的时候，需要用```@class```这个关键字，这样能减少类与类之间的依赖。

```objc
如:
store.h:

#import <Foundation/Foundation.h>
#import <CoreData/CoreData.h>

@class User;

@interface Store : NSManagedObject

@property (nonatomic, retain) User *user;

@end

Store.m:

#import "Store.h"

#import "User.h"

@implementation Store

//doSomething

@end
```

### 私有方法

关于私有方法，苹果官方建议呢是是使用统一的前缀来分组和辨识。但是千万不用用下划线来作为前缀来命名，或者你可以在前缀前添加一些特殊的标识。


git commit 格式
---

可以参考[google开源项目的Commit 规范](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#)

参考规范:

1. **添加了新功能** ``` feat(大模块+子模块):#例如实现了某个具体功能 ```
2. **修改了某些功能**  ``` change(大模块+子模块):#例如修改了某个具体的功能 ```
3. **修复了某个Bug**  ``` fix_bug(大模块+子模块):#修复了什么的bug，最好写上*原因*和*解决方法*。```
4. **比较大的改动**
```
   broken_change():
   before: 
   
   after:
```
   
5. **修改了文档** ```docs():```
6. **修改了格式** ```style():```
7. **添加了一些测试** ```test(大模块+子模块):```
8. **一些杂项，比如像解决工程编译错误等问题之后的修改** ```chore():```
9. **对又有代码的重构** ```refactor(大模块+子模块): ```

格式
---
一行代码的长度不能超过80个字母

**函数声明和定义:**

“-”或“+”和返回类型之间应该有一个空格

```objc
如:
- (void)doSomethingWithString:(NSString *)theString {
  ...
}
```
方法大括号和其它大括号（比如if/else/switch/while等等）应在语句的同一行开始，而在新的一行关闭。

```objc
if (user.isHappy) {
//Do something
}
else {
//Do something else
}
```

当有多个参数的时，如果参数太多超过一行，则应该将每个参数分行，并且冒号对齐

```objc
如:
- (void)doSomethingWith:(GTMFoo *)theFoo
                   rect:(NSRect)theRect
               interval:(float)theInterval {
  ...
}
```

当第一个参数短于其他参数的时候，分行时每行至少要缩进4个空格

```objc
             longKeyword:(NSRect)theRect
       evenLongerKeyword:(float)theInterval
                   error:(NSError **)theError {
          ...
}
```
### 方法调用

方法调用应该和方法声明的时候一个格式,要么所有参数放在一行里:

```objc
[myObject doFooWith:arg1 name:arg2 error:arg3];
```
要么每一行一个参数:

```objc
如:
[myObject doFooWith:arg1
               name:arg2
              error:arg3];
```

当第一个参数短语其他参数的时候，分行是每行至少要缩进4个空格:

```objc
如:
[myObj short:arg1
          longKeyword:arg2
    evenLongerKeyword:arg3
                error:arg4];

```
   
@public 和 @private前面为一个空格

```objc
如:
@interface MyClass : NSObject{
 @public
 …
 @private
}
@end
```

### 协议

类型与协议名之间不要空格

```objc
如:
@interface MyProtocoledClass : NsObject<NSWindowDelegate> {
 @private
  id<MyFancyDelegate> _delegate;
}
-  (void)setDelegate:(id<MyFancyDelegate>)aDelegate;
@end
```

### Blocks

```objc
在一行的情况
如:
[operation setCompletionBlock:^{ [self onOperationDone]; }];

//block 在新的行里，需要缩进四个空格
[operation setCompletionBlock:^{
      [self.delegate newDataAvailable];
}];

dispatch_async(_fileIOQueue, ^{
      NSString* path = [self sessionFilePath];
      if (path) {
        // ...
      }
});
```

```objc
//有参数的block,|(SessionWindow *window)|与|{|之间有一个空格
如:
[[SessionService sharedService]
    loadWindowWithCompletionBlock:^(SessionWindow *window) {
        if (window) {
          [self windowDidLoad:window];
        } else {
          [self errorLoadingWindow];
        }
    }];

//block 中有参数且不能在一行中显示
//要行与行之间要相对有四个空格
[[SessionService sharedService]
      loadWindowWithCompletionBlock:
            ^(SessionWindow *window) {
                   if (window) {
                     [self windowDidLoad:window];
                   } else {
                     [self errorLoadingWindow];
                   }
            }];

//很长的block可以被定义不在一行里
void (^largeBlock)(void) = ^{
    // ...
};
[_operationQueue addOperationWithBlock:largeBlock];
```


	        



