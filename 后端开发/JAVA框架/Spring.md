# Spring核心概念

## IOC控制反转

- 控制反转(Inversion Of Control ):使用对象时,由主动new产生对象转换为由**外部提供对象**,此过程中对象创建控制权由程序转移到外部,此思想称为控制反转
- **IOC容器**负责对象的**创建,初始化**等一系列工作,被创建或被管理在IOC容器中的对象称为**Bean**
- IOC的实现:

1. 使用xml文件配置IOC容器:resource目录下新建一个Spring Config文件

```xml
id是对象在这个容器中的名称,
name是别名(可以用逗号/分号/空格分开),
class是类的全名(注意必须是实现类,不能是接口或抽象类!)
scope:单实例对象(默认)/多实例对象(每次getBean都会新建一个实例)
<bean id="net" name="network netframe" class="com.example.Network" scope="singleton"/>
```

2. 获取IOC容器,之后获取Bean

```java
@Test
public void testIOC(){
    //获取IOC容器
    ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
    //获取Bean
    Network net = (Network)ctx.getBean("net");
}
```

## DI依赖注入

- 在容器中建立bean与bean之间的**依赖关系**的整个过程,称为依赖注入
- 原来是自己new依赖对象,现在是**直接让IOC注入进来**

- DI实现:

1. 创建set依赖对象的方法:

```java
private Network network;
public void setNetwork(Network network){
    this.network = network;
}
```

2. 修改properties中的依赖

```xml
property:配置属性
name:属性名称(set方法后面的名称)
ref:具体依赖哪一个其他Bean
<bean id="web" class="frame.implementation.Website">
    <property name="network" ref="net"/>
</bean>
```

## 推荐适用范围

- 适合交给容器管理的bean:工具对象(可复用)
- 不适合交给容器管理的bean:实体类对象(有状态,用来存储数据)

# IOC相关内容

## Bean实例化

- 使用构造方法:底层原理是反射(可以访问私有成员变量和方法),需要提供默认无参构造方法

- 构造方法:

## Bean的生命周期

- 配置init-method和destroy-metohd钩子函数

  ```xml
  <bean id="net" name="network netframe" class="frame.implementation.Network" scope="singleton" init-method="init" destroy-method="destroy"/>
  <bean id="web" class="frame.implementation.Website" init-method="init" destroy-method="destroy">
      <property name="network" ref="net"/>
  </bean>
  ```

# DI相关内容

## Setter注入

首先必须在原始类的定义中提供setter方法!

- 引用类型:**ref属性初始化**(引用其他bean)

  ```xml
  <bean id="web" class="frame.implementation.Website" init-method="init" destroy-method="destroy">
      <property name="network" ref="net"/>
      <property name="server" ref="serv"/>
  </bean>
  ```

- 基本类型:**value属性初始化**(value值自己提供)

  ```xml
  <bean id="user:309" class="frame.implementation.User">
      <property name="userid" value="309"/>
      <property name="username" value="Jerry"/>
  </bean>
  ```

## 构造器注入

首先必须自己提供构造方法

其次把setter注入中的property换成constructor-arg,其他不变

不推荐,耦合度比较高

## 自动注入

- 什么叫自动注入:IOC容器根据**bean所依赖的资源**在容器中**自动查找**并**注入**到bean中的过程
- 注意事项:
  - 自动注入设置前必须要提供setter方法
  - 按照类型自动注入必须只能有唯一一个bean,否则不知道选择哪个
  - 自动注入优先级低于手动注入(setter注入和构造器注入)
- 提供autowire属性,说明自动注入的类型:
  - 按照Type(bean的类型)
  - 按照Name:变量根据变量名自动在容器中找同名id的bean注入

```xml
<bean id="web" class="frame.implementation.Website" autowire="byType">
    <!--        <property name="network" ref="net"/>-->
    <!--        <property name="server" ref="serv"/>-->
</bean>
```

## 集合注入

如何在bean中注入集合类型的对象?

- 数组:array;列表:list;集合:set;哈希表:map;Properties:props

```xml
<bean id="userInfo" class="com.demo.User">
    <property name="friendsArr">
        <array>
            <value>19231</value>
            <value>57231</value>
            <value>12930</value>
        </array>
    </property>
    <property name="blackSet">
        <set>
            <value>12413</value>
            <value>54612</value>
            <value>71309</value>
        </set>
    </property>
</bean>
```

## 第三方bean创建

1. 在maven中创建依赖项
2. 找到对应的包名(全名),填入class属性中
3. 查看类的源码:
   1. 看一看都有哪些关键的配置参数
   2. 注意是使用构造方法初始化还是使用setter方法,使用正确的方法注入

## 加载properties文件





# 注解开发

## 半注解开发(注解+配置文件)

1. 添加配置文件:

   ```java
   @Component("bookDao")//bean的id
   public class BookDaoImpl implements BookDao {...}
   ```
   
2. 配置文件扫描:

   ```java
   <context:component-scan base-package="com.demo"/>
   ```

## 纯注解开发

使用配置类来代替配置文件

```java
@Configuration
@ComponentScan({"com.demo","com.redis"...})
public class SpringConfig(){}
```
## 注解集合

`<bean id="" class="" scope="" init-method="" destroy-method="">`

```java
//id,class
@Component("")
@Controller("")//控制类(处理用户请求)
@Service("")//服务类(业务核心逻辑)
@Repository("")//持久层(与数据库交互)
//scope
@Scope("")
//init-method
@PostConstruct
//destroy-method
@PreDestroy
```
`<context:component-scan base-package="com.demo"/>`

```java
@Configuration//标明是配置类
@ComponentScan({"...","..."})//说明扫描的包
public class SpringConfig
```

`<property ref=""></property>`

```java
@Autowired//按照类型注入
@Qualifier("name")//按照名称注入
@Value(xxx)//简单类型注入
```

`<context:property-placeholder location="jdbc.properties"/>`

```java
@Configuration
@ConponentScan({"..."})
@PropertySource({"xxx","yyy"})//在配置类上添加
public class SpringConfig(){...}
```

## 注解开发管理第三方Bean

#### 方法一:使用包扫描引入

在SpringConfig上添加包扫描

```java
@Configuration
@ComponentScan("com.config")
public class SpringConfig(){}
```

在**第三方配置类**上添加配置注解:

@Configuration:说明这是一个配置类

@Bean:代表返回的对象默认被配置为Bean?

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

#### 方法二:使用@Import导入(推荐)

在SpringConfig上添加

