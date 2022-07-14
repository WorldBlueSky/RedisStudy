



 [Redis 中 String（字符串）类型的命令](https://www.redis.com.cn/string.html) 



## set  存一个键值对 key-value



```set bash
set key value
```



 Redis SET 命令用于设置给定 key 的值。如果 key 已经存储其他值， SET 就覆写旧值，且无视类型。 



## get  根据key 取 value

 Redis Get 命令用于获取指定 key 的值。如果 key 不存在，返回 nil 。如果key 储存的值不是字符串类型，返回一个错误。 



```bash
get key
```



```bash
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> get k1
"v1"
```



## append 对 key 的value进行追加字符串



&emsp;&emsp;如果 key 已经存在并且是一个字符串， APPEND 命令将 value 追加到 key 原来的值的末尾。

如果 key 不存在， APPEND 就简单地将给定 key 设为 value ，就像执行 SET key value 一样。



返回值是追加指定值之后， key 中字符串的长度。



```bash
append key value # 如果当前key 不存在，那么相当于 set key
```



```bash
127.0.0.1:6379> set user hello
OK
127.0.0.1:6379> get user
"hello"
127.0.0.1:6379> append user redis # 在原有key 的value后面进行追加操作
(integer) 10      # 返回的是追加后的字符串长度
127.0.0.1:6379> get user
"helloredis"
```



## mset 批量增加 key-value

```bash
mset key value [key] [value] ...
```



mset 设置 k2-v2, k3-v3 ，批量增加字符串键值对

```bash
127.0.0.1:6379> mset k2 v2 k3 v3
OK
127.0.0.1:6379> keys *
1) "k2"
2) "k1"
3) "k3"
```



## mget 批量获取 value

```bash
mget k1 [key] [key]
```

```bash
127.0.0.1:6379> keys *
1) "k2"
2) "k1"
3) "k3"
4) "k4"
127.0.0.1:6379> mget k1 k2 k3 k4
1) "v1"
2) "v2"
3) "v3"
4) "v4"
```





## strlen 统计key存储的 value的长度



 Strlen 命令用于获取指定 key 所储存的字符串值的长度。当 key 储存的不是字符串值时，返回一个错误。 

返回值是字符串值的长度。 当 key 不存在时，返回 0。 

```bash
127.0.0.1:6379> get user
"helloredis"
127.0.0.1:6379> strlen user
(integer) 10
```



## incr 自增操作 i++



incr 给当前的key 自增，相当于++操作，value必须是数字

```bash
incr key
```



如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 INCR 操作。

如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。



设置比如说B站视频播放量 views 为 0 ,只要用户点击那么执行自增操作++

```bash
127.0.0.1:6379> set views 0
OK
127.0.0.1:6379> get views
"0"
127.0.0.1:6379> incr views
(integer) 1
127.0.0.1:6379> get views
"1"
```



## incrby 按照步长增加 i+=increment



incrby 就是给指定的key 进行指定增加大小

```bash
incrby key increment
```



如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 INCRBY 命令。

如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。



incrby 返回的是增加过后 key 的结果，可以给指定key 增加指定的大小



```bash
127.0.0.1:6379> get views
"1"
127.0.0.1:6379> incrby views 10
(integer) 11
127.0.0.1:6379> get views
"11"
```



## decr 自减操作 i--

```bash
decr key
```



```bash
127.0.0.1:6379> get views
"11"
127.0.0.1:6379> decr views
(integer) 10
127.0.0.1:6379> get views
"10"
```



## decrby 按照步长减少 i-=decrement



```bash
decrby key decrement
```



```bash
127.0.0.1:6379> get views
"10"
127.0.0.1:6379> decrby views 5
(integer) 5
127.0.0.1:6379> get views
"5"

```



## getRange 获取某个范围的值



```bash
getrange key start end
```



获取key对应字符串的下标从start到end的值（start 从0开始）



```bash
127.0.0.1:6379> set k1 "helloRedis"
OK
127.0.0.1:6379> get k1
"helloRedis"
127.0.0.1:6379> getrange k1 0 2 #截取字符串[0，2]下标的全部字符串
"hel"
```



如果end为负数，就是从字符串右边开始数第几个字符，-1就是倒数第一个字符,-2倒数第二个字符



```bash
127.0.0.1:6379> getrange k1 0 -2
"helloRedi"
```



0到 -1 通常是作为查询这个key对应的value 的命令，可以查到value的全部字符



```bash
127.0.0.1:6379> getrange k1 0 -1 # 等同于 get key
"helloRedis"
```



## setRange  替换指定范围的值



```
set key offset value
```

offset 从0开始，代表偏移量，对应从哪个下标开始修改



```BASH
127.0.0.1:6379> set name zhang3
OK
127.0.0.1:6379> get name
"zhang3"
127.0.0.1:6379> setRange name 5 z # 从5下标开始修改，替换后面的value
(integer) 6
127.0.0.1:6379> get name
"zhangz"
127.0.0.1:6379> setRange name 0 rain7666 # 从0下标修改，后面进行替换
(integer) 8
127.0.0.1:6379> get name
"rain7666"
```





## setnx key存在设置失败

```bash
setnx key
```



**setnx （set if not exists）**



如果key存在那么设置失败，如果不存在，那么设置成功

很像mysql中的 不存在再建库建表的语句

```mysql
create database if not exists databasename
```



如果redis中不存在k1，那么正常这是与set功能一样

```bash
127.0.0.1:6379> setnx k1 v1
(integer) 1
127.0.0.1:6379> get k1
"v1"
```



如果redis中存在k1，那么设置失败,返回0

```bash
127.0.0.1:6379> setnx k1 vvv1
(integer) 0
127.0.0.1:6379> get k1
"v1"
```





## setex 设置key及过期时间



```bash
setnx key seconds value
```



```bash
127.0.0.1:6379> setex k3 10 v3 # 设置k3-v3 10秒过期
OK
127.0.0.1:6379> get k3 #此时获取k3 ，拿到v3
"v3"
127.0.0.1:6379> ttl k3  # 剩余3秒
(integer) 3
127.0.0.1:6379> ttl k3 # 剩余2秒
(integer) 2
127.0.0.1:6379> ttl k3 #剩余1秒
(integer) 1
127.0.0.1:6379> ttl k3
(integer) -2
127.0.0.1:6379> get k3 #此时再去获取k3,拿到空值
(nil)

```



**(set with expire)**



## msetnx 批量不存在再设置 



```bash
127.0.0.1:6379> msetnx k1 v1 k2 v2
(integer) 0
```

重点：msetnx 也是一个原子性的操作，要么一起成功，要么一起失败



没有 msetex



## 存储对象类型



json 格式的字符串表示对象类型的内容



```bash
127.0.0.1:6379> set user:1 {name:rain7,age:18} 
OK                 #用 json 字符串存储 user:1 这个对象：代表分级  1代表{id}
127.0.0.1:6379> get user:1
"{name:rain7,age:18}"

```



普通键值对表示对象类型的内容

对象层级更加明确清晰，对象中的某些属性可以进行复用，非常方便

```bash
127.0.0.1:6379> mset user:1:name rain7 user:1:age 18
OK
127.0.0.1:6379> get user:1
"{name:rain7,age:18}"
127.0.0.1:6379> get user:1:name
"rain7"
127.0.0.1:6379> get user:1:age
"18"
```



## getset 先获取再设置（更新）



```bash
getset key value
```



这是一个组合命令，相当于返回旧值之后用设置新值,常用于更新操作



如果key 不存在那么返回nil，设置新的值

如果key存在那么返回旧值，设置新的值



```bash
127.0.0.1:6379> set name zhangsan 
OK
127.0.0.1:6379> get name
"zhangsan"
127.0.0.1:6379> getset name rain7666
"zhangsan"
127.0.0.1:6379> get name
"rain7666"
```



这是一个原子操作，与CAS非常类似



## 小结



&emsp;&emsp;String类型的使用，value除了可以是字符串，还可以是数字，可以作为计数器进行使用（incr,decr）应用于统计数量相关的属性（阅读量、播放量、点赞数）,可以用用于对象存储等。