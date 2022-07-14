# Redis 学习（六）---- Set 常用指令





set 是一个无序的、不重复的 集合，如果我们想要存储list这样的数据，又不想重复的话，那么set类型可以。



set 类型的指令 都是 s开头的



## sadd 给集合中添加元素

```bash
sadd key member1 [member2]...
```



给集合中添加一个元素，返回增加成功的元素的数量

```bash
127.0.0.1:6379> sadd set1 v1 v2 v3 v4
(integer) 4
```



## smembers 查看集合中所有成员



```bash
smemebers key 
```



smember  来查看 set类型的key中 所有元素的value

```bash
127.0.0.1:6379> smembers set1
1) "v4"
2) "v3"
3) "v2"
4) "v1"
```



## srem 移除集合中的元素



```bash
srem key element [element]...
```



srem命令可以移除集合下一个或者多个元素



srem 来移除 set1 中 v1 元素



```bash
127.0.0.1:6379> smembers set1
1) "v4"
2) "v3"
3) "v2"
4) "v1"
127.0.0.1:6379> srem set1 v1 # 移除set1 中v1元素
(integer) 1
127.0.0.1:6379> smembers set1  # 再次查看移除成功
1) "v4"
2) "v3"
3) "v2"
```



## scard 获取集合中元素数量



```bash
scard key
```



scard 可以获取集合中的元素数量



```bash
127.0.0.1:6379> smembers set1
1) "v4"
2) "v3"
3) "v2"
127.0.0.1:6379> scard set1
(integer) 3

```



## smove 转移一个集合元素到另一个集合中



```bash
smove source distination element
```



将 set2 中的 v1 元素 转移到 set1 中

```bash
127.0.0.1:6379> sadd set1 k1 k2 k3 k4
(integer) 4
127.0.0.1:6379> sadd set2 v1 v2 v3 v4
(integer) 4
127.0.0.1:6379> smove set2 set1 v1
(integer) 1
127.0.0.1:6379> smembers set1
1) "k2"
2) "k4"
3) "k3"
4) "k1"
5) "v1"

```



## spop 随机弹出集合中的一个元素



```bash
spop key
```



spop 是随机弹出 一个集合中的元素，弹出后返回弹出的元素



```bash
127.0.0.1:6379> smembers set1 # 查看set1中的全部成员
1) "k2"
2) "k4"
3) "k3"
4) "k1"
5) "v1"
127.0.0.1:6379> spop set1 # 随机弹出一个元素 k1
"k1"
127.0.0.1:6379> smembers set1 # 再次查看，发现k1已经被弹出了
1) "k2"
2) "v1"
3) "k3"
4) "k4"
```



## sismember 查看元素是否属于当前集合



```
sismember key value
```



查看当前value 是否属于 当前key，如果属于 返回 1，如果不属于返回0



```bash
127.0.0.1:6379> smembers set1
1) "k2"
2) "v1"
3) "k3"
4) "k4"
127.0.0.1:6379> sismember set1 666
(integer) 0
127.0.0.1:6379> sismember set1 k2
(integer) 1
```



## srandmember 随即返回几个集合中的元素



```bash
srandmember key count
```



随机返回当前 key 中 count 个 元素



```bash
127.0.0.1:6379> smembers set1
1) "k2"
2) "v1"
3) "k3"
4) "k4"
127.0.0.1:6379> srandmember set1 3
1) "k2"
2) "k4"
3) "k3"
```





## 交集、并集、差集



![1657529943994](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657529943994.png)



![1657529982014](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657529982014.png)



![1657530026317](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657530026317.png)



## sinter  寻找多个集合的交集

```bash
sinter key [key]...
```



## sunion 寻找多个集合的并集

```bash
sunion key [key]...
```



## sdiff 寻找多个集合的差集

```bash
sdiff key [key]
```



下面通过 set1 、set2 来说明指令的使用



```bash
127.0.0.1:6379> sadd set1 k1 k2 k3 v1 v2 v3
(integer) 6
127.0.0.1:6379> sadd set2 k2 v2 a b c d
(integer) 6
127.0.0.1:6379> sinter set1 set2 # set1 与 set2 并集的元素
1) "k2"
2) "v2"
127.0.0.1:6379> sdiff set1 set2 #  set1 中与set2 不同的元素
1) "k1"
2) "k3"
3) "v3"
4) "v1"
127.0.0.1:6379> sunion set1 set2  # set1 与 set2 所有的元素
 1) "v1"
 2) "v2"
 3) "k3"
 4) "v3"
 5) "d"
 6) "k1"
 7) "b"
 8) "a"
 9) "k2"
10) "c"
```