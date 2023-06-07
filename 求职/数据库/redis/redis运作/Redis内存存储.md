redisDb代表redis数据库结构，其中的dict结构主要存数据。而dict就是[[Set#编码方式|HASHTABLE]]。

过期键是存在另一个字典上，就是expires。

dict和expires当中的key都是存的string对象指针，不会重复占用。