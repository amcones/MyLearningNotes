一组连接起来的string的集合，最大元素个数是2^32-1。可以存储一批任务数据，一批消息等。

## 命令

### 写

1. LPUSH/RPUSH
从左/右插入一个或多个元素，返回list中元素的总数。
2. LPOP/RPOP
从左右移出并获取一个元素
3. DEL
删除对象，返回值为删除了几行

### 读

1. LLEN
查看list的长度
2. LRANGE
查看start到stop的元素，可以用负数

## 底层实现

### 编码方式

3.2之前，List有两种编码方式，一种ZIPLIST，另一种LINKEDLIST。

当满足如下条件，用ZIPLIST：
1. 列表对象保存的所有字符串长度都小于**64字节**
2. 列表对象元素少于512个（LIST的限制）

ZIPLIST底层用压缩列表（数组）实现，内存排列紧凑。因此插入数据时后面的数据都要挪，如果空间不够还要开新空间然后整体移动，会有很多拷贝。

不满足ZIPLIST则使用LINKEDLIST，底层是链表连接，牺牲内存提高更新效率。

3.2版本引入了QUICKLIST，是ZIPLIST和LINKEDLIST的结合体，在LINKEDLIST每个节点当中存一个ZIPLIST。当数据较少，QUICKLIST的节点只有一个，相当于ZIPLIST，数据很多的时候同时利用ZIPLIST和LINKEDLIST的优势。

但ZIPLIST本身存在一个连锁更新的问题，所以Redis7.0之后，使用LISTPACK取代了ZIPLIST。

### 压缩列表数据结构

ZIPLIST：

![[截屏2023-05-11 17.55.54.png]]

- zlbytes：表示该ZIPLIST一共占了多少字节数，包含zlbytes本身占据的字节
- zltail：表示相对最后一个节点偏移多少字节，快速定位到尾部。
- zllen：表示有多少个数据节点
- entry1~3：数据节点
- zlend：特殊的节点，表示ZIPLIST结束

ZIPLIST节点的结构：
- prevlen：表示上一个节点的数据长度，定位上一个节点，所以压缩列表才可以从后往前操作。**如果前一个节点长度小于254字节（zlend占了第255个），那么prevlen占1Byte，否则占5Byte。**（这是ZIPLIST的性质，LIST是使用方）
- encoding：编码类型
- entry-data：实际的数据

encoding由数据类型和数据长度组成，使用这个可以实现从前往后操作。

查询时，通常可以用zllen直接返回长度，但如果长度超出65535就需要遍历。查询指定节点同样需要遍历。

更新时平均$O(n)$，而且可能带来连锁更新（prevlen导致）。不是指插入数据后后面的节点都要后移，而是插入数据后，后面节点的prevlen需要变化，如果变成了5个Byte，那么这个entry的长度也膨胀，后面的数据也会变化，导致新数据插入导致后移完成后，还需要逐步迭代更新，$O(n^2)$。

LISTPACK为了解决ZIPLIST的连锁更新，需要不记录prevlen并且还能找到上一个节点的起始位置。

LISTPACK节点定义：
- encoding-type：编码类型
- element-data：数据内容
- element-tot-len：存储整个节点除它自身之外的长度

element-tot-len占用的每个字节第一个bit标识是否结束，0是结束，1是继续，剩下7个bit表示数据大小。当需要从后往前找时，可以从后往前查找每一个字节，找到上一个entry的element-tot-len的结束标志就可以算出上一个节点的开头。