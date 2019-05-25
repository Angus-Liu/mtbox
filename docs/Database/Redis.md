# Redis

## # 基础

> [《Redis 设计与实现》- 第 1 章 引言](https://book.douban.com/subject/25900156/)
>
> [Redis 设计与实现](http://redisbook.com/)

Redis 数据库里面的每个键值对（key-value pair）都是由对象（object）组成的：

- 其中，数据库的键总是一个字符串对象（string object）；
- 而数据库的值则可以是字符串对象、列表对象（list object）、哈希对象（hash object）、集合对象（set object）、有序集合对象（sorted set object）这五种对象中的其中一种。

## # 渐进式 Rehash

> [《Redis 设计与实现》- 第 4 章 字典](https://book.douban.com/subject/25900156/)
>
> [渐进式 rehash](http://redisbook.com/preview/dict/incremental_rehashing.html)

随着操作的不断执行，哈希表保存的键值对会逐渐地增多或者减少，为了让哈希表的负载因子（load factor）维持在一个合理的范围之内，当哈希表保存的键值对数量太多或者太少时，程序需要对哈希表的大小进行相应的扩展或者收缩。

扩展或收缩哈希表需要将 ht[0] 里面的所有键值对 rehash 到 ht[1] 里面， 但是， 这个 rehash 动作并不是一次性、集中式地完成的， 而是分多次、渐进式地完成的。

渐进式 rehash 的好处在于它采取分而治之的方式， 将 rehash 键值对所需的计算工作均滩到对字典的每个添加、删除、查找和更新操作上， 从而避免了集中式 rehash 而带来的庞大计算量。

### 渐进式 rehash 执行期间的哈希表操作

因为在进行渐进式 rehash 的过程中， 字典会同时使用 ht[0] 和 ht[1] 两个哈希表， 所以在渐进式 rehash 进行期间， 字典的删除（delete）、查找（find）、更新（update）等操作会在两个哈希表上进行： 比如说， 要在字典里面查找一个键的话， 程序会先在 ht[0] 里面进行查找， 如果没找到的话， 就会继续到 ht[1] 里面进行查找， 诸如此类。

另外， 在渐进式 rehash 执行期间， 新添加到字典的键值对一律会被保存到 ht[1] 里面， 而 ht[0] 则不再进行任何添加操作： 这一措施保证了 ht[0] 包含的键值对数量会只减不增， 并随着 rehash 操作的执行而最终变成空表。

## # 持久化

> [《Redis 设计与实现》- 第 10 章 RDB 持久化](https://book.douban.com/subject/25900156/)
>
> [《Redis 设计与实现》- 第 11 章 AOF 持久化](https://book.douban.com/subject/25900156/)
>
> [redis 的 rdb 和 aof 持久化的区别](https://blog.csdn.net/jackpk/article/details/30073097)

### RDB 持久化

RDB 持久化功能所生成的 RDB（Redis DataBase） 文件是一个经过压缩的二进制文件，通过该文件可以还原生成 RDB 文件时的数据库状态。

因为 RDB 文件是保存在硬盘里面的，所以即使 Redis 服务器进程退出，甚至运行 Redis 服务器的计算机停机，但只要 RDB 文件仍然存在，Redis 服务器就可以用它来还原数据库状态。

### AOF 持久化

除了 RDB 持久化功能之外，Redis 还提供了 AOF（Append Only File）持久化功能。与 RDB 持久化通过保存数据库中的键值对来记录数据库状态不同，AOF 持久化是通过保存 Redis 服务器所执行的写命令来记录数据库状态的。