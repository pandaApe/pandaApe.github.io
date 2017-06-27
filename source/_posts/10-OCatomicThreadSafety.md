---
title: 使用atomic修饰的属性并不会线程安全
date: 2016-06-20 21:37:41
categories: iOS
---

## 误解
一开始我们都认为 **noatomic** 非原子性，其修饰的属性是非线程安全的，而 **atomic**原子性修饰的属性是线程安全。其实事实并非如此。
被**atomic**修饰的属性，所说的线程安全，只是把setter 和getter方法中对属性进行读写时才会加锁。Objective-C的Runtime是开源的，我们可以在源代码中一窥究竟:
<!--more-->
## 从源码着手
对于一个没有自定义setter方法的属性，编译后编译器会生成一个 `objc_setProperty_non_gc(id self, SEL _cmd, ptrdiff_t offset, id newValue, BOOL atomic, signed char shouldCopy)` 的调用，源码在下面：

```C
void objc_setProperty(id self, SEL _cmd, ptrdiff_t offset, id newValue, BOOL atomic, BOOL shouldCopy) {
#if SUPPORT_GC
    (UseGC ? objc_setProperty_gc : objc_setProperty_non_gc)
#else
    objc_setProperty_non_gc
#endif
        (self, _cmd, offset, newValue, atomic, shouldCopy);
}

PRIVATE_EXTERN void objc_setProperty_non_gc(id self, SEL _cmd, ptrdiff_t offset, id newValue, BOOL atomic, BOOL shouldCopy) {
    // Retain release world
    id oldValue, *slot = (id*) ((char*)self + offset);

    // atomic or not, if slot would be unchanged, do nothing.
    if (!shouldCopy && *slot == newValue) return;
   
    if (shouldCopy) {
        newValue = (shouldCopy == OBJC_PROPERTY_MUTABLECOPY ? [newValue mutableCopyWithZone:NULL] : [newValue copyWithZone:NULL]);
    } else {
        newValue = objc_retain(newValue);
    }

    if (!atomic) {
        oldValue = *slot;
        *slot = newValue;
    } else {
        spin_lock_t *slotlock = &PropertyLocks[GOODHASH(slot)];
        _spin_lock(slotlock);
        oldValue = *slot;
        *slot = newValue;        
        _spin_unlock(slotlock);        
    }

    objc_release(oldValue);
}
```
源码的方法命名有点不习惯，但是也并不会影响理解，第23行有个是否使用**atomic**修饰的判断，我们看else中的代码。

从使用PropertyLocks数组中的一个给写操作上锁，在赋完值之后进行解锁操作。

也就是说，**atomic**只是在 setter方法 中加锁，当多个线程同时写操作时，要进行排队。A线程对属性进行写操作，B线程不可以对该属性进行写操作，只有等A线程访问结束，B线程才能操作。问题在于，当A线程再想对属性进行读操作，读取的是B线程写的数据，这就破坏了线程安全,。如果再有C线程在A线程读操作前Release这个属性，那么程序就会Crash.
综上，**atomic** 操作是原子性的，它只保证了属性的setter和getter时的线程安全，并不能保证属性线程安全，**atomic**的使用只是更好的降低的线程出错的可能性。


## 问题来了
那么问题来了，Apple为什么只给setter和getter方法加锁而不直接使用**@synchronized**加锁呢？
看看 **synchronized** 操作的[源码](https://github.com/opensource-apple/objc4/blob/master/runtime/objc-sync.mm)， 简单的一个注解，其实做了许多事情，每个**@synchronized()**代码块，使用了 objc_sync_enter、objc_sync_exit和id2data三个加解锁序列，而这种作法相比使用一个**atomic**来修饰，只给setter、getter方法加锁的方式 在性能上要慢很多。读写操作一个属性时，通常需要很快来完成，**atomic**的方式要适合这项工作。

在必要时，可以使用**@synchronized**，但是在保证多线程操作在发生错误的时候，不会发生dead lock。

## 参考链接： 
[1]:  [Hash碰撞与拒绝服务攻击 ](http://www.cnblogs.com/xuanhun/archive/2012/01/01/2309571.html)
[2]:  [Source code of objc-accessors.m file ](http://opensource.apple.com/source/objc4/objc4-493.9/runtime/Accessors.subproj/objc-accessors.m)
[3]:  [iOS多线程开发－线程安全 ](http://www.jianshu.com/p/e7e44dfb1d2b)
[4]:  [Source code of @synchronized ](https://github.com/opensource-apple/objc4/blob/master/runtime/objc-sync.mm)
[5]:  [Synchronized-swimming ](http://googlemac.blogspot.sg/2006/10/synchronized-swimming.html)
[6]:  [线程安全类的设计 ](http://objccn.io/issue-2-4/)
[7]:  [atomic一定是线程安全的吗？ ](http://m.blog.csdn.net/article/details?id=49687215)



