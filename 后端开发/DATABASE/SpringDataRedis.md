# 新手上路

## 搭建环境

1. 依赖项:lombok和nosql中的SpringDataRedis

2. 引入连接池(默认lettuce,也可以jedis)和测试模块

   ```java
   <dependency>
       <groupId>org.apache.commons</groupId>
       <artifactId>commons-pool2</artifactId>
   </dependency>
   <dependency>
           <groupId>org.junit.jupiter</groupId>
           <artifactId>junit-jupiter-api</artifactId>
   </dependency>
   ```

3. 建立配置文件application.yaml

   ```yaml
   spring:
     data:
       redis:
         host: 107.173.181.210
         port: 6379
         password: 6d2ySW2py
         lettuce:
           pool:
             max-active: 8
             max-idle: 8
             min-idle: 0
             max-wait: -1ms
   ```

4. 连接测试:

   ```java
   @SpringBootTest
   public class RedisTest {
       @Autowired
       private RedisTemplate redisTemplate;
       @Test
       void testString(){
           //插入一条String类型的数据,可以是任何Object都行因为会序列化
           redisTemplate.opsForValue().set("name","王八蛋");
       }
   }
   ```
   
## 常用方法

- delete(key):删除key

- `opsForValue()`：操作 Redis 字符串类型。
- `opsForHash()`：操作 Redis 哈希（Hash）类型。
- `opsForList()`：操作 Redis 列表（List）类型。
- `opsForSet()`：操作 Redis 集合（Set）类型。
- `opsForZSet()`：操作 Redis 有序集合（Sorted Set）类型。
- `opsForGeo()`：操作 Redis 地理位置（Geo）类型。
- `opsForHyperLogLog()`：操作 Redis HyperLogLog 类型。
- `opsForStream()`：操作 Redis Stream 类型。


## 序列化和反序列化

- RedisTemplate在存入Redis数据库的时候会进行序列化和反序列化(**用字节码存储**):
  - 好处:能够**存储所有数据类型**,只要是Object就能进行存储
  - 坏处:存入Redis数据库中的内容是**"乱码"**(可读性差),而且**内存占用大**

- RedisTemplate默认使用jdk序列化器,需要自己指定其他序列化器

  - **Key使用String序列化器:RedisSerializer.string()**

  - **Value使用json序列化器:GenericJackson2JsonRedisSerializer()**

  - 配置序列化器:

    ```java
    @Configuration
    public class RedisConfig {
        @Bean
        public RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory connectionFactory){
            //创建RedisTemplate对象
            RedisTemplate<String,Object> template = new RedisTemplate<>();
            //设置连接工厂
            template.setConnectionFactory(connectionFactory);
            //创建JSON序列化工具
            GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
            //设置Key序列化
            template.setKeySerializer(RedisSerializer.string());
            template.setHashKeySerializer(RedisSerializer.string());
            //设置Value序列化
            template.setValueSerializer(jsonRedisSerializer);
            template.setHashValueSerializer(jsonRedisSerializer);
            //返回
            return template;
        }
    }
    ```
    
  - json序列化器缺点:class信息会占用很大的空间,内存利用效率低
    
    ```json
    "{
    	"@class":"redis.demo.data.User",
    	"name":"Alice",
    	"age":13
    }"//可以看到class信息占用的空间比其他信息加起来还多
    ```
  
- StringRedisTemplate

  - 为了节省内存空间,一般使用String类型的序列化器,要求只存储String类型的Key和Value
  - 当我们需要存储Java对象的时候需要**手动完成对象的序列化和反序列化**
  
  ```java
  @SpringBootTest
  public class StringRedisTest {
      @Autowired
      private StringRedisTemplate redisTemplate;
      private static final ObjectMapper mapper = new ObjectMapper();
      @Test
      void testString() throws JsonProcessingException {
          //准备对象
          User user = new User("Alice",13);
          //手动序列化
          String json = mapper.writeValueAsString(user);
          //写入一条数据到redis
          redisTemplate.opsForValue().set("user:310",json);
  
          //读取数据
          String val = redisTemplate.opsForValue().get("user:310");
          //反序列化
          User res = mapper.readValue(val,User.class);
          System.out.println(res);
      }
  }
  ```
  
## Hash类型操作

- put()