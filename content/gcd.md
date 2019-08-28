### 进程和线程

- 进程：正在运行的可执行文件，拥有虚拟的内存空间和系统资源，比如端口号。

- 线程：一个独立的代码执行路径。

### 同步和异步

- 同步会等待任务执行完再继续执行，不具备开启新线程的能力。

- 异步不会等待任务完成后才返回，会立即返回，具备开启新线程的能力。

### dispatch_after

- 指定时间后将任务追加到队列当中，而不是指定时间后立即执行任务。

```
dispatch_queue_t serialQueue = dispatch_queue_create("chunf.com.afterTest", DISPATCH_QUEUE_SERIAL);
dispatch_async(serialQueue, ^{
     sleep(5); //任务1
});
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), serialQueue, ^{
     NSLog(@"任务2"); //2秒后并不会打印
});

```

### dispatch_barrier

- 栅栏函数，顾名思义就像一个栅栏，可以控制并发队列任务的执行顺序

```
//dispatch_barrier_async 函数需要使用自己创建的队列，如果使用全局队列，则使用效果与dispatch_async并无差异

dispatch_queue_t concurrentQueue = dispatch_queue_create("chunf.com.afterTest", DISPATCH_QUEUE_CONCURRENT);
    
dispatch_async(concurrentQueue, ^{
    sleep(2);
    NSLog(@"任务1");
});
    
dispatch_barrier_async(concurrentQueue, ^{
    NSLog(@"任务2");
});
    
dispatch_async(concurrentQueue, ^{
    NSLog(@"任务3");
});
    
//打印顺序 任务1 任务2 任务3

```

### dispatch_once

- 一次性代码，线程安全,oc常用来创建单例

```
 static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        //code to be executed once
    });

```

### dispatch_group

- 队列组，使用场景一般是判断多个任务都执行完，才做某事

```
dispatch_queue_t concurrentQueue = dispatch_queue_create("chunf.com.afterTest", DISPATCH_QUEUE_CONCURRENT);
dispatch_group_t group = dispatch_group_create();
    
dispatch_group_async(group, concurrentQueue, ^{
    NSLog(@"任务1");
});
    
dispatch_group_async(group, concurrentQueue, ^{
    NSLog(@"任务2");
});
    
dispatch_group_notify(group, concurrentQueue, ^{
    NSLog(@"任务3");
});
    
//--------------------前后两种写法并无不同-------------------------
// 任务3 会等到任务1和任务2都完成才执行。
    
dispatch_group_enter(group);
dispatch_async(concurrentQueue, ^{
    NSLog(@"任务1");
    dispatch_group_leave(group);
});
    
dispatch_group_enter(group);
dispatch_async(concurrentQueue, ^{
    NSLog(@"任务2");
    dispatch_group_leave(group);
});
    
dispatch_group_notify(group, concurrentQueue, ^{
    NSLog(@"任务3");
});

```

### dispatch_symaphore

- 信号量，常见的有`creat`,`signal`,`wait`三个函数，signal使信号量加1，wait使信号量减1。常用来做线程同步，也可用来控制代码执行顺序，比如我们公司的项目用来控制多个弹框的的展示顺序（要注意避免阻塞主线程）。

```

dispatch_semaphore_t semphore = dispatch_semaphore_create(1);

// 函数内部
dispatch_semaphore_wait(semphore, DISPATCH_TIME_FOREVER); //信号量-1
[arr addObject:person]; //线程安全
dispatch_semaphore_signal(semphore); //信号量+1

```

```

// alert1 > alert2 > alert3
dispatch_semaphore_t semphore = dispatch_semaphore_create(0);

[self showAlert1WithDismissHandler ^{
   dispatch_semaphore_signal(semphore); //信号量+1
}];

dispatch_semaphore_wait(semphore, DISPATCH_TIME_FOREVER); //信号量-1
[self showAlert2WithDismissHandler ^{
   dispatch_semaphore_signal(semphore); //信号量+1
}];

dispatch_semaphore_wait(semphore, DISPATCH_TIME_FOREVER); //信号量-1
[self showAlert3WithDismissHandler ^{
   dispatch_semaphore_signal(semphore); //信号量+1
}];

```

### 锁死分析

- 锁死是相对于队列来说的，而不是线程，明白了这点对于理解一些死锁现象很有帮助。

```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        dispatch_sync(dispatch_get_main_queue(), ^(void){
            NSLog(@"这里死锁了"); // 任务2
        });
    }
    return 0;
}
```

- 简单分析： dispatch_sync函数可以当做一个任务1，优先添加到了主队列当中，任务2随后添加到了主队列，因为串行队列，任务2需要等待任务1完成后才执行，但同步函数必须任务执行完才返回，所以造成了任务1等待2，任务2等待1的情况，互相等待造成了死锁。


