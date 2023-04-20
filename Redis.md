# Redis

Nosql

> 非关系型数据库









### 常用命令：

> 设置键过期时间  expire

~~~bash
EXPIRE key seconds	# 秒为单位
127.0.0.1:6379> set key1 wangweijun
OK
127.0.0.1:6379> keys *
1) "key1"
127.0.0.1:6379> EXPIRE key1 10	# 设置10s后过期
(integer) 1
127.0.0.1:6379> keys *
(empty list or set)		# key1在10s后被删除
~~~



> 清空所有数据库

~~~bash
flushdb
~~~









​		

## 五大数据类型







### Redis-key

















### String（字符串）















### List（列表）

在redis里面，我们可以把**list**当成，栈、队列、阻塞队列

~~~bash
#############################################################################
127.0.0.1:6379> LPUSH list one	# 将一个值或者多个值，插入到列表头部（左）
(integer) 1
127.0.0.1:6379> lpush list two
(integer) 2
127.0.0.1:6379> LPUSH list three
(integer) 3
127.0.0.1:6379> LRANGE list
(error) ERR wrong number of arguments for 'lrange' command
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
3) "one"
127.0.0.1:6379> LRANGE list 0 1
1) "three"
2) "two"
127.0.0.1:6379> RPUSH list right	# 将一个值或者多个值，插入到列表尾部（右）
(integer) 4
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
3) "one"
4) "right"
#############################################################################
127.0.0.1:6379> Lpop list	# 移除list的第一个元素
"three"
127.0.0.1:6379> RPOP list	# 移除list最后一个元素
"right"
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
#############################################################################
Lindex
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
127.0.0.1:6379> lindex list 0	# 通过下标获取 list 中的某一个值
"two"
127.0.0.1:6379> lindex list 1
"one"

#############################################################################
Llen
127.0.0.1:6379> Lpush list one
(integer) 1
127.0.0.1:6379> Lpush list two
(integer) 2
127.0.0.1:6379> LPUSH list three
(integer) 3
127.0.0.1:6379> Llen list	# 返回列表的长度
(integer) 3

#############################################################################
移除某一个值 Lrem
127.0.0.1:6379> lpush list three
(integer) 4
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "three"
3) "two"
4) "one"
127.0.0.1:6379> lrem list 1 one
(integer) 1
127.0.0.1:6379>  LRANGE list 0 -1
1) "three"
2) "three"
3) "two"
127.0.0.1:6379>  LRANGE list 0 -1
1) "three"
2) "three"
3) "two"

127.0.0.1:6379> lrem list 2 three	# 移除了2个three
(integer) 2
127.0.0.1:6379> LRANGE list 0 -1
1) "two"

#############################################################################
trim 修剪
127.0.0.1:6379> RPUSH mylist "Hello"
(integer) 1
127.0.0.1:6379> RPUSH mylist "Hello01"
(integer) 2
127.0.0.1:6379> RPUSH mylist "Hello02"
(integer) 3
127.0.0.1:6379> RPUSH mylist "Hello03"
(integer) 4
127.0.0.1:6379> ltrim mylist 1 2	# 通过下标截取指定的长度，这个list已经改变了，截断了只剩下截取的元素1
OK
127.0.0.1:6379> Lrange mylist 0 -1
1) "Hello01"
2) "Hello02"
#############################################################################
rpop lpush
127.0.0.1:6379> LRANGE mylist 0 -1
1) "Hello"
2) "Hello01"
3) "Hello02"
4) "Hello03"
127.0.0.1:6379> rpoplpush mylist myotherlist	# 移除列表的最后一个元素，将它移动到新的列表中！
"Hello03"
127.0.0.1:6379> LRANGE mylist 0 -1	# 查看原来的列表
1) "Hello"
2) "Hello01"
3) "Hello02"
127.0.0.1:6379> LRANGE myotherlist 0 -1		# 查看目标列表中，确实存在该值！
1) "Hello03"

#############################################################################
lset	将列表中指定下标的值替换为另外一个值，更新操作
127.0.0.1:6379> exists list		# 判断这个列表是否存在
(integer) 0
127.0.0.1:6379> lset list 0 item	# 如果不存在列表我们去更新就会报错
(error) ERR no such key
127.0.0.1:6379> lpush list value1
(integer) 1
127.0.0.1:6379> lrange list 0 0		# 如果存在，更新当前下标的值
1) "value1"
127.0.0.1:6379> lset list 0 item
OK
127.0.0.1:6379> Lrange list 0 0
1) "item"
127.0.0.1:6379> lset list 1 value
(error) ERR index out of range
~~~





### Set（集合）







### Hash（哈希）

Map集合，key-map 这个值是一个map集合！

> hset key field value

~~~bash
127.0.0.1:6379> hset myhash field1 wangweijun
(integer) 1
127.0.0.1:6379> hget myhash field1
"wangweijun"
127.0.0.1:6379> hmset myhash field1 hello field2 world
OK
127.0.0.1:6379> hmget myhash field1 field2
1) "hello"
2) "world"
~~~





## Jedis

### Redis.conf详解















### Redis持久化

Redis是内存数据库，如果不将内存中的数据库状态保存到磁盘，那么一旦服务器进程退出，服务器中的数据库状态也会消失，所以Redis提供了持久化功能！



#### RDB（Redis DataBase）

> 什么是RDB







#### AOF（Append Only File）

将我们的所有命令都记录下来，history，恢复的时候就把这个文件全部执行一遍！

> AOF



### Redis发布订阅

Redis发布订阅（pub/sub）是一种==消息通信模式==：发送者（pub），订阅者（sub）接收消息。

Redis客户端可以订阅任意数量的频道。



### Redis主从复制









### Redis缓存穿透和雪崩

































