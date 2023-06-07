## 命令

### 写

1. SET
新建/修改
2. SETNX
不存在则新建/修改

### 删

1. DEL
返回值为删除了几行

### 读

1. GET
查询某个key，存在返回value，不存在返回nil
2. MGET
查询多个key

## 底层实现

有三种编码方式：
1. INT编码：整数，且范围在LONG_MAX内
2. EMBSTR编码：小于等于阈值字节
3. RAW编码：大于阈值字节

EMBSTR和RAW都是redisObject和SDS组成，SDS是redis对C字符串的封装。EMBSTR下redisObject和SDS内存连续，而RAW分开。

EMBSTR两个结构可以一次性分配空间，但重新分配空间就整体都需要再分配，所以规定只读，**任何写操作后EMBSTR都会变成RAW**，因为认为修改过的字符串是易变的。

### SDS

SDS分为sdshdr8,sdshdr16,sdshdr32,sdshdr64，字段属性一样，应对不同大小字符串。

在结构中有len表示使用多少，alloc表示分配多少内存。两个字段的差值就是预留空间大小。

1. 使用len快速返回长度
2. 增加空余空间，可以追加数据
3. 不再特殊判断'\0'，二进制安全

当len小于1M，预留len
大于1M，预留1M
所以预留空间为min(len,1M)

## 其他

1. 追加命令append
2. String最大512MB
3. EMBSTR的阈值，3.2之前是39，3.2之后是44。
	1. 首先，Redis使用jemalloc作为内存分配器，以64字节为内存单位做分配。
	2. redisObject大小一直是16Bytes
	3. sdshdr占用的内存大小为1Byte(len)+1Byte(alloc)+1Byte(flag)+内联数组大小，内联数组中'\0'占用1Byte，所以能用的大小为44。
	4. Redis3.2之前sdshdr的结构是4Byte的capacity和4Byte的len，非数据字段占用8Byte。后来把SDS分为多个类型，EMBSTR使用sdshdr8节约了6个字节，但多引入flag占据1字节，所以可以用的多了5个Byte。