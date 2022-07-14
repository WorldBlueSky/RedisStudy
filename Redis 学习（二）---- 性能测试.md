Redis 性能测试





redis-benchmark  是 redis 官方自带的性能测试软件，通过指定参数进行测试



![1656919609318](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656919609318.png)



redis的默认选项,测试参数默认设置



> -c 并发redis客户端连接默认50  

> -n 默认请求数为10000 

> -d 默认写入数据大小为2个字节（已更新为3个字节）

> -k 默认只存在一台redis服务器



我们来测试一下，100个并发的连接，100000个请求下redis的性能

```shell
redis-benchmark -h localhost -p 6379 -c 100 -n 100000
```



测试的结果，包括对网络连通的测试，redis中各种命令的测试



![1656920332090](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656920332090.png)



测试分析



那么这个测试的结果怎么查看呢？以测试 set 写入为例子

![1656920597360](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656920597360.png)