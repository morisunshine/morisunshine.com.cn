---
date: '2014-03-13'
slug: objc_is_not_what_you_think_it_is
title: "Objective-C 不是你想的那样"  
description: "
Ruby 和 Objective-C 这两种语言看上去好像天南地北：一种是动态语言，另一种则是静态语言；一种是解释型语言，另一种是编译型语言；一种有简洁的语法，另一种则是有点冗长的语法。从优雅的角度来看，Ruby似乎更能给我们一种自由的编程体验，所以很多人都放弃了Objective-C。"   
category: iOS   
tags: [iOS, ruby, 译文]   
comments: true 
share: true

---

本文由[morisunshine](http://morisunshine.com/)译自["Objective-C isn't what you think it is"](http://news.rapgenius.com/Soroush-khanlou-objective-c-isnt-what-you-think-it-is-if-you-think-like-a-rubyist-annotated)。转载请注明出处！

---

&nbsp;[**消息的传递**](#message_in_a_bottle)       
&nbsp;[**变得越来越动态**](#getting_metaer_and_metaer)  
&nbsp;[**内省**](#introspection)  
&nbsp;[**现学现用**](#cashing_in)  
&nbsp;[**什么是编译器？**](#what_is_comepielur)  

Ruby 和 Objective-C 这两种语言看上去好像天南地北：一种是动态语言，另一种则是静态语言；一种是解释型语言，另一种是编译型语言；一种有简洁的语法，另一种则是有点冗长的语法。从优雅的角度来看，Ruby似乎更能给我们一种自由的编程体验，所以很多人都放弃了Objective-C。


但这是一个不幸的笑话。Objective-C其实并不像别人认为的那样是件紧身衣，它和Ruby一样都受Smalltalk影响，它拥有很多Ruby开发者都喜爱的语言功能–动态方法查找、鸭子类型、开放的类和通常情况下高度可变的runtime等这些功能在Objective-C中同样存在，即使那些不出名的技术也是一样。Objective-C的这些功能都要归功于它的IDE和编译器，但也是因为它们才使你不能自由地编写代码

但是等一下，怎么能说Objective-C是动态语言呢？难道它不是建立在C语言的基础上？

你可以在Objective-C代码中包含任何C或C++的代码，但这不意味着Objective-C仅限于C或C++代码。Objective-C中所有有意思的类操作和对象内省都是来自于一个叫Objective-C Runtime的东西。这个Objective-C Runtime可以和Ruby解释器相媲美。它包含了强大的元编程里所需要的所有重要特性。


其实C语言和Ruby一样是支持这些特性的，用`property_getAttributes`或`method_getImplementation`方法就能将selector对应到具体实现(一个selector处理一个方法)，并判断这个对象能否对这个selector做出反应，再遍历子类树。在Objective-C的众多方法中，最重要的就是`objc_msgSend`方法，是它推动了应用中的每次消息发送。

--- 
  
<a id='message_in_a_bottle' name='message_in_a_bottle'> </a>

##消息的传递

Smalltalk才是实至名归的第一种面向对象语言，它用“从一个对象发送信息给另一个对象”的新概念取代了“调用函数”的旧概念，对后面的语言发展产生了深远的影响。


你可以在Ruby中通过这样写来实现消息的发送：

```ruby

receiver.the_message argument

```

Objective-C的实现方式和Ruby的差不多：

```objc

[receiver theMessage:argument];

```

这些消息实现了鸭子类型的方式，也就是说关注的不是这个对象的类型或类本身，而是这个对象能否对一个消息做出反应。

发送消息真的是非常棒的事，但是只有当消息在传送数据时，它的价值才会被发挥地更大：

```ruby

receiver.send(:the_message, argument)

```
和

```objc

[receiver performSelector:@selector(theMessage:) 
withObject:argument];

```

正如Ruby中方法需要symbol支持一样，Objective-C中selector也需要string来支持。（在Objective-C中没有symbol。）这样就可以让你通过动态的方式使用一个方法。你甚至可以通过`NSSelectorFromString`方法来使用string创建一个selector，并在一个对象里执行它。同样的，我们可以在Ruby中也可以创建一个string或symbol，并把传给`Object#send`方法。

当然，无论是哪种语言，一旦你将一个消息发送给不能处理该消息的对象，那么默认情况下就会抛出一个异常，还会导致应用的崩溃。

当你想在调用一个方法前判断一下这个对象是否能够执行这个方法，你可以用Ruby中的`respond_to？`方法来检查：

```ruby

if receiver.respond_to? :the_message
  receiver.the_message argument
end

```

Objective-C中也有差不多的方法：

```objc

if ([receiver respondsToSelector:@selector(theMessage:)]) {
    [receiver theMessage:someThing];
}

```

---

<a id='getting_metaer_and_metaer' name='getting_metaer_and_metaer'> </a>

##变得越来越动态

如果你想在一个不能修改的类（像系统类）中添加你想要的方法，那么Objective-C里的category一定不会让你失望 -- 很像Ruby中的“开放类”。

举个例子，如果你想将Rails中的`to_sentence`方法添加到`NSArray`类中，我们只需要对`NSArray`这个类进行扩展就好了：

```objc

@interface NSArray (ToSentence)

- (NSString *)toSentence;

@end


@implementation NSArray (ToSentence)

- (NSString *)toSentence {
    if (self.count == 0) return @&quot;&quot;;
    if (self.count == 1) return [self lastObject];
    NSArray *allButLastObject = [self subarrayWithRange:NSMakeRange(0, self.count-1)];
    NSString *result = [allButLastObject componentsJoinedByString:@&quot;, &quot;];
    BOOL showComma = self.count &gt; 2;
    result = [result stringByAppendingFormat:@&quot;%@ and &quot;, showComma ? @&quot;,&quot; : @&quot;&quot;];
    result = [result stringByAppendingString:[self lastObject]];
    return result;
}

@end

```

Category是在编译的时候将方法添加到程序中 -- 让我们在runtime中动态捕捉它们怎么样？

有些消息可以嵌套数据，就像Rails的dynamic finders。Ruby通过对`method_missing` 和 `respond_to`这两个方法的重写，先匹配模式，再将新方法的定义添加到这个对象中。


Objective-C中的流程是差不多，但我们不是重写`doesNotRecognizeSelector:`方法（相当于Ruby中的`method_missing`方法），而是在`resolveClassMethod:`方法中捕捉Category添加的方法。假设我们有一个叫`+findWhere:equals:`的类方法，它可以得到property的名称和值，那么通过正则表达式就可以很容易实现找到property的名字，并通过block来注册这个selector。

```objc

+ (BOOL)resolveClassMethod:(SEL)sel {
    NSString *selectorName = NSStringFromSelector(sel);

    NSRegularExpression *regex = [NSRegularExpression regularExpressionWithPattern:@&quot;^findWhere(\\w+)Equals:$&quot; options:0 error:nil];
    NSTextCheckingResult *result = [regex firstMatchInString:selectorName options:0 range:NSMakeRange(0, selectorName.length)];
    if (result) {
        NSRange propertyNameRange = [result rangeAtIndex:1];
        NSString *propertyName = [selectorName substringWithRange:propertyNameRange];

        IMP implementation  = imp_implementationWithBlock((id) ^(id self, id arg1) {
            return [self findWhere:propertyName equals:arg1];
        });

        Class metaClass = object_getClass(self);

        class_addMethod(metaClass, sel, implementation, &quot;@@:@@&quot;);
        return YES;
    }

    return [super resolveClassMethod:sel];
}

```

这个方法的优点就是我们不需要去重写`respondsToSelector:`，因为每个在类中注册过的selector都会去调用这个方法。现在让我们调用`[RGSong findWhereTitleEquals:@“Mercy”]`。当`findWhereTitleEquals:`第一次被调用的时候，runtime并不知道这个方法，所以它会调用`resolveClassMethod:`，这时我们就将`findWhereTitleEquals:`这个方法动态添加进去，当第二次调用`findWhereTitleEquals:`的时候，因为它已经被添加过了，所以就不会再调用`resolveClassMethod:`了。

这里还有一些别的方法来实现捕捉动态方法。你可以通过重写`resolveClassMethod:` 和 `resolveInstanceMethod:`方法（就像上面的一样），可以将消息传递给不同的对象或全权接管这个“调用”，并在消息传递之前，做你想这个消息要完成的任何事。这些方法都会导致运行成本的增加，特别在`-forwardInvocation:`中会达到顶峰，在这种情况下我们必须要实例化一个对象才能去执行它们。`-forwardInvocation:`方法中默认调用`doesNotRecognizeSelector`方法，这导致了应用的频繁异常或崩溃。

---

<a id='introspection' name='introspection'> </a>

##内省  

动态方法决议并不只是像Ruby和Objective-C这样的语言的技术支持。你也可以通过在runtime中用一种有意思的方式去操作这些对象。

就像在Ruby中调用`MyClass#instance_methods`一样，你可以在Objective-C中调用`class_copyMethodList([MyClass class], &amp;numberOfMethods)`来得到一个对象中方法的列表。你还可以通过`class_copyPropertyList`方法得到一个类中property的列表，它能在你的模型中实现不可思议的内省。比如在这个`Rap Genius`应用中，我们用这个功能来将JSON中的字典映射到本地对象上。

（如果你非常喜欢Ruby中的mixin，那么Objective-C强大的动态支持也能能实现同样的效果。 Vladimir Mitrovic有一个叫`Objective-Mixin`的库，它能在runtime时将一个类中的实现复制到另一个类中。）

---

<a id='cashing_in' name='cashing_in'> </a>

##现学现用

所有的动态工具都可以用来创建像Core Data这样的东西，Core Data是一个有点像ActiveRecord的持久化对象图。在Core Data中，relationship是“有缺陷的”，也就是说他们只有在被别的对象访问时，才会被加载。每个property的accessor和mutator在runtime中都被重写（使用的就是我们上面提到的动态方法决议）。如果我们访问了一个还没有被加载的对象时，框架就会从持久性储存中动态加载这个对象并将它返回。它保持了内存的低利用率，避免了在任何一个物体被获取时，实体对象图表都要被加载到内存中这样情况的发生。

当Core Data实体中的mutator被调用时，系统会将那个对象标记为需要清理，不需要去重写每个property的getter和setter。

这就是元程序，羡慕吧！

---

<a id='what_is_comepielur' name='what_is_comepielur'> </a>

##什么是编译器？ 

很明显，Objective-C和Ruby并不是同一种语言，目前为止最大的不同就是Objective-C是一种编译型语言。

这就是这些技术中最需要注意的地方。在编译时，编译器会先确定你应用使用的每个selector是不是都在应用中。如果你处理的这个对象有类型信息，那么编译器也会检查确保这个selector在头文件有声明过，这样做就是为了防止在对象中调用未声明的selector。有些方法可以绕过这些讨厌的限制，包括关闭相关的编译警告。这里就是实践元程序化的Objective-C最好的练习。

你可以通过将selector的类型储存为不知道的类型或`id`来从对象中删除这些类型信息。因为编译器不认识这个类型，所以它只能假设你的程序可以接受发给它的任何消息（假设这些消息在应用中的其他地方被声明了，并且相关的编译标识已经打开）。

善意的忠告：如果我们关掉编译器标识和把对象保存成`id`类型，那么将会非常危险的事！其实Objective-C中最好的东西之一就是编译器（是的，比元程序还要好）。类型检查保证了我们更快的写和重构代码，也是我们在编程时少犯错误。因为没有人会关掉那些警告，所以你很难去分享你那些`id`类型的代码。大部分Objective-C开发者还是更愿意使用更强的类型而不是元程序。

事实证明Objective-C更受束缚--但因为编译器能提高更多的安全性和速度，所以我们只能选择这样并承担后果。

事实再次告诉我们，这些语言都是差不多的，Ruby开发者应该享受Objective-C，即使那些中括号让我们望而却步。
