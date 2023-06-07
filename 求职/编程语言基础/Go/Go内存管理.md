## 内存管理

1. Page
操作系统管理内存以页为单位，但go当中的page与操作系统的为倍数关系。
2. Span
一组连续的page，go内存管理的基本单位。
3. mcache
保存了各种大小的span，并按span class分类，小对象直接从mcache分配内存，起到缓存作用，可以无锁访问。每个P拥有一个mcache，最多需要GOMAXPROCS个mcache就可以实现各线程对mcache的无锁访问。
4. mcentral
所有内存共享的缓存，需要加锁访问，根据span class对span分类，串联成链表，当mcache某个级别的span的内存被分配光，就会向mcentral申请一个。mcache每个级别的span有两个链表。
5. mheap
堆内存的抽象，把从OS申请的内存组织成span并保存，mcentral不够用就向mheap申请，mheap的span不够就向OS申请，同样加锁。mheap把span组织成2棵树，然后把span分配到heapArena管理，包含地址映射和span是否包含指针等位图，位了更高效利用内存，分配回收再利用。
6. 内存分配
go一共有67种span规格。
为对象寻找span流程：
	1. 计算对象所需内存大小size
	2. 根据size到size class映射，计算出所需的size class
	3. 根据size class和对象是否包含指针计算span class
	4. 获取该span class指向的span
大对象（大于32KB）直接在mheap上分配，优先从`free`找可用的span，没有就找`scav`，还没有就向OS申请，再重新搜索两棵树。如果找到的span比需求大，就分成两部分，一部分正好等于需求，另一部分放回`free`。

## GC触发

- 在堆上分配大对象时检测此时是否满足GC条件，满足就GC（自动GC）
- 调用runtime.GC（主动GC）

系统GC触发条件：当前堆上活跃对象大于初始化时设置的GC触发阈值。或者两分钟没有GC。

GC的时候处理的都是span。