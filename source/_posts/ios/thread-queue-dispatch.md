---
title: iOS 线程、队列与派发
date: 2019/1/1
tags: iOS
comments: true
---

iOS 线程、队列与派发
<!--more-->

## 基本概念

* 进程

    进程，指一个正在运行的程序的实例。
    
    进程内部拥有独立的内存空间，用来存放程序的正文、数据以及打开的文件等。进程是线程的容器，是资源管理的单位。
    
* 线程

    线程，是操作系统直接支持的执行单元，CPU调度的基本单位，是代码执行的路径；一个进程内至少包含一个线程。
    
    线程只拥有程序的少量资源，但是他与进程内的其他线程共享进程的整个内存空间；线程的切换速度远快于进程。
    
* 串行(Serial)

    串行，是指多个任务时，各个任务按顺序执行，完成一个之后才能进行下一个。
    
* 并发(Concurrent)

    并发,是一个物理CPU在若干道程序之间多路复用，是对有限物理资源强制行使多用户共享以提高效率。
    
    并发的实质，就是任务数量多于CPU核心数量，操作系统通过各种任务调度算法，实现多个任务一起执行。
    
* 队列

    队列是用于保存以及管理任务的，队列根据情况，调度线程来执行其中的任务。
    
    队列分为**串行队列**和**并发队列**；队列都是先进先出的。
    
    - 串行队列
    
        串行队列，需要等待前一个任务执行完成才能执行下个任务。
        
        串行队列只需要一个线程就可以完成任务的派发。
        
    - 并发队列
    
        并发队列，允许多个任务同时执行；
        
        并发队列，任务开始执行的时间是按照顺序的，但是任务结束时间并不确定。
        
        并发队列的并发执行，需要通过新建线程来实现
        
    - 主队列
    
        主队列，是一种的特殊的串行队列；
        
        主队列的任务只能在主线程执行，并且需要等待主线程的 Runloop 空闲时才能派发。
        
* 派发

    派发，分为**同步派发**和**异步派发**。
    
    - 同步派发
    
        同步派发任务，阻塞当前线程，并等待，直到任务执行完返回。
        
    - 异步派发
    
        异步派发任务，不会阻塞当前线程，也不等待任务返回。
    
    
## 实验

### 设计方案

**注意，代码是默认执行在主线程的**

两个线程：**主线程**  **子线程**

三个队里：**主队列**  **串行队列**  **并发队列**

两种派发：**同步派发**  **异步派发**

将上述进行组合，共有 2 x 3 x 2 = 12 种情况

```objc
// 通用方法，输出当前线程
- (void)printCurrentThread {
    NSThread *currentThread = [NSThread currentThread];
    NSLog(@"cur = %@", currentThread);
}
```
### 实验 1：在主线程中派发任务

* 在 **主线程** 向 **主队列** 执行 **同步派发**

```objc
    // 情景 1：
    // 在 **主线程** 向 **主队列** 执行 **同步派发**
    // 结果：
    // 程序崩溃
    NSLog(@"--start--");
    dispatch_queue_t mainQueue = dispatch_get_main_queue();
    dispatch_sync(mainQueue, ^{
        NSLog(@"task1--start");
        [self printCurrentThread];
        sleep(3);
        NSLog(@"task1--end");
    });
    dispatch_sync(mainQueue, ^{
        NSLog(@"task2--start");
        [self printCurrentThread];
        NSLog(@"task2--end");
    });
    NSLog(@"--end--");
```

* 在 **主线程** 向 **主队列** 执行 **异步派发**

```objc
    // 情景 2：
    // 在 **主线程** 向 **主队列** 执行 **异步派发**
    // 结果：
    /*
     14:51:12.     --start--
     14:51:12.     --end--
     14:51:12.     task1--start
     14:51:12.     cur = **NSThread: 0x600000330100**{number = 1, name = main}
     14:51:15.     task1--end
     14:51:15.     task2--start
     14:51:15.     cur = **NSThread: 0x600000330100**{number = 1, name = main}
     14:51:15.     task2--end
     */
    NSLog(@"--start--");
    dispatch_queue_t mainQueue = dispatch_get_main_queue();
    dispatch_async(mainQueue, ^{
        NSLog(@"task1--start");
        [self printCurrentThread];
        sleep(3);
        NSLog(@"task1--end");
    });
    dispatch_async(mainQueue, ^{
        NSLog(@"task2--start");
        [self printCurrentThread];
        NSLog(@"task2--end");
    });
    NSLog(@"--end--");
```

* 在 **主线程** 向 **串行队列** 执行 **同步派发**

```objc
    // 情景 3：
    // 在 **主线程** 向 **串行队列** 执行 **同步派发**
    // 结果
    /*
     15:10:51.     --start--
     15:10:51.     task1--start
     15:10:51.     cur = **NSThread: 0x600001340300**{number = 1, name = main}
     15:10:54.     task1--end
     15:10:54.     task2--start
     15:10:54.     cur = **NSThread: 0x600001340300**{number = 1, name = main}
     15:10:54.     task2--end
     15:10:54.     --end--
     */
    NSLog(@"--start--");
    dispatch_queue_t serialQueue = dispatch_queue_create("serialQueue",
                                                       DISPATCH_QUEUE_SERIAL);
    dispatch_sync(serialQueue, ^{
        NSLog(@"task1--start");
        [self printCurrentThread];
        sleep(3);
        NSLog(@"task1--end");
    });
    dispatch_sync(serialQueue, ^{
        NSLog(@"task2--start");
        [self printCurrentThread];
        NSLog(@"task2--end");
    });
    NSLog(@"--end--");
```

* 在 **主线程** 向 **串行队列** 执行 **异步派发**

```objc
    // 情景 4：
    // 在 **主线程** 向 **串行队列** 执行 **异步派发**
    /*
     15:13:03.     --start--
     15:13:03.     --end--
     15:13:03.     task1--start
     15:13:03.     cur = **NSThread: 0x6000035ecc80**{number = 6, name = (null)}
     15:13:06.     task1--end
     15:13:06.     task2--start
     15:13:06.     cur = **NSThread: 0x6000035ecc80**{number = 6, name = (null)}
     15:13:06.     task2--end
     */
    NSLog(@"--start--");
    dispatch_queue_t serialQueue = dispatch_queue_create("serialQueue",
                                                       DISPATCH_QUEUE_SERIAL);
    dispatch_async(serialQueue, ^{
        NSLog(@"task1--start");
        [self printCurrentThread];
        sleep(3);
        NSLog(@"task1--end");
    });
    dispatch_async(serialQueue, ^{
        NSLog(@"task2--start");
        [self printCurrentThread];
        NSLog(@"task2--end");
    });
    NSLog(@"--end--");
```

* 在 **主线程** 向 **并发队列** 执行 **同步派发**

```objc
    // 情景 5：
    // 在 **主线程** 向 **并发队列** 执行 **同步派发**
    /*
     15:20:01.    --start--
     15:20:01.    task1--start
     15:20:01.    cur = **NSThread: 0x60000000c380**{number = 1, name = main}
     15:20:04.    task1--end
     15:20:04.    task2--start
     15:20:04.    cur = **NSThread: 0x60000000c380**{number = 1, name = main}
     15:20:04.    task2--end
     15:20:04.    --end--
     */
    NSLog(@"--start--");
    dispatch_queue_t concurrentQueue = dispatch_queue_create("concurrentQueue",
                                                       DISPATCH_QUEUE_CONCURRENT);
    dispatch_sync(concurrentQueue, ^{
        NSLog(@"task1--start");
        [self printCurrentThread];
        sleep(3);
        NSLog(@"task1--end");
    });
    dispatch_sync(concurrentQueue, ^{
        NSLog(@"task2--start");
        [self printCurrentThread];
        NSLog(@"task2--end");
    });
    NSLog(@"--end--");
```

* 在 **主线程** 向 **并发队列** 执行 **异步派发**

```objc
    // 情景 6：
    // 在 **主线程** 向 **并发队列** 执行 **异步派发**
    /*
     15:17:35.     --start--
     15:17:35.     --end--
     15:17:35.     task1--start
     15:17:35.     task2--start
     15:17:35.     cur = **NSThread: 0x6000004d4ec0**{number = 3, name = (null)}
     15:17:35.     cur = **NSThread: 0x6000004d83c0**{number = 4, name = (null)}
     15:17:35.     task2--end
     15:17:38.     task1--end
     */
    NSLog(@"--start--");
    dispatch_queue_t concurrentQueue = dispatch_queue_create("concurrentQueue",
                                                       DISPATCH_QUEUE_CONCURRENT);
    dispatch_async(concurrentQueue, ^{
        NSLog(@"task1--start");
        [self printCurrentThread];
        sleep(3);
        NSLog(@"task1--end");
    });
    dispatch_async(concurrentQueue, ^{
        NSLog(@"task2--start");
        [self printCurrentThread];
        NSLog(@"task2--end");
    });
    NSLog(@"--end--");
```

### 实验 2：在子线程派发任务

* 在 **其他线程** 向 **主队列** 执行 **同步派发**

```objc
    // 情景 7：
    // 在 **其他线程** 向 **主队列** 执行 **同步派发**
    /*
     15:34:08    --start--
     15:34:08    cur = **NSThread: 0x600001019cc0**{number = 7, name = (null)}
     15:34:08    task1--start
     15:34:08    cur = **NSThread: 0x60000105c980**{number = 1, name = main}
     15:34:11    task1--end
     15:34:11    task2--start
     15:34:11    cur = **NSThread: 0x60000105c980**{number = 1, name = main}
     15:34:11    task2--end
     15:34:11    --end--
     */
    [NSThread detachNewThreadWithBlock:^{
        NSLog(@"--start--");
        [self printCurrentThread];
        dispatch_queue_t mainQueue = dispatch_get_main_queue();
        dispatch_sync(mainQueue, ^{
            NSLog(@"task1--start");
            [self printCurrentThread];
            sleep(3);
            NSLog(@"task1--end");
        });
        dispatch_sync(mainQueue, ^{
            NSLog(@"task2--start");
            [self printCurrentThread];
            NSLog(@"task2--end");
        });
        NSLog(@"--end--");
    }];
```

* 在 **其他线程** 向 **主队列** 执行 **异步派发**

```objc
    // 情景 8：
    // 在 **其他线程** 向 **主队列** 执行 **异步派发**
    /*
     15:35:34    --start--
     15:35:34    cur = **NSThread: 0x60000260ba80**{number = 6, name = (null)}
     15:35:34    --end--
     15:35:34    task1--start
     15:35:34    cur = **NSThread: 0x600002608080**{number = 1, name = main}
     15:35:37    task1--end
     15:35:37    task2--start
     15:35:37    cur = **NSThread: 0x600002608080**{number = 1, name = main}
     15:35:37    task2--end
     */
    [NSThread detachNewThreadWithBlock:^{
        NSLog(@"--start--");
        [self printCurrentThread];
        dispatch_queue_t mainQueue = dispatch_get_main_queue();
        dispatch_async(mainQueue, ^{
            NSLog(@"task1--start");
            [self printCurrentThread];
            sleep(3);
            NSLog(@"task1--end");
        });
        dispatch_async(mainQueue, ^{
            NSLog(@"task2--start");
            [self printCurrentThread];
            NSLog(@"task2--end");
        });
        NSLog(@"--end--");
    }];
```

* 在 **其他线程** 向 **串行队列** 执行 **同步派发**

```objc
    // 情景 9：
    // 在 **其他线程** 向 **串行队列** 执行 **同步派发**
    /*
     15:39:10.    --start--
     15:39:10.    cur = **NSThread: 0x60000181d840**{number = 8, name = (null)}
     15:39:10.    task1--start
     15:39:10.    cur = **NSThread: 0x60000181d840**{number = 8, name = (null)}
     15:39:13.    task1--end
     15:39:13.    task2--start
     15:39:13.    cur = **NSThread: 0x60000181d840**{number = 8, name = (null)}
     15:39:13.    task2--end
     15:39:13.    --end--
     */
    [NSThread detachNewThreadWithBlock:^{
        NSLog(@"--start--");
        [self printCurrentThread];
        dispatch_queue_t serialQueue = dispatch_queue_create("serialQueue",
                                                           DISPATCH_QUEUE_SERIAL);
        dispatch_sync(serialQueue, ^{
            NSLog(@"task1--start");
            [self printCurrentThread];
            sleep(3);
            NSLog(@"task1--end");
        });
        dispatch_sync(serialQueue, ^{
            NSLog(@"task2--start");
            [self printCurrentThread];
            NSLog(@"task2--end");
        });
        NSLog(@"--end--");
    }];
```

* 在 **其他线程** 向 **串行队列** 执行 **异步派发**

```objc
    // 情景 10：
    // 在 **其他线程** 向 **串行队列** 执行 **异步派发**
    /*
     15:40:04.    --start--
     15:40:04.    cur = **NSThread: 0x6000000c8e80**{number = 7, name = (null)}
     15:40:04.    --end--
     15:40:04.    task1--start
     15:40:04.    cur = **NSThread: 0x6000000ced80**{number = 6, name = (null)}
     15:40:07.    task1--end
     15:40:07.    task2--start
     15:40:07.    cur = **NSThread: 0x6000000ced80**{number = 6, name = (null)}
     15:40:07.    task2--end
     */
    [NSThread detachNewThreadWithBlock:^{
        NSLog(@"--start--");
        [self printCurrentThread];
        dispatch_queue_t serialQueue = dispatch_queue_create("serialQueue",
                                                           DISPATCH_QUEUE_SERIAL);
        dispatch_async(serialQueue, ^{
            NSLog(@"task1--start");
            [self printCurrentThread];
            sleep(3);
            NSLog(@"task1--end");
        });
        dispatch_async(serialQueue, ^{
            NSLog(@"task2--start");
            [self printCurrentThread];
            NSLog(@"task2--end");
        });
        NSLog(@"--end--");
    }];
```

* 在 **其他线程** 向 **并发队列** 执行 **同步派发**

```objc
    // 情景 11：
    // 在 **其他线程** 向 **并发队列** 执行 **同步派发**
    /*
     15:41:12.    --start--
     15:41:12.    cur = **NSThread: 0x600002defb80**{number = 7, name = (null)}
     15:41:12.    task1--start
     15:41:12.    cur = **NSThread: 0x600002defb80**{number = 7, name = (null)}
     15:41:15.    task1--end
     15:41:15.    task2--start
     15:41:15.    cur = **NSThread: 0x600002defb80**{number = 7, name = (null)}
     15:41:15.    task2--end
     15:41:15.    --end--
     */
    [NSThread detachNewThreadWithBlock:^{
        NSLog(@"--start--");
        [self printCurrentThread];
        dispatch_queue_t concurrentQueue = dispatch_queue_create("concurrentQueue",
                                                           DISPATCH_QUEUE_CONCURRENT);
        dispatch_sync(concurrentQueue, ^{
            NSLog(@"task1--start");
            [self printCurrentThread];
            sleep(3);
            NSLog(@"task1--end");
        });
        dispatch_sync(concurrentQueue, ^{
            NSLog(@"task2--start");
            [self printCurrentThread];
            NSLog(@"task2--end");
        });
        NSLog(@"--end--");
    }];
```

* 在 **其他线程** 向 **并发队列** 执行 **异步派发**

```objc
    // 情景 12：
    // 在 **其他线程** 向 **并发队列** 执行 **异步派发**
    /*
     15:42:38    --start--
     15:42:38    cur = **NSThread: 0x6000036f28c0**{number = 7, name = (null)}
     15:42:38    --end--
     15:42:38    task1--start
     15:42:38    task2--start
     15:42:38    cur = **NSThread: 0x6000036aac40**{number = 6, name = (null)}
     15:42:38    cur = **NSThread: 0x6000036c2840**{number = 4, name = (null)}
     15:42:38    task2--end
     15:42:41    task1--end
     */
    [NSThread detachNewThreadWithBlock:^{
        NSLog(@"--start--");
        [self printCurrentThread];
        dispatch_queue_t concurrentQueue = dispatch_queue_create("concurrentQueue",
                                                           DISPATCH_QUEUE_CONCURRENT);
        dispatch_async(concurrentQueue, ^{
            NSLog(@"task1--start");
            [self printCurrentThread];
            sleep(3);
            NSLog(@"task1--end");
        });
        dispatch_async(concurrentQueue, ^{
            NSLog(@"task2--start");
            [self printCurrentThread];
            NSLog(@"task2--end");
        });
        NSLog(@"--end--");
    }];
```


### 实验结果

1. 在主线程向主队列同步派发，会造成死锁。

    同步派发，会阻塞当前线程，该情况即阻塞主线程，然后等待任务返回；此时任务在主队列中，主队列中的任务需要等待主线程来执行。
    
2. 在主线程向主队列异步派发，任务在主线程中按顺序执行。

3. 在主线程向串行队列同步派发，任务在主线程中按顺序执行。

4. 在主线程向串行队列异步派发，任务在同一个子线程里按顺序执行。

    **在主线程中派发任务，并不代表任务一定在主线程里执行。**
    
5. 在主线程向并发队列同步派发，任务在主线程里按顺序执行。

    第 3、5 条，之所以会在主线程里执行，是因为 apple 进行了性能优化，切换线程是耗性能的。
    
6. 在主线程向并发队列异步派发，任务在不同的子线程里同时执行。

7. 在子线程向主队列同步派发，任务在主线程里按顺序执行。

8. 在子线程向主队列异步派发，任务在主线程里按顺序执行。

9. 在子线程向串行队列同步派发，任务在同一个子线程里按顺序执行。

10. 在子线程向串行队列异步派发，任务在同一个子线程里按顺序执行，但是该子线程不是当前子线程。

11. 在子线程向并发队列同步派发，任务在同一个子线程里按顺序执行。

12. 在子线程向并发队列异步派发，任务在两个不同的子线程里同时执行，并且与当前子线程也不相同。

## 一些结论

* 主队列的任务都在主线程里执行
* 主线程里派发任务，不一定在主线程里执行（4 主线程里向串行队列异步派发，任务在子线程里执行）
* 同步派发的特征是塞当前线程，而不是让任务在当前线程立即执行（5 子线程向主队列同步派发，任务在主线程里执行）
* 串行队列异步派发，任务会在一个新子线程中执行（4 10 主线程/子线程向串行队列异步派发）
* 串行队列同步派发，任务会在当前线程中执行（3 9 主线程/子线程向串行队列同步派发）