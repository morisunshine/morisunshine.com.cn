---
date: '2015-07-18'
title: "Jazzhand开源动画框架代码分析"  
slug: jazzhand_animation
description: "Jazz Hands是IFTTT发布的一个基于关键帧的动画框架，可以用于手势，滚动视图，KVO或者ReactiveCocoa，还是非常好用的。"  
category: iOS  
tags: [iOS, 源码] 
comments: true 
share: true
---

[Jazz Hands](https://github.com/IFTTT/JazzHands)是IFTTT发布的一个基于关键帧的动画框架，可以用于手势，滚动视图，KVO或者ReactiveCocoa，还是非常好用的。

##一.原理分析

`IFTTTAnimator`是动画的执行者，但其实其中的代码量非常少，而它的重要部门就是被实时调用，而这里就体现在scrollView的delegate中的`- (void)scrollViewDidScroll:(UIScrollView *)aScrollView`方法，因为这个方法，只要scrollView滑动就会调用，所以每次被调用，就会去执行animatore中的动画，所以animator只是负责管理动画对象，和在对应位置执行动画就可以了，其他的都不用去管。
`IFTTTAnimationFrame`是动画中参数的集合，比如像位置，颜色等用于动画的参数
`IFTTTAnimationKeyFrame`是用来将这些参数和时间对应起来，也就是某一帧的动画效果效果
`IFTTTAnimation`才是对应具体某个动画的对象，他将动画对象，时间和动画参数结合起来，有了后面比较酷炫的动画效果。

接下来着重来介绍一下`IFTTTAnimation`中的方法
`- (void)addKeyFrame:(IFTTTAnimationKeyFrame *)keyFrame`将keyFrame添加到KeyFrames的数组中，为了保证时间顺序，要根据时间顺序来排列，并将每个时间段的参数都按顺序排列。
`- (IFTTTAnimationFrame *)animationFrameForTime:(NSInteger)time`是获取某一时间的keyFramte
`- (void)animate:(NSInteger)time`在某一个时刻执行相应时刻的动画，这里不同的动画效果是不一样，所以要在继承的子类被实现。

```
- (CGFloat)tweenValueForStartTime:(NSInteger)startTime endTime:(NSInteger)endTime startValue:(CGFloat)startValue endValue:(CGFloat)endValue atTime:(CGFloat)time

```

这个是比较重要的衔接作用，这个方法是计算出当前时间的动画参数的值，

```
- (IFTTTAnimationFrame *)frameForTime:(NSInteger)time startKeyFrame:(IFTTTAnimationKeyFrame *)startKeyFrame endKeyFrame:(IFTTTAnimationKeyFrame *)endKeyFrame

```
这个获取在某一个时间区间中的某个时间的动画参数

##二.分析执行过程

下面分析一段Demo中的代码：

```
// apply a 3D zoom animation to the first label
IFTTTTransform3DAnimation * labelTransform = [IFTTTTransform3DAnimation animationWithView:self.firstLabel];
IFTTTTransform3D *tt1 = [IFTTTTransform3D transformWithM34:0.03f];
IFTTTTransform3D *tt2 = [IFTTTTransform3D transformWithM34:0.3f];
tt2.rotate = (IFTTTTransform3DRotate){ -(CGFloat)(M_PI), 1, 0, 0 };
tt2.translate = (IFTTTTransform3DTranslate){ 0, 0, 50 };
tt2.scale = (IFTTTTransform3DScale){ 1.f, 2.f, 1.f };
[labelTransform addKeyFrame:[IFTTTAnimationKeyFrame keyFrameWithTime:timeForPage(0) andAlpha:1.0f]];
[labelTransform addKeyFrame:[IFTTTAnimationKeyFrame keyFrameWithTime:timeForPage(1) andTransform3D:tt1]];
[labelTransform addKeyFrame:[IFTTTAnimationKeyFrame keyFrameWithTime:timeForPage(1.5) andTransform3D:tt2]];
[labelTransform addKeyFrame:[IFTTTAnimationKeyFrame keyFrameWithTime:timeForPage(1.5) + 1 andAlpha:0.0f]];
[self.animator addAnimation:labelTransform];

```

首先动画的执行者是animator，是属于IFTTTAnimator类。
IFTTTTransform3DAnimation是一个某一动画效果对象
IFTTTTransform3D是其中的一些参数值

首先是配置动画，上面的labelTransform、tt1、tt2都是动画参数。labelTransform也是动画对象，将上面的动画参数转化成帧参数。并添加到动画对象中。使用`addKeyFrame:`的过程中，首先先将这些`keyFrame`进行排序，再用
```
- (CGFloat)tweenValueForStartTime:(NSInteger)startTime endTime:(NSInteger)endTime startValue:(CGFloat)startValue endValue:(CGFloat)endValue atTime:(CGFloat time
    
```
计算出每个位置时间对应的动画参数，添加到对应数组中。
因为`IFTTTJazzHandsViewController`是继承`IFTTTAnimatedScrollViewController`类的，而其中最重要的就是，用了scrollView中的`scrollViewDidScroll`代理方法中，操作`animator`，将他里面的动画对象都按时间来展现出来，也就是用了`- (void)animate:(NSInteger)time`方法。

##三.总结

这个开源库还是非常精简，而且思路非常清晰，依然基于Core Animation之上，因为它只是针对于UIKit上去做帧的配置，对帧的封装上更加灵活，但是缺点是实现复杂的动画时，代码量比较大。
