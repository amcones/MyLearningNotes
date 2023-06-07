进程/线程调度开销大（CPU不区分进程线程，毕竟实际干活的都是线程），所以go在用户态建立协程，与线程之间建立M:N的关系，中间用调度器调度。

## GMP

G：Goroutine协程
M：Machine(thread)线程
P：processor处理器

![[截屏2023-05-07 20.03.44.png]]

G优先存放到P的本地队列（不超过256个），存不下了放在全局队列。

P的数量可以通过GOMAXPROCS配置，可以是环境变量也可以是runtime.GOMAXPROCS()。（默认为CPU个数，runtime.NumCPU()个）

M是当前操作系统分配到当前Go程序的内核线程数，可以通过runtime/debug中SetMaxThreads()来设置。但很少用，因为M是动态的。

## 设计策略

### 复用线程

避免频繁创建、销毁线程，而是对线程的复用。

- work stealing机制
空闲的P从忙的P的队列里偷一个G过来执行
- hand off机制
当一个G阻塞，创建/唤醒一个thread，把P转到新的thread上运行

### 利用并行

用GOMAXPROCS限制使用的CPU

### 抢占

一个goroutinue最多占用CPU10ms，抢占式并发，优先级平等

### 全局G队列

从全局队列当中偷取G

## 调度器的生命周期

### M0

- 启动程序后编号为0的主线程
- 在全局变量runtime.m0中，不需要在heap上分配
- 负责执行初始化和启动第一个G
- 启动第一个G以后，就和其他M一样

### G0

- 每次启动一个M，都会第一个创建的goroutine
- 仅用于负责调度的G
- 不指向任何可执行的函数
- 每个M都会有一个自己的G0
- 在调度或系统调用时M会切换到G0来调度
- M0的G0会放在全局空间

