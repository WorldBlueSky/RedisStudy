# Redis 学习（一） ---- Redis 的安装



Redis 官方更推荐 Linux系统的使用，window不推荐。



![1656910090459](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656910090459.png)



## Window安装



### 1、github下载 zip



redis 的 github下载地址



 [Releases · tporadowski/redis · GitHub](https://github.com/tporadowski/redis/releases) 



如果github不能访问，那么看下面这篇文章



 [GitHub520: 本项目无需安装任何程序，通过修改本地 hosts 文件，试图解决： GitHub 访问速度慢的问题 GitHub 项目中的图片显示不出的问题 花 5 分钟时间，让你"爱"上 GitHub。 (gitee.com)](https://gitee.com/klmahuaw/GitHub520) 





### 2、下载好的压缩包



![1656908957839](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656908957839.png)





> redis-server-exe   ----> 开启redis服务  



> redis-cli.exe       - --->  redis 客户端程序



> redis-check-aop.exe  ----> 查看 redis 的aop文件是否正常



> redi-benchmark.exe  --->  redis 性能测试的程序  





### 3、开启redis服务，客户端测试是否连通



点击redis-server.exe，开启 redis服务，默认端口号是 6379



![1656909425967](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656909425967.png)



点击redis-cli.exe ,打开redis 客户端，测试是否连接



如果未打开 redis 服务，无法连接 redis



![1656909528241](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656909528241.png)



成功连接之后，通过ping命令测试是否连通



![1656909575124](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656909575124.png)



### 4、redis 简单操作



通过 set 和 get 的方式，在redis中进行简单存储，**输入语句之后不是以分号作为结束，不能输入分号**



![1656909768276](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656909768276.png)





## Linux 安装



## （1）环境安装 gcc



基本的环境安装gcc， 系统自带gcc版本是4.8.5，只需要更新下gcc到5.4以上即可 



查看gcc版本号

```shell
gcc -v
```

![1656912749512](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656912749512.png)



安装gcc

```shell
yum install gcc
```



升级gcc

```shell
yum -y install centos-release-scl
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
scl enable devtoolset-9 bash
```



需要注意的是scl命令启用只是临时的，退出shell或重启就会恢复原系统gcc版本。如果要长期使用gcc 9.3的话：

```shell
echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
```



## （2）安装 redis 源码文件



安装redis源码文件压缩包tar

```shell
wget http://download.redis.io/releases/redis-6.0.8.tar.gz
```

![1656912831510](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656912831510.png)



解压缩

```shell
 tar xzf redis-6.0.8.tar.gz
```

![1656912799438](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656912799438.png)





进入redis-6.0.8 文件夹，可以看到 redis-conf 配置文件



![1656912968248](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656912968248.png)



## （3）安装相关文件及运行环境



安装 redis 相关的文件以及环境，在redis-6.0.8 目录下，不是src下

```shell
make
```

文件安装完成，生成了 src文件

![1656913195991](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656913195991.png)



检查是否安装完成，此时已经将redis程序安装到 服务器的默认路径了，之前的文件是C源码文件

```shell
make install
```



![1656913497259](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656913497259.png)





## （4）查看根据源码安装的 redis 相关程序

进入到redis 默认安装路径  ==**/usr/local/bin**==

安装好的redis程序在这个目录下



![1656913812601](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656913812601.png)





## （5）修改Redis 程序的配置文件



在当前目录下 创建 一个放配置文件的目录 myconfig

```shell
mkdir myconfig
```



![1656914118462](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656914118462.png)



将 ==/root/redis-6.0.8/redis.conf== 拷贝到 当前目录的文件夹 ==myconfig== 中

```shell
cp /root/redis-6.0.8/redis.conf myconfig
```



![1656914228070](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656914228070.png)



## （6）修改具体配置



redis 服务默认不是后台启动的，修改配置文件

```shell
vim redis.conf
```



i 进入编辑模式， :wq  保存退出



![1656914646920](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656914646920.png)



开启redis-server ，通过 myconfig/redis.conf 配置文件进行开启

```shell
redis-server myconfig/redis.conf
```



查看redis 的进程

```shell
ps -ef | grep redis
```

![1656915134073](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656915134073.png)



如果忘记加配置文件，那么杀死 redis 进程重新开启

```shell
kill -9 [pid]
```



redis 服务已开启，我们 开启指定的端口号（-p）的redis 客户端程序

```shell
redis-cli -p 6379
```



成功连接，简单操作可以使用



![1656915208844](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656915208844.png)



客户端输入 shutdown 关闭 redis服务，终止 redis服务进程，按 exit 退出客户端，此时 redis 服务以及客户端进程全部结束



![1656915988364](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1656915988364.png)



只输入exit 退出客户端，不关闭 redis 服务

```shell
exit
```



