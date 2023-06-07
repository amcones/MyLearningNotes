一个field、value都为string的hash表。每个hash可以存储$2^{32}-1$个键值对。适用于$O(1)$时间查找某个field对应数据的场景，比如任务信息配置，就可以任务类型为field，配置参数为value。

## 命令

### 写

1. HSET
为集合对应field设置value数据
2. HSETNX
如果field不存在则设置
3. HDEL
删除指定field，可以一次删多个
4. DEL
删除HSET对象
5. HMSET
设置多个键值对（已被HSET取代）

### 读

1. HGETALL
查找全部数据
2. HGET
查找某个key
3. HLEN
查找HSET中元素总数
4. HSCAN
从指定位置查找一定数量的HSET数据（COUNT属性只有在由ziplist转为dict才生效）

## 底层原理

### 编码格式

两种，一种是ZIPLIST（LISTPACK），一种是HASHTABLE，同时满足两个条件用ZIPLIST（LISTPACK）：
1. HSET对象保存所有值和键的长度都小于64字节
2. HSET对象元素个数少于512个
否则用[[Set#底层实现|HASHTABLE]]。

使用ZIPLIST时就是将field-value当作entry放入ZIPLIST。

使用HASHTABLE时，和SET的区别是SET中value始终为NULL，在HSET中有对应值。