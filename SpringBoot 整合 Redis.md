# SpringBoot 整合 Redis



jedis：采用直连的server，多个线程操作是不安全的，使用jedis pool连接池 BIO



lettuce：采用netty，实例可以在多个线程中共享，不存在线程不安全的情况，可以减少线程数量





springboot中redis的自动配置类（在springAutoConfigure 的 jar包中找到）



![1657357872768](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657357872768.png)



（1）这里的RedisTemplate中的设置都是默认的，可以通过这个对象对redis的一些配置进行设置

（2）通常我们使用泛型，前面那个参数设置成 String,否则在设置的时候需要强制类型转换 <String,Object>



**我们自己可以在注册一个bean，RedisTemplate来对redis进行一些配置。**



## 一、导入spring-data-redis 的依赖



在springboot创建项目的时候就可以添加



```xml
   <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```



##  二、配置连接



在application.properties 中进行配置 redis相关的设置

```properties
#配置redis
spring.redis.host=82.156.167.56
spring.redis.port=6379
spring.redis.password=123456
```



远程连接 redis 需要修改 redis.conf 配置文件

```bash
1、修改redis服务器的配置文件 redis.conf

注释以下绑定的主机地址
# bind 127.0.0.1

2、修改redis服务器的参数配置

修改redis的保护模式为no，不启用
 protected-mode "no"
 
 设置访问redis的密码[可选可不选]
 requirepass yourpassword
```



设置密码之后，在客户端登陆需要 输入密码才能登陆

```shell
auth yourpassword
```





## 三、测试！

```java
package com.rain;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.RedisTemplate;

@SpringBootTest
class DemoApplicationTests {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    @Test
    void contextLoads() {
        System.out.println(redisTemplate.opsForValue().set("k1","v1"));
    }

}
```





## 四、RedisTemplate 的使用



redisTemplate类中对应 redis的每个类型都有对应的方法

![1657361308982](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657361308982.png)

我们先选对应的类型方法之后，在进行选对应的命令，此时就和我们学到redis命令一样的操作了

![1657361388578](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657361388578.png)

除了类型的操作，一些基本的操作（选择数据库、事务等）直接通过redisTemplate 使用

![1657361511507](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657361511507.png)



一些与数据库相关的命令，必须先获取RedisConnection,在使用对应的方法，获取连接通过redistemplate的 getConnectionFactory 的 getConnection() 方法获得



![1657361621851](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657361621851.png)





## 五、默认RedisTemplate 存在的问题



但是使用默认的RedisTemplate 会存在一些问题

演示一下

使用java连接远程redis，设置了key-value, value 为中文

```java
package com.rain;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.connection.RedisConnection;
import org.springframework.data.redis.core.RedisTemplate;

@SpringBootTest
class DemoApplicationTests {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    @Test
    void contextLoads() {
        redisTemplate.opsForValue().set("name","我是张三!");
    }

}
```

我们再去服务器上看一看存储的内容

```bash
127.0.0.1:6379> get name
"\xe6\x88\x91\xe6\x98\xaf\xe5\xbc\xa0\xe4\xb8\x89!"
```

中文成了乱码！！这是因为所有的操作跨平台传输都需要序列化， 都需要经过 把一个对象状态保存成一种跨平台识别的字节格式，然后其他的平台才可以通过字节信息解析还原对象信息。 我们可以转换成json格式在发送，RedisTemplate 使用的是JDK默认的序列化工具，



![redistemplate对象内部提供的序列化](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657363987045.png)





&emsp;&emsp;同时redisTemplate 对象还提供了一些常用的序列化对象，默认是jdk序列化 ，jdk序列化就可能会让中文进行转义，为了使中文不进行转义，那么可以使用JSon格式的序列化对象进行操作，我们重新定义 Redistemplate 对象并注册

![1657364044597](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657364044597.png)





## 六、自定义 RedisTemplate



&emsp;&emsp;创建一个config包，自己定义一个配置类（@Configuration），然后拿原来的bean作为模板，进行修改一些序列化的属性。



```java
package com.rain.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.net.UnknownHostException;

@Configuration
public class RedisConfig {
    //配置我们自己的redisTemplate  固定模板
    @Bean("MyredisTemplate")
    @SuppressWarnings("all") //告诉编译器忽略全部的警告，不用在编译完成后出现警告信息
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory)
            throws UnknownHostException {

        //我们为了自己开发方便，一般直接使用<String, Object>类型
        RedisTemplate<String, Object> template = new RedisTemplate<String,Object>();
        //连接工厂
        template.setConnectionFactory(factory);

        //Json的序列化配置
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper(); //JackSon对象
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);

        //String类型的序列化配置
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();


        //Key采用String的序列化操作
        template.setKeySerializer(stringRedisSerializer);
        //Hash的key也采用String的序列化方式
        template.setHashKeySerializer(stringRedisSerializer);
        //value序列化采用jackson
        template.setValueSerializer(jackson2JsonRedisSerializer);
        //Hash的value序列化也采用jackson
        template.setHashValueSerializer(jackson2JsonRedisSerializer);

        //配置完之后将所有的properties设置进去
        template.afterPropertiesSet();
        return template;
    }
}


```



&emsp;&emsp;可以传递对象类型的key-value 了，key使用String序列化，value 使用 JSON 序列化（主要是改变了之前的序列化规则，使得传递的类型都可以序列化。）



## 七、测试



在测试中，传递一个对象类型的参数，成功运行



```java
package com.rain;

import com.rain.pojo.AdminStrator;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.RedisTemplate;

@SpringBootTest
class DemoApplicationTests {

    @Autowired
    private RedisTemplate<Object, Object> redisTemplate;

    @Test
    void contextLoads() {
        AdminStrator user = new AdminStrator(1,"张三",15);
        redisTemplate.opsForValue().set("admin",user);
    }

}
```



在服务器端打开redis客户端，必须以 --raw ，命令打开，才可支持中文

```bash
redis-cli -p 6379 --raw
```



查看 key 是否添加成功，已经对应的 value

```bash
127.0.0.1:6379> keys *
admin
127.0.0.1:6379> get admin
["com.rain.pojo.AdminStrator",{"id":1,"name":"张三","age":15}]

```



我们再通过java程序看是否能获得key的值



```java
package com.rain;

import com.rain.pojo.AdminStrator;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.RedisTemplate;

@SpringBootTest
class DemoApplicationTests {

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    @Test
    void contextLoads() {
        System.out.println(redisTemplate.opsForValue().get("admin"));;
    }

}
```



运行结果，成功拿到 redis 中 ”admin“ 对应的value信息



![1657366051025](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657366051025.png)





## 八、企业中 RedisTemplate 的使用

&emsp;&emsp;企业中因为 redisTemplate 使用的时候 有很多方法，还分了很多层，与 jedis 相比没有那么方便，所以通常会直接给RedisTemplate 套一层使用的工具类，到时候我们直接通过工具类进行调用命令，达到提高效率的目的。  



&emsp;&emsp;创建一个utils工具包，把 RedisUtil 代码模板粘贴进来，同时注意，模板中注入的时我们自定义的 RedisTemplate类,可支持对象的序列化。



```java
package com.rain.utils;


import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.TimeUnit;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;
import org.springframework.util.CollectionUtils;

import lombok.extern.slf4j.Slf4j;

/**
 *
 * @author 王赛超 基于spring和redis的redisTemplate工具类 针对所有的hash 都是以h开头的方法 针对所有的Set 都是以s开头的方法 不含通用方法 针对所有的List 都是以l开头的方法
 */
@Component
@Slf4j
public class RedisUtil {
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    // =============================common============================
    /**
     * 指定缓存失效时间
     *
     * @param key
     *            键
     * @param time
     *            时间(秒)
     * @return
     */
    public boolean expire(String key, long time) {
        try {
            if (time > 0) {
                redisTemplate.expire(key, time, TimeUnit.SECONDS);
            }
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 根据key 获取过期时间
     *
     * @param key
     *            键 不能为null
     * @return 时间(秒) 返回0代表为永久有效
     */
    public long getExpire(String key) {
        return redisTemplate.getExpire(key, TimeUnit.SECONDS);
    }

    /**
     * 判断key是否存在
     *
     * @param key
     *            键
     * @return true 存在 false不存在
     */
    public boolean hasKey(String key) {
        try {
            return redisTemplate.hasKey(key);
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 删除缓存
     *
     * @param key
     *            可以传一个值 或多个
     */
    @SuppressWarnings("unchecked")
    public void del(String... key) {
        if (key != null && key.length > 0) {
            if (key.length == 1) {
                redisTemplate.delete(key[0]);
            } else {
                redisTemplate.delete(CollectionUtils.arrayToList(key));
            }
        }
    }

    // ============================String=============================
    /**
     * 普通缓存获取
     *
     * @param key
     *            键
     * @return 值
     */
    public Object get(String key) {
        return key == null ? null : redisTemplate.opsForValue().get(key);
    }

    /**
     * 普通缓存放入
     *
     * @param key
     *            键
     * @param value
     *            值
     * @return true成功 false失败
     */
    public boolean set(String key, Object value) {
        try {
            redisTemplate.opsForValue().set(key, value);
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }

    }

    /**
     * 普通缓存放入并设置时间
     *
     * @param key
     *            键
     * @param value
     *            值
     * @param time
     *            时间(秒) time要大于0 如果time小于等于0 将设置无限期
     * @return true成功 false 失败
     */
    public boolean set(String key, Object value, long time) {
        try {
            if (time > 0) {
                redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
            } else {
                set(key, value);
            }
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 递增 适用场景： https://blog.csdn.net/y_y_y_k_k_k_k/article/details/79218254 高并发生成订单号，秒杀类的业务逻辑等。。
     *
     * @param key
     *            键
     * @param by
     *            要增加几(大于0)
     * @return
     */
    public long incr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递增因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, delta);
    }

    /**
     * 递减
     *
     * @param key
     *            键
     * @param delta
     *            要减少几(小于0)
     */
    public long decr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递减因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, -delta);
    }

    // ================================Map=================================
    /**
     * HashGet
     *
     * @param key
     *            键 不能为null
     * @param item
     *            项 不能为null
     * @return 值
     */
    public Object hget(String key, String item) {
        return redisTemplate.opsForHash().get(key, item);
    }

    /**
     * 获取hashKey对应的所有键值
     *
     * @param key
     *            键
     * @return 对应的多个键值
     */
    public Map<Object, Object> hmget(String key) {
        return redisTemplate.opsForHash().entries(key);
    }

    /**
     * HashSet
     *
     * @param key
     *            键
     * @param map
     *            对应多个键值
     * @return true 成功 false 失败
     */
    public boolean hmset(String key, Map<String, Object> map) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * HashSet 并设置时间
     *
     * @param key
     *            键
     * @param map
     *            对应多个键值
     * @param time
     *            时间(秒)
     * @return true成功 false失败
     */
    public boolean hmset(String key, Map<String, Object> map, long time) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key
     *            键
     * @param item
     *            项
     * @param value
     *            值
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key
     *            键
     * @param item
     *            项
     * @param value
     *            值
     * @param time
     *            时间(秒) 注意:如果已存在的hash表有时间,这里将会替换原有的时间
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value, long time) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 删除hash表中的值
     *
     * @param key
     *            键 不能为null
     * @param item
     *            项 可以使多个 不能为null
     */
    public void hdel(String key, Object... item) {
        redisTemplate.opsForHash().delete(key, item);
    }

    /**
     * 判断hash表中是否有该项的值
     *
     * @param key
     *            键 不能为null
     * @param item
     *            项 不能为null
     * @return true 存在 false不存在
     */
    public boolean hHasKey(String key, String item) {
        return redisTemplate.opsForHash().hasKey(key, item);
    }

    /**
     * hash递增 如果不存在,就会创建一个 并把新增后的值返回
     *
     * @param key
     *            键
     * @param item
     *            项
     * @param by
     *            要增加几(大于0)
     * @return
     */
    public double hincr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, by);
    }

    /**
     * hash递减
     *
     * @param key
     *            键
     * @param item
     *            项
     * @param by
     *            要减少记(小于0)
     * @return
     */
    public double hdecr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, -by);
    }

    // ============================set=============================
    /**
     * 根据key获取Set中的所有值
     *
     * @param key
     *            键
     * @return
     */
    public Set<Object> sGet(String key) {
        try {
            return redisTemplate.opsForSet().members(key);
        } catch (Exception e) {
            log.error(key, e);
            return null;
        }
    }

    /**
     * 根据value从一个set中查询,是否存在
     *
     * @param key
     *            键
     * @param value
     *            值
     * @return true 存在 false不存在
     */
    public boolean sHasKey(String key, Object value) {
        try {
            return redisTemplate.opsForSet().isMember(key, value);
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 将数据放入set缓存
     *
     * @param key
     *            键
     * @param values
     *            值 可以是多个
     * @return 成功个数
     */
    public long sSet(String key, Object... values) {
        try {
            return redisTemplate.opsForSet().add(key, values);
        } catch (Exception e) {
            log.error(key, e);
            return 0;
        }
    }

    /**
     * 将set数据放入缓存
     *
     * @param key
     *            键
     * @param time
     *            时间(秒)
     * @param values
     *            值 可以是多个
     * @return 成功个数
     */
    public long sSetAndTime(String key, long time, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().add(key, values);
            if (time > 0)
                expire(key, time);
            return count;
        } catch (Exception e) {
            log.error(key, e);
            return 0;
        }
    }

    /**
     * 获取set缓存的长度
     *
     * @param key
     *            键
     * @return
     */
    public long sGetSetSize(String key) {
        try {
            return redisTemplate.opsForSet().size(key);
        } catch (Exception e) {
            log.error(key, e);
            return 0;
        }
    }

    /**
     * 移除值为value的
     *
     * @param key
     *            键
     * @param values
     *            值 可以是多个
     * @return 移除的个数
     */
    public long setRemove(String key, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().remove(key, values);
            return count;
        } catch (Exception e) {
            log.error(key, e);
            return 0;
        }
    }

    // ============================zset=============================
    /**
     * 根据key获取Set中的所有值
     *
     * @param key
     *            键
     * @return
     */
    public Set<Object> zSGet(String key) {
        try {
            return redisTemplate.opsForSet().members(key);
        } catch (Exception e) {
            log.error(key, e);
            return null;
        }
    }

    /**
     * 根据value从一个set中查询,是否存在
     *
     * @param key
     *            键
     * @param value
     *            值
     * @return true 存在 false不存在
     */
    public boolean zSHasKey(String key, Object value) {
        try {
            return redisTemplate.opsForSet().isMember(key, value);
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    public Boolean zSSet(String key, Object value, double score) {
        try {
            return redisTemplate.opsForZSet().add(key, value, 2);
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 将set数据放入缓存
     *
     * @param key
     *            键
     * @param time
     *            时间(秒)
     * @param values
     *            值 可以是多个
     * @return 成功个数
     */
    public long zSSetAndTime(String key, long time, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().add(key, values);
            if (time > 0)
                expire(key, time);
            return count;
        } catch (Exception e) {
            log.error(key, e);
            return 0;
        }
    }

    /**
     * 获取set缓存的长度
     *
     * @param key
     *            键
     * @return
     */
    public long zSGetSetSize(String key) {
        try {
            return redisTemplate.opsForSet().size(key);
        } catch (Exception e) {
            log.error(key, e);
            return 0;
        }
    }

    /**
     * 移除值为value的
     *
     * @param key
     *            键
     * @param values
     *            值 可以是多个
     * @return 移除的个数
     */
    public long zSetRemove(String key, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().remove(key, values);
            return count;
        } catch (Exception e) {
            log.error(key, e);
            return 0;
        }
    }
    // ===============================list=================================

    /**
     * 获取list缓存的内容
     *
     * @取出来的元素 总数 end-start+1
     *
     * @param key
     *            键
     * @param start
     *            开始 0 是第一个元素
     * @param end
     *            结束 -1代表所有值
     * @return
     */
    public List<Object> lGet(String key, long start, long end) {
        try {
            return redisTemplate.opsForList().range(key, start, end);
        } catch (Exception e) {
            log.error(key, e);
            return null;
        }
    }

    /**
     * 获取list缓存的长度
     *
     * @param key
     *            键
     * @return
     */
    public long lGetListSize(String key) {
        try {
            return redisTemplate.opsForList().size(key);
        } catch (Exception e) {
            log.error(key, e);
            return 0;
        }
    }

    /**
     * 通过索引 获取list中的值
     *
     * @param key
     *            键
     * @param index
     *            索引 index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
     * @return
     */
    public Object lGetIndex(String key, long index) {
        try {
            return redisTemplate.opsForList().index(key, index);
        } catch (Exception e) {
            log.error(key, e);
            return null;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key
     *            键
     * @param value
     *            值
     * @return
     */
    public boolean lSet(String key, Object value) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key
     *            键
     * @param value
     *            值
     * @param time
     *            时间(秒)
     * @return
     */
    public boolean lSet(String key, Object value, long time) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key
     *            键
     * @param value
     *            值
     * @return
     */
    public boolean lSet(String key, List<Object> value) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key
     *            键
     * @param value
     *            值
     * @param time
     *            时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value, long time) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 根据索引修改list中的某条数据
     *
     * @param key
     *            键
     * @param index
     *            索引
     * @param value
     *            值
     * @return
     */
    public boolean lUpdateIndex(String key, long index, Object value) {
        try {
            redisTemplate.opsForList().set(key, index, value);
            return true;
        } catch (Exception e) {
            log.error(key, e);
            return false;
        }
    }

    /**
     * 移除N个值为value
     *
     * @param key
     *            键
     * @param count
     *            移除多少个
     * @param value
     *            值
     * @return 移除的个数
     */
    public long lRemove(String key, long count, Object value) {
        try {
            Long remove = redisTemplate.opsForList().remove(key, count, value);
            return remove;
        } catch (Exception e) {
            log.error(key, e);
            return 0;
        }
    }

}


```



&emsp;&emsp;模板写好后，我们再使用的时候，就不需要注入 RedisTemplate，需要注入 RedisUtil,使用的时候直接调用相关的命令即可。



![1657366519433](C:\Users\rain7\AppData\Roaming\Typora\typora-user-images\1657366519433.png)