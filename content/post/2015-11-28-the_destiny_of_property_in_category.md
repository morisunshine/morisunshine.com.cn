---
date: '2015-11-28'
title: Category中property的命运
slug: the_destiny_of_property_in_category
category: iOS
tags: [iOS, runtime]
description: "在category中，我们可以添加我们需要的类方法和实例方法，并且可以在其中使用需要扩展的类中的实例变量，但是我们在category中添加property是不提倡，这似乎已经成为iOS开发中的常识，但是今天我想带着这个问题来进行一下研究，为什么苹果不提倡在category中添加property？"
comments: true 
share: true

---

##前言

在category中，我们可以添加我们需要的类方法和实例方法，并且可以在其中使用需要扩展的类中的实例变量，但是我们在category中添加property是不提倡，这似乎已经成为iOS开发中的常识，但是今天我想带着这个问题来进行一下研究，为什么苹果不提倡在category中添加property？

首先我们可以在[iOS的文档](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/CustomizingExistingClasses/CustomizingExistingClasses.html)中，找到一句这样的话来说明原因：

>Categories can be used to declare either instance methods or class methods but are not usually suitable for declaring additional properties. It’s valid syntax to include a property declaration in a category interface, but it’s not possible to declare an additional instance variable in a category. This means the compiler won’t synthesize any instance variable, nor will it synthesize any property accessor methods. You can write your own accessor methods in the category implementation, but you won’t be able to keep track of a value for that property unless it’s already stored by the original class.
```

翻译过来大概的意思是：

>category可以被用来申明实例方法或者类方法，但是我们在其中添加额外的property一般是不合适的，虽然你我们在接口中申明一个property是符合语法的，但是这里不可能声明一个额外的实例变量。这就意味着编译器不会合成任何实例变量，也不会合成任何property的访问方法，你可以在代码中实现自己的访问方法，但是你还是不能跟踪到这个property的值除非在原始的类中就有已经保存了。

从文档中我们可以大概知道原因，因为在category中申明的propery，编译器不会生成property的访问方法和实例变量，似乎问题到这里已经找到了答案，但是我们不经要问，为什么不能生成变量呢？如何我们想要在category中实现一个property，我们应该怎么做呢？带着这些问题我们继续往下看。

##问责编译器

既然一切都是编译器的错，那就让我们看看，编译器到底对category中的property做了什么事情？

举个栗子看，定义下面一个类和它的category，实现忽略，保存为Shylock.h和Shylock.m

```

@interface Shylock : NSObject

@property (nonatomic, copy) NSString *startTime;

@end

@interface Shylock (Watson)

@property (nonatomic, copy) NSString *endTime;

@end

```

使用clang的重写命令：

`clang -rewrite-objc sark.m`

同级目录下会生成Shylock.cpp，这就是objc代码重写成c++(基本就是c)的实现。

在这么多代码中，我们只能通过搜索关键字来查找了，首先我们来搜索，startTime，有涉及到的代码有这么多

```
extern "C" unsigned long OBJC_IVAR_$_Shylock$_startTime;
struct Shylock_IMPL {
    struct NSObject_IMPL NSObject_IVARS;
    NSString *_startTime;
};

static NSString * _I_Shylock_startTime(Shylock * self, SEL _cmd) { return (*(NSString **)((char *)self + OBJC_IVAR_$_Shylock$_startTime)); }
extern "C" __declspec(dllimport) void objc_setProperty (id, SEL, long, id, bool, bool);

static void _I_Shylock_setStartTime_(Shylock * self, SEL _cmd, NSString *startTime) { objc_setProperty (self, _cmd, __OFFSETOFIVAR__(struct Shylock, _startTime), (id)startTime, 0, 1); }

extern "C" unsigned long int OBJC_IVAR_$_Shylock$_startTime __attribute__ ((used, section ("__DATA,__objc_ivar"))) = __OFFSETOFIVAR__(struct Shylock, _startTime);

static struct /*_ivar_list_t*/ {
    unsigned int entsize;  // sizeof(struct _prop_t)
    unsigned int count;
    struct _ivar_t ivar_list[1];
} _OBJC_$_INSTANCE_VARIABLES_Shylock __attribute__ ((used, section ("__DATA,__objc_const"))) = {
    sizeof(_ivar_t),
    1,
    { {(unsigned long int *)&OBJC_IVAR_$_Shylock$_startTime, "_startTime", "@\"NSString\"", 3, 8} }
};


static struct /*_method_list_t*/ {
    unsigned int entsize;  // sizeof(struct _objc_method)
    unsigned int method_count;
    struct _objc_method method_list[2];
} _OBJC_$_INSTANCE_METHODS_Shylock __attribute__ ((used, section ("__DATA,__objc_const"))) = {
    sizeof(_objc_method),
    2,
    { {(struct objc_selector *)"startTime", "@16@0:8", (void *)_I_Shylock_startTime},
    {(struct objc_selector *)"setStartTime:", "v24@0:8@16", (void *)_I_Shylock_setStartTime_} }
};

static struct /*_prop_list_t*/ {
    unsigned int entsize;  // sizeof(struct _prop_t)
    unsigned int count_of_properties;
    struct _prop_t prop_list[1];
} _OBJC_$_PROP_LIST_Shylock __attribute__ ((used, section ("__DATA,__objc_const"))) = {
    sizeof(_prop_t),
    1,
    { {"startTime","T@\"NSString\",C,N,V_startTime"} }
};

static struct _class_ro_t _OBJC_CLASS_RO_$_Shylock __attribute__ ((used, section ("__DATA,__objc_const"))) = {
    0, __OFFSETOFIVAR__(struct Shylock, _startTime), sizeof(struct Shylock_IMPL), 
    (unsigned int)0, 
    0, 
    "Shylock",
    (const struct _method_list_t *)&_OBJC_$_INSTANCE_METHODS_Shylock,
    0, 
    (const struct _ivar_list_t *)&_OBJC_$_INSTANCE_VARIABLES_Shylock,
    0, 
    (const struct _prop_list_t *)&_OBJC_$_PROP_LIST_Shylock,
};

```
###让我们看看编译器对startTime(类中的property)做了什么
在`_ivar_list_t`中添加了`_startTime变量`
在`_method_list_t`中添加了`startTime`和`setStartTime`两个方法
在`_prop_list_t`中添加了`startTime`这个property


###而关于endTime(category中的property)呢?

```
static struct /*_prop_list_t*/ {
    unsigned int entsize;  // sizeof(struct _prop_t)
    unsigned int count_of_properties;
    struct _prop_t prop_list[1];
} _OBJC_$_PROP_LIST_Shylock_$_Watson __attribute__ ((used, section ("__DATA,__objc_const"))) = {
    sizeof(_prop_t),
    1,
    { {"endTime","T@\"NSString\",C,N"} }
};

```
只是在category中的`_prop_list_t`中增加了endTime这个property，寥寥几笔带过，真是为category中的property感到心寒呀，果然和文档中说的一样，编译器并没有生成任何访问方法和实例变量。

但是我们还是不死心，让我们自己实现一下访问方法后，编译器会不会有所心动呢。

下面我们来添加Getter和Setter方法，再通过clang的重写命令，发现只是在category中的方法列表中添加这两个方法

```
static struct /*_method_list_t*/ {
    unsigned int entsize;  // sizeof(struct _objc_method)
    unsigned int method_count;
    struct _objc_method method_list[2];
} _OBJC_$_CATEGORY_INSTANCE_METHODS_Shylock_$_Watson __attribute__ ((used, section ("__DATA,__objc_const"))) = {
    sizeof(_objc_method),
    2,
    { {(struct objc_selector *)"setEndTime:", "v24@0:8@16", (void *)_I_Shylock_Watson_setEndTime_},
    {(struct objc_selector *)"endTime", "@16@0:8", (void *)_I_Shylock_Watson_endTime} }
};
```

到了这一步，我们可以发现编译器对category中的property是非常偏心的。   

其实我们可以看看runtime下的`category_t`结构体

```
struct category_t {
    const char *name;
    classref_t cls;
    struct method_list_t *instanceMethods;
    struct method_list_t *classMethods;
    struct protocol_list_t *protocols;
    struct property_list_t *instanceProperties;
};
```

而`_class_ro_t`的结构体

```
struct _class_ro_t {
    unsigned int flags;
    unsigned int instanceStart;
    unsigned int instanceSize;
    unsigned int reserved;
    const unsigned char *ivarLayout;
    const char *name;
    const struct _method_list_t *baseMethods;
    const struct _objc_protocol_list *baseProtocols;
    const struct _ivar_list_t *ivars;
    const unsigned char *weakIvarLayout;
    const struct _prop_list_t *properties;
};

```
我们发现在categoyr_t本身就少了`ivars`这样的数组变量，这也不能怪编译器，后面我们再讨论一下为什么category中不添加变量列表呢？

##改造Property

既然编译器不帮他，那让我们看看还有谁可以帮助的呢。

既然我们能帮他实现访问方法，那我们能不能给他添加一个变量呢，于是很多人就会想到用runtime中的`Associated Objects`也就是关联对象，这里看一下[nshipster](http://nshipster.com/associated-objects/)对它的介绍，这里我们从代码出发，在runtime的源码中看看相关代码，我们先看看`objc_getAssociatedObject`这个方法里做了什么，
我们可以看到最后调用的是
```
void _object_get_associative_reference(id object, void *key, id value, uintptr_t policy)
```
这个方法

```
id value = nil;
    uintptr_t policy = OBJC_ASSOCIATION_ASSIGN;
    {
        AssociationsManager manager;
        AssociationsHashMap &associations(manager.associations());
        disguised_ptr_t disguised_object = DISGUISE(object);
        AssociationsHashMap::iterator i = associations.find(disguised_object);
        if (i != associations.end()) {
            ObjectAssociationMap *refs = i->second;
            ObjectAssociationMap::iterator j = refs->find(key);
            if (j != refs->end()) {
                ObjcAssociation &entry = j->second;
                value = entry.value();
                policy = entry.policy();
                if (policy & OBJC_ASSOCIATION_GETTER_RETAIN) ((id(*)(id, SEL))objc_msgSend)(value, SEL_retain);
            }
        }
    }
    if (value && (policy & OBJC_ASSOCIATION_GETTER_AUTORELEASE)) {
        ((id(*)(id, SEL))objc_msgSend)(value, SEL_autorelease);
    }
    return value;

```

大概思路是通过维护Map，通过对象来生成一个唯一的 unsigned long 的变量来作为横坐标，查找到之后，再通过key做纵坐标去查找，这样就能找到对应的变量，也就是说只要在`某个对象`中key是唯一的，就能设置和获取对应的变量，这样就与class无关。

同样的思路，
1. 我们也可以在需要category的类中，添加一个字典或者其他映射表，来作为映射，来提供后面的扩展。
2. 也可以通过一个常住于内存的映射表对象来实现类似与_object_set_associative_reference这个的功能，

##延伸的问题：为什么不能在category中添加变量呢？

参考这个[答案](http://stackoverflow.com/a/21035984/2194236)中的一个评论，我认为的原因是这样的，category是在runtime时被添加到class中的，这里可以阅读sunnyxx的[objc category的秘密](http://blog.sunnyxx.com/2014/03/05/objc_category_secret/),也就是说这个时候class已经被注册成功，storage layout也已经确定，这时候category中再添加实例变量，对原来的storage layout并没有用。就像是在`class_addIvar`只能添加在`objc_allocateClassPair`之后，和`objc_registerClassPair`之前，而在`objc_registerClassPair`添加的变量，是不能保留的。