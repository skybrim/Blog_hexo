---
title: Objective-C NSString
date: 2019/1/1
tags: iOS
comments: true
---

研究了一下 Objective-C 中 NSSting 
<!--more-->

* 代码

```objectivec
// __NSCFConstantString
// retainCount (-1,实际是4294967295),
// 无法被释放
// 内存地址相同，低地址，猜测存放在常量区
// 指针地址不同
NSString *test1 = [[NSString alloc] initWithString:@"123"];
NSString *test2 = @"123";
self.testStr = [test2 copy];

// __NSCFString
// retainCount 1
// 内存地址不同，在 heap 上
// 指针地址不同，指针在 stack
NSMutableString *test3 = [[NSMutableString alloc] initWithString:@"123"];
NSString *test4 = [NSString stringWithFormat:@"11111111111111111111111111111111123"];

// NSTaggedPointerString
// 内存地址相同，地址就是值，并非真正的对象，是一个伪对象
// 指针地址不同，指针在栈上
NSString *test5 = [test3 copy];
NSString *test6 = [NSString stringWithFormat:@"123"];
NSString *test7 = [NSString stringWithFormat:@"%@", @"123"];

// 用 stringWithFormat 创建的 NSString
// 字符串短，类型是 NSTaggedPointerString， 字符串长，类型是 __NSCFString

NSLog(@"test1 class:%@ ++ 内存地址：%p ++ 指针地址：%p",[test1 class],test1,&test1);
NSLog(@"test2 class:%@ ++ 内存地址：%p ++ 指针地址：%p",[test2 class],test2,&test2);
NSLog(@"self.testStr class:%@ ++ 内存地址：%p ++ 指针地址：%p",[self.testStr class],self.testStr,&_testStr);
NSLog(@"\n");
NSLog(@"test3 class:%@ ++ 内存地址：%p ++ 指针地址：%p",[test3 class],test3,&test3);
NSLog(@"test4 class:%@ ++ 内存地址：%p ++ 指针地址：%p",[test4 class],test4,&test4);
NSLog(@"\n");
NSLog(@"test5 class:%@ ++ 内存地址：%p ++ 指针地址：%p",[test5 class],test5,&test5);
NSLog(@"test6 class:%@ ++ 内存地址：%p ++ 指针地址：%p",[test6 class],test6,&test6);
NSLog(@"test7 class:%@ ++ 内存地址：%p ++ 指针地址：%p",[test7 class],test7,&test7);
```

* 输出

```objectivec
test1 class:__NSCFConstantString ++ 内存地址：0x100df9668 ++ 指针地址：0x7ffeeee194c8
test2 class:__NSCFConstantString ++ 内存地址：0x100df9668 ++ 指针地址：0x7ffeeee194c0
self.testStr class:__NSCFConstantString ++ 内存地址：0x100df9668 ++ 指针地址：0x7f8043604978

test3 class:__NSCFString ++ 内存地址：0x600001f6fa50 ++ 指针地址：0x7ffeeee194b8
test4 class:__NSCFString ++ 内存地址：0x60000045bd40 ++ 指针地址：0x7ffeeee194b0

test5 class:NSTaggedPointerString ++ 内存地址：0xfad007b675b0e973 ++ 指针地址：0x7ffee03e34a8
test6 class:NSTaggedPointerString ++ 内存地址：0xfad007b675b0e973 ++ 指针地址：0x7ffee03e34a0
test7 class:NSTaggedPointerString ++ 内存地址：0xfad007b675b0e973 ++ 指针地址：0x7ffee03e3498
```

* 分析

test1 的创建 NSString 方式，在编译器中已经完全被字面量的创建方式取代了，编译器会自动帮你。  
test2 字面量创建 NSString  
test3 是对 test2 的指针拷贝  
这三种方式穿件的 NSString，类型都是 __NSCFConstantString，应该是放在常量区，指针存放在 stack  
内容相同的时候，内存地址也相同  

test3 NSMutableString 对象  
test4 stringWithFormat 创建，注意，其内容一定要大于 8 个字节  
test3&test4，类型是 __NSCFString，是对象，和 OC 中的普通对象基本一致，存储在 heap 中，指针在 stack  
内存地址不会相同  

test5 是对 test3 的深拷贝，从 NSMutableString 转变为 NSString  
test6 和 test7 都是 stringWithFormat 创建的短字符串，小于 8 个字节  
test5、test6、test7 ，类型都是 NSTaggedPointerString，tagged pointer 实际是一个伪对象，他的地址就是他的值，如下图可以看出，这三个并没有 isa 指针  
![isa](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/20191014112326.png)

关于 tagged pointer ，可以看唐巧大佬的文章:[深入理解 Tagged Pointer](https://www.infoq.cn/article/deep-understanding-of-tagged-pointer/)
