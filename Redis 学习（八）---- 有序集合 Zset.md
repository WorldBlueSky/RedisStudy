## Redis 学习（八）---- 有序集合 Zset



Zset 和 set 非常的类似。是一个**没有重复元素**的**有序**集合




&emsp;&emsp;zset中每个成员都有一个 score ， score用来按照从最低分到最高分进行排序，集合中的成员是唯一的，但是评分是可以重复的



## （1）zadd 将元素放到 zset集合中



```bash
zadd key score member [score] [member]...
```



将一个或者多个元素及其 score 放到zset集合中。存的时候按照 score 从低到高 依次存入到集合中



```bash
127.0.0.1:6379> zadd topn 200 java 300 c++ 400 mysql 500 php
(integer) 4
```



## （2）zrange 分数按照升序展示



```bash
zrange key start stop
```



因为存的时候是按照score从低到高排的，可以按照索引进行查询



```bash
127.0.0.1:6379> zrange topn 0 3
1) "java"
2) "c++"
3) "mysql"
4) "php"
127.0.0.1:6379> zrange topn 0 -1
1) "java"
2) "c++"
3) "mysql"
4) "php"
```



如果我们想要显示评分的话，那么在后面再加参数



```bash
127.0.0.1:6379> zrange topn 0 -1 withscores
1) "java"
2) "200"
3) "c++"
4) "300"
5) "mysql"
6) "400"
7) "php"
8) "500"
```



在命令后面加上 withscore ，显示的结果中 member 和 score 是紧靠排列的



## （3）zrangebyscore查询key中指定分数范围的结果



```bash
zrangebyscore key min max [withscores]
```



查询 topn 中 300 到400 之间的结果

```bash
127.0.0.1:6379> zrangebyscore topn 300 400 withscores
1) "c++"
2) "300"
3) "mysql"
4) "400"

```



## （4）zrevrange 将分数按照降序展示



```bash
zrevrange key start end
```





```bash
127.0.0.1:6379> zrevrange topn 0 -1
1) "php"
2) "mysql"
3) "c++"
4) "java"
```



## （5） zrevrangebyscore 分数高分到低分范围内结果展示



```bash
zrevrangebyscore key max min [withscores]
```



前一个是分数最大值，后一个是分数最小值，在这个范围内进行查找



```bash
127.0.0.1:6379> zrevrangebyscore topn 400 300
1) "mysql"
2) "c++"
```



## （）zcard 返回key中所有的成员数

```bash
zcard key
```



```bash
127.0.0.1:6379> zrange topn 0 -1 withscores
1) "java"
2) "200"
3) "c++"
4) "300"
5) "mysql"
6) "400"
7) "php"
8) "500"
127.0.0.1:6379> zcard topn
(integer) 4

```





## （）zcount 得到key指定分数范围的成员数

```bash
zcount key min max
```



```bash
127.0.0.1:6379> zrange topn 0 -1 withscores
1) "java"
2) "200"
3) "c++"
4) "300"
5) "mysql"
6) "400"
7) "php"
8) "500"
127.0.0.1:6379> zcount topn 300 500 
(integer) 3
```



## （）zincrby 为key的元素score加上增量



```bash
zincrby key incrment member
```



为key中的 指定 member 的 score 加上 incrment 的增量，通过负数实现减法



## （）zrem 删除集合中的member



```bash
zrange key member [member]...
```



删除一个或者多个key中的member



```bash
127.0.0.1:6379> zrange topn 0 -1
1) "java"
2) "c++"
3) "mysql"
4) "php"
127.0.0.1:6379> zrem topn mysql
(integer) 1
127.0.0.1:6379> zrange topn 0 -1
1) "java"
2) "c++"
3) "php"
```





## （）zremrangebyscore  按照分数范围进行删除

```bash
zremrangebyscore key min max
```

```bash
127.0.0.1:6379> zremrangebyscore topn 100 200
(integer) 1
```



## （）zrank 返回该member 在 key 中的排名（低到高）,从0开始

```bash
zrank key member
```

返回member的索引，从0 开始，分数从低到高排



```bash
127.0.0.1:6379> zrange topn 0 -1
1) "c++"
2) "php"
127.0.0.1:6379> zrank topn c++
(integer) 0
```



## （）zrevrank 返回该member在key中的排名（高到低），从0开始



```bash
zrevrank key member
```



列表中元素按照从高到低排列，返回member的索引，索引从0开始



查找文章001 在评分排行榜中的排名



```bash
127.0.0.1:6379> zrevrange article 0 -1 withscores
文章002
500
文章001
300
文章004
150
文章003
100
127.0.0.1:6379> zrevrank article 文章001
1

```





## 如何实现一个文章访问量的排行榜？



设置文章的评分分数

```bash
127.0.0.1:6379> zadd article 300 文章001 500 文章002 100 文章003 150 文章004
```



展示排行榜，按照分数从高到低展示

```bash
127.0.0.1:6379> zrevrange article 0 -1 withscores
文章002
500
文章001
300
文章004
150
文章003
100
```

