# list 数据结构



list 是一键多值



之前的String类型对应的存储结构是一对一的关系，key-value



![1657455934892](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657455934892.png)

list类型对应的存储结构，其实是一个双向链表，可以当作栈，也可以当作队列，对两端的操作性能很高，对中间节点的操作性能价差（链表查询慢）

![1657456129977](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657456129977.png)





了解详细list的数据结构，可以看下面这篇文章



 [Redis列表list 底层原理 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/102422311) 





# list常用命令



list 一般命令都以 l 开头，除非是涉及到方向的指令



## lRange 查看key指定位置的数据



```bash
lrange key start end
```



通常使用 start=0,end=-1,来查看key中全部的数据



```bash
127.0.0.1:6379> lrange list1 0 -1
k4
k3
k2
k1
127.0.0.1:6379> lrange list1 0 2
k4
k3
k2
```



## lpush 创建key从左边插入数据



```bash
lpush key element [element]...
```



向当前的 list 类型的 key 中从左边开始插入数据，可以插入多个



```bash
127.0.0.1:6379> lpush list1 k1 k2 k3 k4
4
```

返回插入成功的数据的个数



上述插入的指令效果

![1657456589898](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657456589898.png)



我们查询list中的数据

```bash
127.0.0.1:6379> lrange list1  0 -1
k4
k3
k2
k1
```



&emsp;&emsp;发现存储的数据确实从k4 左边到右边的顺序，证实了lpush 是从左边插入的，如果单边一直插入，那么类似于栈的效果。



## lpop 已有的key从左边弹出一个value

```bash
lpop key
```





## Rpush 创建从右边插入数据

```bash
Rpush key element [element]...
```



我们创建一个新的key,从右边插入数据，可以插入多个



```bash
127.0.0.1:6379> lpush list2 k1 k2 k3 k4
4
127.0.0.1:6379> rpush list k1 k2 k3 k4
4
127.0.0.1:6379> lrange list 0 -1
k1
k2
k3
k4
```



查询list之后发现，rpush存储的数据是从右边开始插入的，下面是右插的数据结构



![1657456928298](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657456928298.png)

## rpop 已有的key从右边弹出一个value



```bash
rpop key
```





## linsert 在指定位置插入value



```bash
linsert key before|after pivot element
```



pivot 必须是当前 key 中的一个具体的value，在value的前面或者后面插入 element，



如果pivot不存在，那么插入就没效果，返回0，如果插入成功的话，会返回最终list中的元素个数



before|after  选择在目标值的前插还是后插



```bash
127.0.0.1:6379> rpush name zhangsan lisi wangwu # 创建list类型的key,插入数据
3
127.0.0.1:6379> lrange name 0 -1 # 查询数据
zhangsan
lisi
wangwu
127.0.0.1:6379> linsert name before lisi 666 # 在 name 中 lisi 的前面插入 666
4 # 返回最终的list元素个数
127.0.0.1:6379> lrange name 0 -1  # 查询数据
zhangsan
666           #插入成功
lisi
wangwu
```







## lrem 删除当前 key中的 value



rem==remove



```bash
lrem key count value [value]...
```



count>=value 的个数，否则会报错







## llen 查询当前key的数据长度



```bash
llen key 
```



返回key中数据的个数



## lindex 查询当前key指定下标数据



```bash
lindex key index
```



查询key的指定 index下标的数据，index如果超出范围那么返回为空



```bash
127.0.0.1:6379> lindex k1 0
v4
```





## ltrim 截取当前key中指定范围下标的数据



```bash
ltrim key start end
```



trim在字符串中意味着去除首尾空格，在这里是截取当前 key 的 [start,end] 中的数据



```bash
127.0.0.1:6379> lrange k1 0 -1 # 查询key中的数据
v4 # 0
v3  # 1
v2  # 2
127.0.0.1:6379> ltrim k1 1 2 # 截取1下标到2下标中的元素 [1,2]
OK
127.0.0.1:6379> lrange k1 0 -1 # 查询数据
v3
v2

```



## lset 替换指定下标对应的value 



```bash
lset key index value
```





```bash
127.0.0.1:6379> lrange name 0 -1 # 查询数据
zhangsan
666
lisi
127.0.0.1:6379> lset name 1 "helloworld" # 将下标1的666替换成 helloworld
OK
127.0.0.1:6379> lrange name 0 -1 # 查询数据，替换成功
zhangsan
helloworld
lisi

```



## RpopLpush 移除队尾元素放到队头



```bash
rpoplpush key source destination
```



将一个key的队尾元素，放到另一个key的队头



```bash
127.0.0.1:6379> lrange name 0 -1 # 查询name数据
zhangsan
helloworld
lisi
127.0.0.1:6379> rpoplpush name name #相当于把自己的队尾元素放到队头了
lisi
127.0.0.1:6379> lrange name 0 -1 # 放置成功
lisi
zhangsan
helloworld
```

