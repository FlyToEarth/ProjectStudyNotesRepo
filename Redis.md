# 简介

用作数据库、缓存、消息队列等场景，最热门的NoSQL数据库之一。

传统数据库性能瓶颈越来越明显，主要是磁盘IO导致的。IO操作速度比磁盘慢很多，如果能够把数据存储在内存中的话，就可以大大提高性能。就有了Redis这种基于内存的数据存储系统。

Redis支持五种基本数据类型和五种高级数据类型

使用方式：CLI、API、GUI

优点

- 性能极高
- 数据类型丰富，单键值对最大支持512M的数据
- 简单易用，支持所有主流编程语言
- 支持数据持久化、主从复制、哨兵模式等高可用特性



# 安装

Redis和Docker一样必须运行于Linux系统环境中

在Linux系统直接`apt install`即可

在Windows中比较麻烦

1. 第一种方式是使用WSL，在WSL中安装Redis

2. 第二种方式使用Docker，从Docker中pull一个Redis镜像来运行

   ​	注：家庭版Windows想用Docker Desktop请百度hyper-V）

3. 第三种是Windows中`.exe`来安装，需要注意的是这种安装方式只能安装到`5.0`版本的Redis



# 启动Redis

非Docker安装的使用`redis-server`命令就可以启动Redis服务

Docker安装的Redis比较特殊，要使用Docker容器来运行

- 法一：直接用当前终端启动Redis服务

  - 命令格式：`docker run redis`

  - 缺点：打开的这个终端就不能干别的事了，只能放在这里运行Redis服务端

    ![image-20240221235413916](.\images\image-20240221235413916.png)

- 法二：后台运行

  - 以下内容来自博客：[在Docker中启动Redis并进入Redis](https://blog.csdn.net/sinat_41258771/article/details/104423106)

  - 命令格式：`docker run -d redis`，`-d`表示后台运行，此时终端会显示一个字符串，代表容器的ID，可以使用`docker ps`来查看正在运行的容器。（记得区分容器和镜像~）

    ![image-20240221235645754](.\images\image-20240221235645754.png)

  - 拓展命令格式：`docker run -d -p 6379:6379   --name="myredis"  redis`，`-p`表示进行端口映射，将容器中的6379映射到运行Docker机器中的6379端口，`--name`表示自定义容器名

  - 优点：这个终端还可以继续干别的事，不会被Redis服务占用

- 拓展：结束Docker容器的运行

  `docker stop 容器ID`，正在运行的容器的ID通过`docker ps`来查看

启动了Redis服务端后，还需要连接Redis服务端才能使用Redis

使用`docker exec -it Redis容器的ID或NAME redis-cli`来进入Redis的客户端

此时终端头应该变成这样：

![image-20240222000708811](.\images\image-20240222000708811.png)

这样就代表成功连接到了Redis的服务端，可以使用Redis了

当启动过Redis后Docker就会生成一个容器，下一次可以通过`docker start 容器id`来运行创建好的容器了

# RedisInsight的使用

![image-20240222125334494](.\images\image-20240222125334494.png)



# 数据

Redis中的数据是以键值对的形式存储的，需要指定一个键和一个值，键和值之间使用空格来分隔。

**注：Redis中的键值对是大小写敏感的，但Redis操作用的命令是不敏感的，如get GET都可以正常使用**

## 字符串String

使用SET设置一个键值对，如`SET name wuqing`。通过GET来获取键对应的值。如下：

![image-20240222001159683](.\images\image-20240222001159683.png)

**注：对同一个键重复set会覆盖其中的值**

**Redis中默认使用字符串来存储数据**，而且是二进制安全的。可以把很多类型的数据存放到Redis中，如数值、布尔、序列化的对象。如下：

![image-20240222001443842](.\images\image-20240222001443842.png)

可以看到数字被加上了引号，表名这是一个字符串

通过DEL删除键，如下：

![image-20240222001532192](.\images\image-20240222001532192.png)

通过EXISTS判断一个键是否存在，如下：

![image-20240222001803022](.\images\image-20240222001803022.png)

可以使用KEYS查看数据库中都有哪些键，需要添加一个匹配的参数，如`*`表示匹配所有的键，`*123`表示匹配以123结尾的键。

**可以使用FLUSHALL来删除掉所有的键**，如下：

![image-20240222002048793](.\images\image-20240222002048793.png)

Redis中的键和值都是二进制形式存储，所以默认不支持中文显示。

![image-20240222002219528](.\images\image-20240222002219528.png)

这里的存储是没有问题的，只是显示是二进制的形式，只需要退出连接，重新登录一下，不过要加上参数`--raw`表示以原始形式显示内容，如下：

![image-20240222002401577](.\images\image-20240222002401577.png)

使用TTL来查看键的过期时间，输出`-1`的话表示没有设置过期时间。

使用EXPIRE设置键的过期过期时间，如设置name的过期时间为10s

![image-20240222122358521](.\images\image-20240222122358521.png)

6表示6秒后过期，-2表示已经过期，再GET name也不会有任何输出

也可以使用SETEX直接设置带过期时间的键值对

![image-20240222122624640](.\images\image-20240222122624640.png)

通过SETNX当键不存在时来设置键，如果存在则不修改值

![image-20240222122858722](.\images\image-20240222122858722.png)

# 列表List

和Python的列表很像，可以存储各种类型到一个列表中，下标同样从0开始

如添加一个字母到letter列表中

这里的LPUSH表示在列表头添加元素，同样还有RPUSH表示在结尾添加元素

![image-20240222125712651](.\images\image-20240222125712651.png)

可以在命令行中使用LRANGE key start end来展示一个列表某区间的值![image-20240222125807727](.\images\image-20240222125807727.png)

这里使用了`-1`作为列表的结尾（所以说和Python的列表很像~）

LPUSH可以一次添加多个元素，相当于入栈，可以看到顺序是反着的

![image-20240222130400726](.\images\image-20240222130400726.png)

![image-20240222130708809](.\images\image-20240222130708809.png)

通过LPOP从表头删除元素，指定删除元素的个数即可

![image-20240222131007245](.\images\image-20240222131007245.png)

通过LLEN查看列表的长度

![image-20240222131104159](.\images\image-20240222131104159.png)

使用LTRIM删除列表中指定范围以外的元素，注意：下标从0开始

![image-20240222131458739](.\images\image-20240222131458739.png)

# 集合Set

Set中元素无序、不可重复

使用SADD添加元素到集合中：SADD course redis，返回1表示添加成功

使用SMEMBERS查看集合元素：SMEMBERS course

如果添加重复元素，如再来一次SADD course redis，返回的0表示false添加失败

使用SISMEMBER判断元素是否在集合中：SISMEMBER course redis，返回1表示在集合中，返回0表示不在集合中

使用SREM删除集合中的元素：SREM course redis，返回1表示删除成功

集合的运算：交并补对应SINTER SUNION SDIFF



# 有序集合SortedSet，ZSet

有序集合的每个元素都会关联一个浮点类型的分数，按照分数对元素**进行从小到大的排序**。同样的，集合中的元素是唯一的，但分数可以重复。相关命令都以Z开头。

ZADD添加元素：ZADD result 700 清华 699 北大 400 带专，返回3表示添加成功

ZRANGE查看集合元素：ZRANGE result 0 -1，这样只会输出元素不会输出分数，可以加上参数WITHSCORES

ZSCORE查看某元素的分数：ZSCORE result 清华

ZRANK查看某元素的排名：ZRANK result 清华，可以看到输出3表示从小到大排序是3

ZREVRANK查看从大到小的排名：ZREVRANK result 清华，可以看到输出0，这样就是第一名了

ZREM删除一个元素：ZREM result 清华



# 哈希Hash

键值对的集合，适合用来存储对象。相关命令都以H开头。

HSET添加KV对：HSET person name wuqing

HGET获取KV对：HGET person name

HGETALL获取所有键值对：HGETALL person

HDEL删除KV对：HDEL person age，返回1表示删除成功

HEXISTS判断K是否存在：HEXISTS person name，返回1表示存在，0不存在

HKEYS获取所有键：HKEYS person

HLENS获取所有键值对的数量：HLENS person



# 发布订阅功能

PUBLISH 将消息发送到指定频道，SUBSCRIBE 订阅频道来接收频道消息

类似聊天，可以发字符串

但是消息不能持久化，无法记录历史消息

# Stream

解决发布订阅功能的局限性，**注：Stream的消息需要是键值对**

XADD向Stream中添加消息：XADD wuqing * "course redis"，wuqing为频道名，*表示自动生成一个id，添加消息的内容是键course属性redis，如果手动指定ID的话要注意格式为`a-b`的形式，a表示一个时间戳，b表示一个序列号如`1-0 2-0 3-0`，**手动指定要保证ID的递增**

XLEN查看Stream中消息的数量：XLENS wuqing

XRANGE查看Stream中消息的详细内容：XRANGE wuqing - +，使用`- +`表示所有消息

XDEL删除消息：XDEL wuqing 消息ID

也可以用XTRIM来删除消息

XREAD读取（消费）消息：XREAD count 2 block 1000 streams wuqing 0，`count 2`表示读取两条消息，如果没有则`block 1000`阻塞1秒，最后指定Stream流的消息队列，0表示从第一条开始读取。**可以重复读取**

可以创建消费者组，在组中添加消费者：XGROUP CREATE wuqing group1 0，消息频道为wuqing，组名为group1，ID为0

XINFO GROUPS查看消费者组的信息：XINFO GROUPS wuqing

XGROUP CREATECONSUMER添加消费者：XGROUP CREATECONSUMER wuqing group1 consumer1

XREADGROUP读取消息：XREADGROUP GROUP group1 consumer1 COUNT 2 STREAMS wuqing >，`>`表示读取最新的消息

# 地理空间

3.2+的新特性，提供了存储地理位置信息的数据结构，支持对地理位置进行计算操作。相关命令以GEO开头。

GEOADD添加一个地理位置信息：GEOADD city 116 39 beijing，`116 39`是经纬度，城市名字是beijing，返回1则添加成功。可以一次添加多个信息。

GEOPOS获取地理位置的经纬度：GEOPOS city beijing，返回经度+纬度

GEODIST计算地理位置之间的距离：GEODIST city beijing shanghai，返回的数字单位是米，想换算成千米的话需要在命令后加上KM，如GEODIST city beijing shanghai KM

GEOSEARCH搜索指定范围内成员并返回，可以以成员位置或指定经纬度为中心按照圆或矩形来搜索：GEOSEARCH city FROMMEMBER shanghai BYRADIUS 300 KM

# HyperLogLog

做基数统计的算法。基数：如果集合中每个元素都是唯一且不重复的，那么集合的基数就是元素的个数。似乎就是不重复元素的个数

原理是使用随机算法，牺牲一定精确度换取更小内存消耗。

PFADD添加元素：PFADD course redis

PFCOUNT查看基数：PFCOUNT course，返回基数值

添加另一个来进行合并：PFADD course2 python redis

PFMERGE合并多个HyperLogLog：PFMERGE result course course2，把合并结果放到result中

此时的基数为3

# 位图Bitmap

字符串的拓展，使用string模拟bit数组，下标就是偏移量，值只有0和1。支持位运算。所有命令以BIT开头。

SETBIT设置某偏移量的值：SETBIT dianzan 0 1，把偏移量0的位置设置为1，再SETBIT dianzan 1 0，把1的位置设置为0，这样就设置了一个长度为2的位图。

GETBIT获取某偏移量的值：GETBIT dianzan 0

一位一位设置很麻烦，bitmap本质上是一个字符串，直接使用字符串命令设置它的值：SET dianzan "\xF0"，直接用一个十六进制数来设置

BITCOUNT统计某个key中多少个bit是1：BITCOUNT dianzan

BITPOS获取出现的第一个0或1的位置，可以设置查询区域：BITPOS dianzan

# 位域Bitfield

将很多小的整数存储到一个较大的位图中，可以更高效实用内存。

假设我们创建了一个新角色，等级为1，有100个金币

BITFIELD设置信息：BITFIELD player:1 set u8 #0 1，player:1是ID，u8表示8位无符号整数，#0为第一个位置，1表示等级为1

获取等级：BITFIELD player:1 get u8 #0

设置金币：BITFIELD player:1 set u32 # 1 100

数值加法：BITFIELD player:1 incr by u32 #1 100，这样就把金币加了100

# 事务

一次请求中执行多个命令。通过MULTI和EXEC实现。

注：和关系型数据库不同，Redis不能保证同时成功和同时失败，而是允许有成功和失败。

Redis可以保证三点：

1. 在exec之前所有的命令都会被放入到一个队列中缓存起来，不会立即执行
2. 收到exec后开始执行，有一个命令失败，其他仍然会执行
3. 执行过程中，其他客户端提交的请求不会被插入到事务的执行命令序列中

MULTI开启事务，返回一个OK并在命令提示符中添加了（TX）

之后的所有命令都会返回QUEUED表示命令被放入到队列中。

执行EXEC后会执行所有的命令，并返回每条命令的执行是否成功，成功返回OK或者结果，失败返回一个error信息。

# 持久化

如果没有持久化的话，Redis中根本存不住数据（因为Redis是基于内存的）

- RDB方式Redis Database

  在指定时间间隔内，将内存中的数据写入磁盘，可以通过`redis.conf`文件来配置，使用`save`参数来设置（注：Docker中修改配置文件略麻烦一些）也可以在命令行中手动save。因为是保存了某一时刻的数据，所以RDB方式更适合做备份。

  另一个问题：生产环境中为Redis开辟的内存区域一般都比较大，内存中数据同步到硬盘需要较长时间，这段时间内Redis都是阻塞的不能接受任何请求，显示是不行的。Redis提供了bgsave命令用来开启一个子进程将内存中的数据写入到硬盘中。有一些性能损耗，因为创建子进程需要时间，此时还是会被阻塞，不能做到秒级快照。

- AOF方式Append only File

  执行写命令时，不仅会将命令写到内存中，还会将命令写入到一个追加的文件中。这个文件就是AOF文件，会以日志形式记录每一个写操作，Redis重启后会通过重新执行AOF文件中的命令来重建数据库的内容。

  同样通过`redis.conf`来配置，修改`appendonly`参数值为`yes`

# 主从复制

将一台Redis服务器的数据复制到其他Redis服务器，也叫主节点和从节点。一个主节点可以有多个从节点，每个从节点只能有一个主节点。数据复制是单向的，一般主节点负责写，从节点负责读。主节点会将自己的数据变化通过异步的方式发送给从节点，从节点接收后更新自己的数据，达到主从一致的目的。

![image-20240222210557069](.\images\image-20240222210557069.png)

通过配置文件配置，略

# 哨兵模式

哨兵用来监控Redis各个节点是否运行正常，主要用来执行几个功能

- 监控节点是否正常
- 通知，通过发布订阅模式通知其他节点
- 自动故障转移，主节点故障不能工作时，它会将一个从节点升级为新的主节点，将其他从节点指向新的主节点

因为Docker搞配置文件比较麻烦，所以就都不搞了。