# Redis 学习（七）---- Hash 指令



Hash 是一个键值对的集合

是一个field 和 value 的映射表，经常用户存储对象

![1657731822891](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657731822891.png)



Hash 类型的指令通常以 h开头



![1657732288638](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657732288638.png)

字符串存储对应类型由以上两种方式

```bash
key       value
user      {id=1,name=zhangsan,age=20} --- 第一种

=====第二种======
user:id    1
user:name zhangsan 
user:age   20
```



&emsp;&emsp;第一种对 对象内部属性修改造成很大的不方便



&emsp;&emsp;第二种数据太分散了，造成存储很混乱，有100个user对象，每个user对象中又有十几个user对象，最后造成很混乱。



所以就有了第三种方式，使用hash 类型存储对象



![1657732570015](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657732570015.png)



==key-field-value== 

存储更加方便,改值也很方便



## （1）hset  设置Hash类型key



设置hash类型的key，后跟一个或者多个field-value



```bash
hset [key] [field] [value]...
```



给user 的key中添加 id=1,name=zhangsan ,age=18 几个键值对



```bash
127.0.0.1:6379[2]> hset user id 1 name zhangsan age 18
(integer) 3
```



返回的是成功设置的键值对个数





## （2）hget 获取key 中指定 field 的 value



输入key 以及对应的field，返回对应的value

```bash
hget [key] [field]
```



获取user 当中对应 field 的 value

```bash
127.0.0.1:6379[2]> hget user name
"zhangsan"
127.0.0.1:6379[2]> hget user id
"1"
127.0.0.1:6379[2]> hget user age
"18"

```



## （3）hdel 删除 key 中对应的 field



```bash
hdel key [field]...
```



删除key中一个或者多个 filed



```bash
127.0.0.1:6379[2]> hkeys user1
1) "id"
2) "name"
3) "age"
4) "username"
5) "password"
127.0.0.1:6379[2]> hdel user1 name age
(integer) 2
127.0.0.1:6379[2]> hkeys user1
1) "id"
2) "username"
3) "password"

```





## （4）hexists 查看 key 中 field 是否存在



查看key 的field是否存在，如果不存在返回0，如果存在返回1



```bash
hexists key field 
```



设置user1，id=1,name=lisi,age=18

查看user1 的name 是否存在，存在返回 1

查看user1 的 gender 是否存在，不存在返回 0



```bash
127.0.0.1:6379[2]> hset user1 id 1 name lisi age 18
(integer) 3
127.0.0.1:6379[2]> hget user1 name
"lisi"
127.0.0.1:6379[2]> hexists user1 name
(integer) 1
127.0.0.1:6379[2]> hexists user1 gender
(integer) 0
```



## （5）hkeys 查看 key 中的所有 field

```bash
hkeys key
```

查看key中的所有field

```bash
127.0.0.1:6379[2]> hkeys user1
1) "id"
2) "username"
3) "password"
```



## （6）hvals 查看 key 中的所有 value

```bash
hvals key
```

查看key中所有value值

```bash
127.0.0.1:6379[2]> hvals user1
1) "1"
2) "admin"
3) "123456"
```



## （7）hgetall 查看 key 中的所有 field-value

 ```bash
hgetall key 
 ```



返回key中的 field 和 value

在返回值里，紧跟每个字段名(field name)之后是字段的值(value)，所以返回值的长度是哈希表大小的两倍。 



```bash
127.0.0.1:6379[2]> hgetall user1
1) "id"
2) "1"
3) "name"
4) "lisi"
5) "age"
6) "18"
```



## （8）hlen 查看key中的所有字段数量



```bash
hlen key
```



返回 key 中 field 的数量



```bash
127.0.0.1:6379[2]> hkeys user1
1) "id"
2) "username"
3) "password"
127.0.0.1:6379[2]> hlen user1
(integer) 3

```





## （9）hincrby 增加、减少某个 field 的 value



```bash
hincrby key field incement
```



在这里说明一下，hash类型没有decr、incr、decrby，hash类型的减法是通过 加负数来完成的



```bash
127.0.0.1:6379[2]> hset user id 1
(integer) 1
127.0.0.1:6379[2]> hincrby user id 10
(integer) 11
127.0.0.1:6379[2]> hget user id
"11"
```



## （10）hsetnx 存在及设置失败

```bash
hsetnx key field value ...
```



如果key存在field ，那么设置失败，如果不存在field，那么设置成功



## （11）hash 数据结构



Hash类型的key 数据类型



- 当key的filed-value 长度较短且个数较少时，使用的是ziplist（压缩列表）

  

- 否则使用 hashtable