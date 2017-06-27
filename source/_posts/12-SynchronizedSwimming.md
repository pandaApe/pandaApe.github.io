---
title: 【译】@synchronized Swimming
date: 2016-06-24 21:25:54
categories: 翻译
---

来自谷歌Mac团队，主要介绍 **@synchronized**代码块被编译后是如何工作的，使用之后会付出哪些代价，以及如何避免使用 **@synchronized** 来实现高性能的线程安全。

<!-- more -->

## 正文
（作者按：以往有所不同的是，本文的目标是帮助Mac开发者解惑，如果你不是开发者，可以找一个开发指南，好帮助你了解我们是如何开发软件并使其稳定运行的。）

在Google，软件性能极其重要。即使是有毫秒级的提升空间，我们也会花大量时间使用性能工具和其他技术去使我们的软件更流畅。正好，最近在看多线程的代码，意识到我们在一份共享资源中错误地使用了 __@synchronized__ ：

```objc
+(id)fooFerBar:(id)bar {
@synchronized(self) {
   static NSDictionary *foo = nil;
   if (!foo) foo = [NSDictionary dictionaryWithObjects:...];
}
   return [foo objectWithKey:bar];
}
```

代码里，每次调用 **@synchronized**代码块里的 `fooFerBar` 方法时无疑都会付出数毫秒的性能代价。为什么呢？我们不可以在 `+initialize` 创建资源，因为 `fooFerBar` 是在category里的，当然在caregory里重写 `+initialize` 也不是一件很nice的事。我们也不能使用 `+load` 来做，因为其他的类在他们内部的 `+load` 中可以很容易地调用 `fooFerBar` ， 并且无法保证有顺序地加载。 因此我们唯一的选择就是尽可能少的使用 **@synchronized**，毕竟我们都不想走进那个既不出名又让人恶心的 [double-checked locking anti-pattern ](http://www.aristeia.com/Papers/DDJ_Jul_Aug_2004_revised.pdf) 。

我想知道， **@synchronized**具体是怎么工作的？有没有更好方法来实现线程安全？我“解剖”下Runtime里关于 **@synchronized**的源码，发现了这些：
```C
...
objc_sync_enter
objc_exception_try_enter
setjmp
objc_exception_extract

my actual code

objc_exception_try_exit
objc_sync_exit
...
objc_exception_throw
...
```
由此可以看出，共享资源外一个很简单 **@synchronized** 其实做了大量的初始化和内存销毁操作。在这种情况下，我们不考虑异常安全，通过[ Objective-C documentation on exception handling and thread synchronization ](http://developer.apple.com/documentation/Cocoa/Conceptual/ObjectiveC/Articles/chapter_4_section_9.html), 我们知道**@synchronized**不但会做加锁操作，而且这个锁还是一个对这种特殊用法有害的递归锁。

在[ code (ADC registration required) ](http://www.opensource.apple.com/darwinsource/10.4.7.ppc/objc4-267.1/runtime/objc-sync.m) 中看到 `objc_sync_enter` 和 `objc_sync_exit` 具体实现，我们不难看到在每个 `@synchronized(foo)` ，事实上都要建立3个加锁/解锁序列， 叫`id2data`的`objc_sync_enter`，负责从系统中取锁，并给 `foo` 加锁。序列 `objc_sync_exit` 也叫id2data,负责给`foo`解锁。而`id2data`本身内部数据结构为了安全地得到与`foo`联系的锁也必须进行加锁和解锁操作。

我们需要一个更好的办法。 该到返璞归真的时候了，甩走 `@synchronized` 调用，封装一个我们自己的 `pthread` 来代替：
```C

#include <pthread.h>

+(id)fooFerBar:(id)bar {
   static pthread_mutex_t mtx = PTHREAD_MUTEX_INITIALIZER;
   if (pthread_mutex_lock(&mtx)) {
      printf("lock failed sigh...");
      exit(-1);
}
   static NSDictionary *foo = nil;
   if (!foo) foo = [NSDictionary dictionaryWithObjects:...];
   if (pthread_mutex_unlock(&mtx) != 0)) {
      printf("unlock failed sigh...");
      exit(-1);
}
   return [foo objectWithKey:bar];
}
```
虽然看起来很丑，但是在性能上已经有了重大的提升，达到了我们期望的结果。看上面的代码，我们已经很好的避免了异常阻塞，两个访问锁，和一坨乱七八糟的辅助代码。

至此，我们已经实现了我们代码更快的目标，并且它们运行得还不错，但是，还有没有其他更简洁的办法了呢？毕竟如果代码更简洁，也就意味着会有更少的隐藏bug。

## 原文链接
[@synchronized-swimming ](http://googlemac.blogspot.sg/2006/10/synchronized-swimming.html)
