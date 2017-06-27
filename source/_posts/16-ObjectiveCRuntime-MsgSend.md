---
title: 【Objective-C运行时】-消息发送
date: 2016-07-25 19:29:54
categories: iOS
---

最近在找工作，运行时基本上是面试笔试必问的一道题，能考察出面试者对OC这门语言的掌握程度，及其技术深度。当然，与其相类似的问题还有 Runloop, ARC等等。我准备用一个系列来写写OC中的运行时，一方面能让自己有个系统的掌握，另一方面希望我分享出来的东西对大家有所帮助。

<!-- more -->

## 运行时
Runtime system，对OC来说运行时就相当于一个操作系统，它是OC语言运行的基础。C语言中的函数调用，在编译阶段就确定了应该调用哪个函数，编译完成时顺序执行，而OC是动态语言，函数调用是动态的消息发送，在编译阶段不会确定调用哪个具体的函数，而在运行的时候通过在对象内部查找的方法的方式来进行函数调用。我们先来理解下这几个概念。


## 消息

OC中的函数调用：
```mm
[cat eat:@"food"];
```
cat是消息的接收者(**Receiver**)，eat: 是选择子(**Selector**)，@"food"是参数(**Parameter**)。**选择子和参数一起即称为消息**。代码即可解释为向对象cat发送一条eat的消息。

## objc_msgSend
编译器在编译时会将这条语句转换为一条标准的C语言函数调用，所调用的函数是消息传递机制中的核心函数，即：**objc_msgSend**，原型：
```c
<return_type> objc_msgSend(id self, SEL _cmd, …)
```
这是一个可变参数函数。参数 self和_cmd是隐藏参数，后文会解释。
那么经过编译器转换，刚刚OC的函数调用就会转换为：

```mm
void objc_msgSend(cat,@selector(eat:),@"food");
```


## Class类
要想更深入地了解消息传递我们需要先了解NSObject类中的isa指针。我们在NSObject类中找到isa指针的声明：

```mm
@interface NSObject <NSObject> {
    Class isa  OBJC_ISA_AVAILABILITY;
}
```
追根求源，找到Class的定义：

```c
typedef struct objc_class *Class;

struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class super_class                
    const char *name                                
    long version                                             
    long info                                                
    long instance_size                                 
    struct objc_ivar_list *ivars                      
    struct objc_method_list **methodLists   
    struct objc_cache *cache                       
    struct objc_protocol_list *protocols        
#endif

} OBJC2_UNAVAILABLE;
/* Use `Class` instead of `struct objc_class *` */

```
我们对Class里的属性进行解释：

| Property |        Description   |
|:----:|:---------------------------------------|
| isa | 自己。A pointer to the class definition of which this object is an instance. |
| super_class | 指向父类，如果这个类是根类，则为NULL |
| name | 类名 |
| version | 类的版本信息，初始化默认为0| 
| info | 一些标识信息,如CLS_CLASS (0x1L) 表示该类为普通 class ，其中包含对象方法和成员变量;CLS_META (0x2L) 表示该类为 metaclass，其中包含类方法| 
| instance_size | 该类的实例变量大小(包括从父类继承下来的实例变量)| 
| ivars | 用于存储每个成员变量的地址| 
| methodLists | 存储类中的方法，不包含父类中的方法| 
| cache | 最近调用的方法列表，用于消息发送的效率| 
| protocols | 存储该类所遵守的协议| 

Runtime里的消息发送主要用到，isa，super_class，cache和methodLists

## 消息派发流程
objc_msgSend函数会依据接收者与选择子来调用适当的方法，为了完成此操作：
* **Runtime**通过调用 **object_getClass(isa)** ，找到对应的class
* 在**class**中先去**cache**中 通过SEL查找对应函数**method**
* 若**cache**中未找到，再去**methodLists**中查找
* 若**methodLists**中未找到，则通过**super_Class**找到对应的父类，再在父类的**MethodLists**中查找
* 若最终仍未找到，**Runtime**将执行消息转发或者抛出异常
* 如果在**methodLists**或者父类中找到了，**Runtime**会把**method**加入到**cache**中，并执行该方法。
￼
![](http://ww3.sinaimg.cn/large/8c0dc373gw1f6milo3uagg20980f4q32.gif)

## 隐藏参数
Objective-C中的消息发送在C语言的原型：
```c
<return_type> objc_msgSend(id self, SEL _cmd, …)
```
每一个方法会默认隐藏两个参数，self和_cmd，self代表消息接收者，_cmd代表这个方法的SEL。之所以称这两个参数是隐藏参数，是因为它们不是Objective-C源码中定义的而是在编译阶段插入的。我举个例子帮忙理解下原型中的参数列表： 

```mm
- (NSString *)name;
```

这个方法实际上有两个参数：self和_cmd。

```mm
-(void)setValue:(int)val;
```
这个方法实际上有三个参数：self, _cmd 和 val。

被指定为动态实现的方法的参数类型有如下的要求：

* 第一个参数类型必须是id（就是self的类型）
* 第二个参数类型必须是SEL（就是_cmd的类型）
* 从第三个参数起，可以按照原方法的参数类型定义。

举两个例子来说明：

例一：**setHeight:(CGFloat)height**中的参数**height**是浮点型的，所以第三个参数类型就是f。
例二：再比如**setName:(NSString *)name**中的参数**name**是字符串类型的。

虽然这两个参数没有源码中定义，但是在Xcode中的方法定义中，还是可以使用的。

```mm
- (void)demoMethod
{
    NSString* dec = self.description;
    
    NSLog(@"dec__%@___method___%@",dec,NSStringFromSelector(_cmd));
}
```

这样就晓得在平时在方法定义时写的self是哪里来的了吧。

## 总结

* 消息由选择子及参数构成。给某对象"发送消息"也就相当于在该对象上"调用方法"
发给某对象的全部消息都要由"动态消息派发系统"（**dynamic message dispatch system**）来处理。该系统会查出对应的方法，并执行其代码。
* 在OC中，如果向某对象传递消息，那就会使用动态绑定机制来决定需要调用的方法。在底层，所有方法都是普通C语言函数，然而对象收到消息后，究竟该掉哪个方法则完全于运行期决定，甚至可以在程序运行时改变，这些特性使得OC称为一门真正的动态语言。
* 在实际编写OC时，开发者应该了解其底层工作原理。代码究竟是如何执行的，而且能理解为何在调试的时候，栈信息中总是出现**objc_msgSend**。

## 参考链接
[ Effective Objective-C(第11-14条) ](http://blog.csdn.net/hherima/article/details/38425605)


