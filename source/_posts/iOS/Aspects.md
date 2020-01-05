---
title: Aspects 原理简析
date: 2019/1/1
tags: iOS
comments: true
---

Aspects 是使用 Objective-C 语言编写的AOP 库。在此简单的记录一下原理简析，备忘。
<!--more-->

## 简述

* **下文中，selector 指开发者想 hook 的方法，block 指开发者想执行的代码**

* aspect\_add  
保存所有开发者需要执行的 block 到 容器中  
执行 aspect\_prepareClassAndHookSelector  

* aspect\_prepareClassAndHookSelector  
动态创建子类：aspect\_hookClass  
给 selector 起了一个别名(添加了前缀 Aspects\_\_)，并将 selector 的 imp 指向了 Aspects\_\_selector  
将 \_objc\_msgForward/\_objc\_msgForward\_stret 的 imp 指向了 selector  

* aspect\_hookClass  
动态生成子类  
交换方法 aspect\_swizzleForwardInvocation  

* aspect\_swizzleForwardInvocation  
将 forwardInvocation: 的 IMP 指向了 Aspects 自定义的 IMP \_\_ASPECTS\_ARE\_BEING\_CALLED\_\_  

* \_\_ASPECTS\_ARE\_BEING\_CALLED\_\_  
根据参数 invocation ，获得 selector ，拿到 aliasSelector(即：Aspects\_\_selector) ，根据 aliasSelector 拿到 selector 的 IMP  
从容器中取出所有的 before instead after 的 block，按照顺序，依次执行，没有 instead 的 block ，就执行 aliasSelector。  

* 执行
执行代码，当调用 selector 时，其 imp 是 \_objc\_msgForward/\_objc\_msgForward\_stret ，直接触发了 forwardInvocation: ，此时 forwardInvocation: 的 imp 指向的是自定义的 \_\_ASPECTS\_ARE\_BEING\_CALLED\_\_ ，在其内执行了 block  

## 入口方法

```objectivec
+ (id<AspectToken>)aspect_hookSelector:(SEL)selector
                           withOptions:(AspectOptions)options
                            usingBlock:(id)block
                                 error:(NSError **)error;

- (id<AspectToken>)aspect_hookSelector:(SEL)selector
                           withOptions:(AspectOptions)options
                            usingBlock:(id)block
                                 error:(NSError **)error;
```

## aspect_add

```objectivec
static id aspect_add(id self, SEL selector, AspectOptions options, id block, NSError **error) {
    NSCParameterAssert(self);
    NSCParameterAssert(selector);
    NSCParameterAssert(block);

    __block AspectIdentifier *identifier = nil;
    aspect_performLocked(^{
        //过滤一些方法 . eg: delloc
        if (aspect_isSelectorAllowedAndTrack(self, selector, options, error)) {
            // 使用 lazy load 的方式，为 self 生成一个 AspectsContainer 容器
            AspectsContainer *aspectContainer = aspect_getContainerForObject(self, selector);
            //创建 AspectIdentifier 保存 block
            identifier = [AspectIdentifier identifierWithSelector:selector
                                                           object:self
                                                          options:options
                                                            block:block
                                                            error:error];
            if (identifier) {
                //将 AspectIdentifier 加入到容器中
                [aspectContainer addAspect:identifier
                               withOptions:options];
                // ****** 重要 ******
                // Modify the class to allow message interception.
                aspect_prepareClassAndHookSelector(self, selector, error);
            }
        }
    });
    return identifier;
}
```

## aspect_prepareClassAndHookSelector

```objectivec
static void aspect_prepareClassAndHookSelector(NSObject *self, SEL selector, NSError **error) {
    NSCParameterAssert(selector);
    // 动态生成了一个子类，在这个方法里，hook 了 forwardInvocation
    Class klass = aspect_hookClass(self, error);
    //根据 SEL 获取 IMP
    Method targetMethod = class_getInstanceMethod(klass, selector);
    IMP targetMethodIMP = method_getImplementation(targetMethod);
    // 检查如果目标方法的实现还不是 _objc_msgForward 或者 _objc_msgForward_stret 的话，就进行 hook
    if (!aspect_isMsgForwardIMP(targetMethodIMP)) {
        // Make a method alias for the existing method implementation, it not already copied.
        const char *typeEncoding = method_getTypeEncoding(targetMethod);
         // 给被替换的 selector 取一个别名
        SEL aliasSelector = aspect_aliasForSelector(selector);
        // 为类增加一个 SEL 为 aliasSelector， 实现为 selector 的 IMP
        if (![klass instancesRespondToSelector:aliasSelector]) {
            __unused BOOL addedAlias = class_addMethod(klass, aliasSelector, method_getImplementation(targetMethod), typeEncoding);
            NSCAssert(addedAlias, @"Original implementation for %@ is already copied to %@ on %@", NSStringFromSelector(selector), NSStringFromSelector(aliasSelector), klass);
        }
        // 在先前的动态生成子类的方法 aspect_hookClass 里面，hook 了 forwardInvocation
        // 把原 selector 的 imp 指向 _objc_msgForward 或 _objc_msgForward_stret。
        // 这样当调用原 selector 就可以直接触发消息转发，即 _objc_msgForward
        // We use forwardInvocation to hook in.
        class_replaceMethod(klass, selector, aspect_getMsgForwardIMP(self, selector), typeEncoding);
        AspectLog(@"Aspects: Installed hook for -[%@ %@].", klass, NSStringFromSelector(selector));
    }
}
```

## aspect_hookClass

```objectivec
static Class aspect_hookClass(NSObject *self, NSError **error) {
    NSCParameterAssert(self);
    Class statedClass = self.class;
    Class baseClass = object_getClass(self);
    NSString *className = NSStringFromClass(baseClass);

    // Already subclassed
    if ([className hasSuffix:AspectsSubclassSuffix]) {
        return baseClass;

        // We swizzle a class object, not a single object.
    }else if (class_isMetaClass(baseClass)) {
        return aspect_swizzleClassInPlace((Class)self);
        // Probably a KVO'ed class. Swizzle in place. Also swizzle meta classes in place.
    }else if (statedClass != baseClass) {
        return aspect_swizzleClassInPlace(baseClass);
    }

    // 动态生成 子类
    // Default case. Create dynamic subclass.
    const char *subclassName = [className stringByAppendingString:AspectsSubclassSuffix].UTF8String;
    Class subclass = objc_getClass(subclassName);

    if (subclass == nil) {
        subclass = objc_allocateClassPair(baseClass, subclassName, 0);
        if (subclass == nil) {
            NSString *errrorDesc = [NSString stringWithFormat:@"objc_allocateClassPair failed to allocate class %s.", subclassName];
            AspectError(AspectErrorFailedToAllocateClassPair, errrorDesc);
            return nil;
        }
        // ***** 重要，在这个里 hook forwardInvocation *****
        aspect_swizzleForwardInvocation(subclass);
        aspect_hookedGetClass(subclass, statedClass);
        aspect_hookedGetClass(object_getClass(subclass), statedClass);
        objc_registerClassPair(subclass);
    }

    object_setClass(self, subclass);
    return subclass;
}
```

## aspect_swizzleForwardInvocation

```objectivec
static NSString *const AspectsForwardInvocationSelectorName = @"__aspects_forwardInvocation:";
static void aspect_swizzleForwardInvocation(Class klass) {
    NSCParameterAssert(klass);
    //一般不会去实现 forwardInvocation ，所以不一定有原始 forwardInvocation 的 imp
    //Aspects把forwardInvocation的实现换成了__ASPECTS_ARE_BEING_CALLED__这个函数
    //原始的forwardInvocation实现的名字就变成了__aspects_forwardInvocation
    // If there is no method, replace will act like class_addMethod.
    IMP originalImplementation = class_replaceMethod(klass, @selector(forwardInvocation:), (IMP)__ASPECTS_ARE_BEING_CALLED__, "v@:@");
    if (originalImplementation) {
        class_addMethod(klass, NSSelectorFromString(AspectsForwardInvocationSelectorName), originalImplementation, "v@:@");
    }
    AspectLog(@"Aspects: %@ is now aspect aware.", NSStringFromClass(klass));
}
```

## \_\_ASPECTS\_ARE\_BEING\_CALLED\_\_

```objectivec
static void __ASPECTS_ARE_BEING_CALLED__(__unsafe_unretained NSObject *self, SEL selector, NSInvocation *invocation) {
    NSCParameterAssert(self);
    NSCParameterAssert(invocation);
    // 从消息转发的参数 invocation 中,得到原 selector
    SEL originalSelector = invocation.selector;
     //selector 添加前缀，
    //之前在 aspect_prepareClassAndHookSelector 方法中用新的 selector 添加了新的 method，新的 method 的 imp 是原imp
    //根据这个 selector，在容器中寻找 block，并全部执行
    SEL aliasSelector = aspect_aliasForSelector(invocation.selector);
    //将 invocation 中的 selector 也替换撑新的 selector，imp 是原imp
    invocation.selector = aliasSelector;
    //拿到存储自定义的 block 的容器，包括实例方法和类方法
    AspectsContainer *objectContainer = objc_getAssociatedObject(self, aliasSelector);
    AspectsContainer *classContainer = aspect_getContainerForClass(object_getClass(self), aliasSelector);
    //根据 invocation 拿到 info，根据 info 从容器中取出 block
    AspectInfo *info = [[AspectInfo alloc] initWithInstance:self invocation:invocation];
    NSArray *aspectsToRemove = nil;

    // 执行 AspectPositionBefore 的 block
    // Before hooks.
    aspect_invoke(classContainer.beforeAspects, info);
    aspect_invoke(objectContainer.beforeAspects, info);

    //执行原 selector 或者 Instead
    // Instead hooks.
    BOOL respondsToAlias = YES;
    if (objectContainer.insteadAspects.count || classContainer.insteadAspects.count) {
        aspect_invoke(classContainer.insteadAspects, info);
        aspect_invoke(objectContainer.insteadAspects, info);
    }else {
        Class klass = object_getClass(invocation.target);
        do {
            if ((respondsToAlias = [klass instancesRespondToSelector:aliasSelector])) {
                [invocation invoke];
                break;
            }
        }while (!respondsToAlias && (klass = class_getSuperclass(klass)));
    }

    //执行 AspectPositionAfter 的 block
    // After hooks.
    aspect_invoke(classContainer.afterAspects, info);
    aspect_invoke(objectContainer.afterAspects, info);

    // If no hooks are installed, call original implementation (usually to throw an exception)
    if (!respondsToAlias) {
        // 如果没有执行的话，好执行默认的 forwardInvocation
        invocation.selector = originalSelector;
        SEL originalForwardInvocationSEL = NSSelectorFromString(AspectsForwardInvocationSelectorName);
        if ([self respondsToSelector:originalForwardInvocationSEL]) {
            ((void( *)(id, SEL, NSInvocation *))objc_msgSend)(self, originalForwardInvocationSEL, invocation);
        }else {
            [self doesNotRecognizeSelector:invocation.selector];
        }
    }

     // 做一些额外的清理
    // Remove any hooks that are queued for deregistration.
    [aspectsToRemove makeObjectsPerformSelector:@selector(remove)];
}
```
