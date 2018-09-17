---
date: '2013-07-14' 
title: "iOS设计模式学习---装饰模式" 
slug: ios_decorator
description: "装饰模式（Decorator），动态地为一个对象添加额外的职责，是继承的替代方案，属于结构型模式。通过装饰模式扩展对象的功能比继承子类方式更灵活，使用继承子类的方式，是在编译时静态决定的，即编译时绑定，而且所有的子类都会继承相同的行为。然而，如果使用组合的方式扩展对象的行为，就可以在运行时动态地进行扩展，将来如果需要也可以动态的撤销，而不会影响原类的行为。"  
category: 
  - iOS  
tags: []
---



## 定义

装饰模式（Decorator），动态地为一个对象添加额外的职责，是继承的替代方案，属于结构型模式。通过装饰模式扩展对象的功能比继承子类方式更灵活，使用继承子类的方式，是在编译时静态决定的，即编译时绑定，而且所有的子类都会继承相同的行为。然而，如果使用组合的方式扩展对象的行为，就可以在运行时动态地进行扩展，将来如果需要也可以动态的撤销，而不会影响原类的行为。   
![图片](/media/2013-7-14.jpg)  

## 实例
 
接下来，通过Object-C来实践一下，我设想一个场景，用Decorator模式来实现一下对某个手机的GPS和蓝牙功能扩展
首先，我们需要一个手机的接口或者抽象类，我这里就用抽象类来实现，代码如下：


```objc
@interface AbstractCellPhone : NSObject
- (NSString *)callNumber;
- (NSString *)sendMessage;
@end
```

```objc
#import "AbstractCellPhone.h"

@implementation AbstractCellPhone

- (NSString *)callNumber
{
    return @"phone call somebody";
}

- (NSString *)sendMessage
{
    return @"phone send a message to somebody";
}
@end
```

AbstractCellPhone也就是结构图中的Component，然后，我再来实现Nokia和Moto的手机类，这类要继承AbstractCellPhone，也就是图中ConcreteComponent类要继承Component，实现代码如下：

```objc

#import "AbstractCellPhone.h"   
@interface NokiaPhone : AbstractCellPhone
@end

```

```objc

#import "NokiaPhone.h"

@implementation NokiaPhone

- (NSString *)callNumber
{
    return @"NokiaPhone call somebody";
}

- (NSString *)sendMessage
{
    return @"NokiaPhone send Message to Somebody";
}

@end

```

接下来我需要一个Decorator接口或者抽象类，实现代码如下：

```objc

#import "AbstractCellPhone.h"

@interface Decorator : AbstractCellPhone
{
@protected AbstractCellPhone *abstractCellPhone;
}
-(void)SetComponents:(Components*)component;
@end

```

```objc

#import "Decorator.h"

@implementation Decorator
-(void)SetComponents:(Components*)component{
    components = component;
}

- (NSString *)callNumber
{
    return components.callNumber;
}

- (NSString *)sendMessage
{
    return components.sendMessage;
}
@end

```
正如结构图中，这个Decorator即继承了AbstractCellPhone，又包含了一个私有的AbstractCellPhone的对象。这样做的意义是：Decorator类又使用了另外一个Component类。我们可以使用一个或多个Decorator对象来“装饰”一个Component对象，且装饰后的对象仍然是一个Component对象。在下来，我要实现GSP和蓝牙的功能扩展，它们要继承自Decorator，代码如下：


```objc

#import "Decorator.h"

@interface DecoratorGPS : Decorator

@end

```

```objc

#import "DecoratorGPS.h"

@implementation DecoratorGPS

- (NSString *)callNumber
{
    return [NSString stringWithFormat:@"%@ with GPS", [super callNumber]];
}

- (NSString *)sendMessage
{
    return [NSString stringWithFormat:@"%@ with GPS", [super sendMessage]];
}

@end

```
最后，用客户端程序验证一下：

```objc

int main(int argc, const char *argv[])
{
    @autoreleasepool {
        
        Components *phone = [[ConcreteComponent alloc] init];
        NSLog(@"%@",phone.callNumber);
        NSLog(@"%@",phone.sendMessage);
        ConcreteDecoratorA *GPS = [[ConcreteDecoratorA alloc] init];
        [GPS SetComponents:phone];
        NSLog(@"%@",GPS.callNumber);
        NSLog(@"%@",GPS.sendMessage);
        ConcreteDecoratorB *bluetooth = [[ConcreteDecoratorB alloc] init];
        [bluetooth SetComponents:phone];
        NSLog(@"%@",bluetooth.callNumber);
        NSLog(@"%@",bluetooth.sendMessage);
    }
    return 0;
}

```
执行结果：

```objc
 NokiaPhone call somebody
 NokiaPhone send Message to Somebody
 NokiaPhone call somebody with GPS
 NokiaPhone send Message to Somebody with GPS
 NokiaPhone call somebody with BlueTooth
 NokiaPhone send Message to Somebody with BlueTooth
 
```