1. 进程，线程和协程
进程是CPU分配资源的基本单位，线程是CPU调度资源的基本单位，协程是用户写程序在一个线程上抽象出来的多个执行单位。
我们打开一个软件，比如Word，它就是一个进程，CPU给它分配了一套资源。而用户真正使用它时，用的编辑，保存等功能就是各个线程，它们实际执行任务。协程是程序员写程序去调度程序运行，更加轻量。**协程有自己的栈空间，但共享堆空间。（比如goroutine实际上每个协程都是一个函数，函数当然占用栈空间，同时因为在同一个线程上运行，所以共享堆空间）**。
2. 并行和并发，并行是同时执行，而并发是交替执行。
3. channel
- 可以判定读取或range读取channel当中的值。
- 可以通过定义类型别名来定义单向channel
- 无缓冲channel是同步模式，有缓冲channel是异步模式
- 可以把channel当成锁
- channel产生panic的情况：
	- 关闭一个未初始化的channel
	- 往一个关闭的channel写
	- 重复关闭channel
4. sync，共享内存并发安全
- `sync.WaitGroup`是一个对象，维护一个计数器，通过三个方法来配合使用
	- Add(delta int) 计数器加delta
	- Done() 计数器减1
	- Wait() 阻塞代码直到计数器为0
- `sync.Once`只执行一次，经常用于延迟初始化，初始化后一直驻留在内存。
- `sync.Mutex`互斥锁，注意要在同一个对象上操作，对已经lock或者unlock的锁进行lock或者unlock会报RE
- `sync.RWMutex`读写锁，同时只能有一个读，可以有多个写，读写互斥
- `sync.Map`在Go1.9引入，并发安全版map，不需要初始化就可以使用。
- `sync.Atomic`原子操作
5. select，一个goroutine监听多个channel的读写。**当多个case准备好了，随机执行一个**。
6. context，Go1.7引入的一个标准库接口，用于父子goroutine间值传递和发送cancel信号，通过创建根context后使用4种with方法来扩展使用
7. 定时器`Timer`和`Ticker`。`Timer`当中有一个计时器和一个channel，到期后channel写入一个系统时间。`Ticker`对象字段和`Timer`一致，但会固定时间间隔触发。
8. 协程池，有三种角色
	1. Worker，执行任务的goroutine
	2. Task，具体任务
	3. Pool，池子
根据角色的定义构建字段和方法。