# Redis 非关系型数据库学习（三）---- Redis 基础操作



&emsp;&emsp;在之前的学习中，我们已经在Linux系统上安装了Redis，之后的所有操作都在Linux 系统上完成操作



## 基础知识



## （1）Redis 数据库



- ### select 切换当前数据库

  

redis 默认的数据库数量为16，可以通过查看redis配置文件得知

![1656936308526](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656936308526.png)



redis默认的数据库索引为0 ,我们可以 通过 select 选择一个具体索引的数据库

```shell
select [DBId]
```



![1656936562836](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656936562836.png)



通过select 可以切换到不同的数据库，同时客户端会显示当前数据库的索引号。



- ### Dbsize 查看数据库key数量

  

可以通过 Dbsize 查看当前数据库中的key的数量个数



我们在空的数据库中放一个 key-value

```shell
127.0.0.1:6379[3]> set name admin
OK
```



直接输入命令 Dbsize，可以查看到当前数据库中的key数量

```shell
127.0.0.1:6379[3]> dbsize
(integer) 1
```



## （2）查看数据库的key



- ### keys  [partten]



keys 命令可以查看当前数据库指定信息的key 



插入了两个 key

```shell
127.0.0.1:6379> set name root
OK
127.0.0.1:6379> set names admin
OK
```



我们想要查找 以 na 开头的 key 信息

```shell
127.0.0.1:6379> keys na*
1) "names"
2) "name"
```



查找所有的key

```shell
127.0.0.1:6379> keys *
1) "names"
2) "name"
```



## （3）清除数据库的 key



- ### flushdb 清除当前db的key



这条指令用来清除当前数据库中 所有的key信息



```shell
127.0.0.1:6379> keys *
1) "names"
2) "name"
127.0.0.1:6379> flushdb # 清除当前数据库中所有key
OK
127.0.0.1:6379> keys *
(empty array)

```



- ### flushall 清除所有db的key



这条命令用来清除所有数据库中的 key 信息



```shell
127.0.0.1:6379> flushdb # 清除所有数据库中所有key
OK
```



## （4）设置key值 , 获取key值



- ### set 设置key值 

 Redis SET 命令用于设置给定 key 的值。如果 key 已经存储其他值， SET 就覆写旧值，且无视类型。 

- ### get 获取key值

 Redis Get 命令用于获取指定 key 的值。如果 key 不存在，返回 nil 。如果key 储存的值不是字符串类型，返回一个错误。 

```shell
127.0.0.1:6379> set name RAIN7 # set [key] [value] 设置key 与其对应的 value
OK
127.0.0.1:6379> get name       # get [key]   根据key 值拿到 对应的value
"RAIN7"
```



## （5）查看 指定的key 是否存在



- ### exists 查看key是否存在

  

 Redis EXISTS 命令用于检查给定 key 是否存在。 

 若 key 存在返回 1 ，否则返回 0 。 



```shell
127.0.0.1:6379> set name RAIN7
OK
127.0.0.1:6379> get name
"RAIN7"
127.0.0.1:6379> exists name
(integer) 1

```

可以用 exists 查看 key 是否存在



## （6）查看 key 的类型



- ### type 查看key的类型



Redis Type 命令用于返回 key 所储存的值的类型。 

返回 key 的数据类型，数据类型有：

- none (key不存在)
- string (字符串)
- list (列表)
- set (集合)
- zset (有序集)
- hash (哈希表)





```shell
127.0.0.1:6379> set name RAIN7
OK
127.0.0.1:6379> get name
"RAIN7"
127.0.0.1:6379> type key
none
127.0.0.1:6379> type name
string
```



type + keyname ,可以查看对应key 的类型，如果key 不存在返回 none



## （7）设置查看 key 的过期时间



- ### expire 设置key的过期时间

  

 Redis Expire 命令用于设置 key 的过期时间，key 过期后将不再可用。单位以秒计 



 设置成功返回 1 。 当 key 不存在返回 0 。 



- ### ttl 查看key 剩余过期时间

  

&emsp;&emsp;当 key 不存在时，返回 -2 。 当 key 存在但没有设置剩余生存时间时，返回 -1 。 否则，以秒为单位，返回 key 的剩余生存时间。



**注意：**在 Redis 2.8 以前，当 key 不存在，或者 key 没有设置剩余生存时间时，命令都返回 -1 。



```shell
127.0.0.1:6379> expire name 10 # 设置 name 的过期时间为 10秒
(integer) 1
127.0.0.1:6379> ttl name  # 查看 name 剩余的过期时间
(integer) 3
127.0.0.1:6379> ttl name
(integer) 2
127.0.0.1:6379> ttl name
(integer) 1
127.0.0.1:6379> ttl name
(integer) 0
127.0.0.1:6379> ttl name  # 此时 name 已经不存在，返回-2
(integer) -2
```



## （8）Redis默认端口号6379的由来



讲一点题外的小知识，为什么 Redis的默认端口号是 6379？、



&emsp;&emsp;Merz（梅尔兹） 是一个女明星，Redis 作者 Antirez (安提雷兹) 早年看电视节目，觉得 Merz 在节目中的一些话愚蠢可笑，Antirez 喜欢造“梗”用于平时和朋友们交流，于是造了一个词 "MERZ"，形容愚蠢，与 "[stupid](https://www.zhihu.com/search?q=stupid&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A645322396})" 含义相同。



&emsp;&emsp;后来 Antirez 重新定义了 "MERZ" ，形容”具有很高的技术价值，包含技艺、耐心和[劳动](https://www.zhihu.com/search?q=劳动&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A645322396})，但仍然保持简单本质“。



&emsp;&emsp;到了给 Redis 选择一个数字作为默认端口号时，Antirez 没有多想，把 "MERZ" 在[手机键盘](https://www.zhihu.com/search?q=手机键盘&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A645322396})上对应的数字 6379 拿来用了。



&emsp;&emsp;还有一个基础的知识，在这里先不提，那就是 Redis 单线程与多线程的问题，在这里先不提了，在后面会重新说。

