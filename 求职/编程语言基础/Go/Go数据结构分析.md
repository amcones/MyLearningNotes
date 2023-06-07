## string

string底层的数据结构是一个指向byte切片的指针和一个len。

string不能被改变，只能重新赋值。

byte[]和string可以互转，**会发生内存拷贝**。

字符串拼接最快的方式是`append`，`bytes.Buffer`和`strings.Builder`。

## slice

切片底层的数据结构是一个指针，一个len和一个cap。指针即底层数组。

从原切片截取或复制新切片出来，底层数据结构依然指向原切片的底层。

如果新切片触发了扩容，就会重新分配空间，此时修改原切片对新切片无影响。

切片的扩容机制：
- 1.17之前
1. 如果期望容量大于原容量的2倍，使用期望容量
2. 如果当前切片长度小于1024，就翻倍
3. 如果当前切片长度大于1024，每次增加25%容量直到新容量大于等于期望容量
- 1.18之后
1. 如果期望容量大于原容量的2倍，使用期望容量
2. 如果当前切片长度小于**256**，就翻倍
3. 如果当前切片长度大于256，**按照公式扩容**直到大于期望容量
`newcap = oldcap+(oldcap+3*256)/4`

## map

底层采用哈希表，用变种拉链法解决hash冲突。

拉链法：使用数组和链表保存数据，如果出现hash冲突就把新的元素插入到对应key的链表里。如果冲突严重导致链表过长，可以采用红黑树代替链表。

开放地址法：如果出现hash冲突，就线性向下寻找一个可用空间插入。

map底层数据结构：
- **count：元素数量**
- flags：标记map的状态
- B：桶数以2为底的对数
- noverflow：溢出桶数量近似值
- hash0:哈希种子
- **buckets：指向buckets数组的指针，buckets数组的元素为bmap，如果数组元素个数为0，为nil**
- oldbuckets：扩容时指向老的buckets数组的指针，非扩容时为nil
- nevacuate：表示扩容进度计数器，小于该值的桶已经完成迁移
- **extra：指向mapextra结构的指针，mapextra存储map中的溢出桶**

mapextra结构：
- overflow：溢出桶链表地址
- oldoverflow：老的溢出桶链表地址
- nextOverflow：下一个空闲溢出桶地址

go当中map是一个指向hmap的指针，占用8个字节。hmap包含多个结构为bmap的bucket数组，bucket底层采用优化拉链法连接bmap，链表每个节点保存8个键值对（节省内存对齐空间），多余的保存进溢出桶。在初始化map的时候会预分配一些溢出桶，用完了才会新建。

go对hash值分高8位和低8位使用（正好1字节）

map的访问两种方式，一种直接获取值，一种获取值和布尔，两种调用方法不同。

map数据过多会导致hash性能变差，溢出桶过多导致查找性能变差。此时触发自动扩容。

- 负载因子大于6.5:双倍扩容
- 溢出桶数量过多（溢出桶接近桶的数量，更准确的说大于正常桶或大于2^15）：等量扩容

负载因子大于6.5说明数据桶快用完了，查找下一个key很可能要去遍历溢出桶。

负载因子不大的情况下，频繁插入删除数据可能会造成很多溢出桶，所以要扩容让数据排列更紧密。

扩容时不会一次性全部迁移拷贝，遵循写时拷贝，每次只对用到的数据做迁移。

map遍历顺序随机，一是因为go扩容时key的位置会改变，二是因为hash表每次插入数据位置会变化，所以遍历顺序就是不同的。

## sync.Map

sync.Map读写分离，用两个map实现。读数据优先从read map读，读不到去dirty map读，写只在dirty写，read不加锁，dirty加锁。

- `Store()`：更新/插入键值对
- `Load()`：返回key对应的value
- `Delete()`：删除键值对
- `Range()`：遍历map

read可以看作是dirty的快照，当misses达到dirty的长度，就会提升dirty为read，覆盖read。

read当中entry中的p指针可能有三种取值：nil，expunged和正常值，前两种都表示删除。nil是表示删除的key只存在于read，做一个假删除的标志，之后创建新的dirty会把read中nil的都标记为expunged。因此对于expunged的key，如果要更新就需要操作read和加锁操作dirty。

## channel

channel用make初始化的时候会在堆上分配一个runtime.hchan类型的数据结构。

主要包含了一个循环队列，两个保存阻塞在读和写方向上goroutine的队列，一个锁和一些标记。

## context

context是一个接口，对这个接口有四种实现。还有canceler接口，用于取消方法的实现。如果一个实例既实现了context接口又实现了canceler接口，那么它就是可以取消的。

context主要用于goroutine之间传递上下文信息，比如传递请求的trace_id，追踪全局唯一请求 和 做取消控制，防止goroutine泄漏

context包中包括了四种context的实现

emptyCtx：不具备任何功能，基本直接返回空值
cancelCtx：同时实现两个接口，实现退出通知
timerCtx：封装了cancelCtx和计时器
valueCtx：封装了Context接口和k/v存储变量，实现数据传递

## defer

defer内部有一个指针指向链表，每个defer函数插入头部，执行从头部依次执行。

不要在循环用defer。

defer可以内联，栈，堆方式分配。

## interface

空接口当中包含有一个类型的抽象和一个指向变量的指针。

非空接口当中有一个指向变量的指针和一个itab结构的指针，保存有接口方法列表和data对应动态类型信息。

itabTable使用哈希表（数组实现）缓存用到的itab结构体，key为接口类型与实际类型分别哈希后的异或值。用开放地址法解决hash冲突。