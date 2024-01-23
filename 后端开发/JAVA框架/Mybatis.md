# MyBatis简介

## 什么是MyBatis

Mybatis是一个持久层框架,用于简化JDBC(JAVA访问数据库的标准接口)开发

- 持久层:JAVAEE中分为三层(表现层展示精美页面,业务层处理业务逻辑,持久层持久化存储数据)
- 框架:用来简化开发

Mybatis优势:

- 去除硬编码:把配置信息(比如数据库的IP,用户名,密码)写在配置文件中,不是直接写在文件中
- 简化操作:免除JDBC繁琐代码

## MyBatis快速上手

- 参考网址:[MyBatis入门](https://mybatis.org/mybatis-3/zh/getting-started.html)

1. 导入MyBatis依赖:

   ```xml
   <!--mybatis-->
   <dependency>
       <groupId>org.mybatis.spring.boot</groupId>
       <artifactId>mybatis-spring-boot-starter</artifactId>
       <version>3.0.2</version>
   </dependency>
   <!--mysql连接工具-->
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <version>8.0.33</version>
   </dependency>
   <!--junit单元测试-->
   <dependency>
       <groupId>org.junit.jupiter</groupId>
       <artifactId>junit-jupiter-api</artifactId>
   </dependency>
   <!--logback-->
   <dependency>
       <groupId>ch.qos.logback</groupId>
       <artifactId>logback-core</artifactId>
   </dependency>
   ```

   

2. 编写MyBatis核心**配置文件**-->替换连接信息解决硬编码问题

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "https://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
       <environments default="development">
           <environment id="development">
               <transactionManager type="JDBC"/>
               <dataSource type="POOLED">
                   <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                   <!--使用jdbc进行数据库连接,连接mysql数据库-->
                   <!--ip地址和端口号是107.173.181.210,数据库是mybatistest-->
                   <property name="url" value="jdbc:mysql://107.173.181.210:3306/mybatistest"/>
                   <property name="username" value="admin"/>
                   <property name="password" value="6d2ySW2py"/>
               </dataSource>
           </environment>
       </environments>
       <mappers>
           <mapper resource="UserMapper.xml"/>
       </mappers>
   </configuration>
   ```

3. 编写Sql映射文件

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <!--namespace命名空间-->
   <!--id:用来标识区分statement,resultType:java实体类名称(需要有get和set方法)-->
   <mapper namespace="test">
       <select id="selectAllUsers" resultType="mybatis.demo.data.User">
           select * from tb_user;
       </select>
   </mapper>
   ```

4. 使用sqlSession执行sql语句

   ```java
   @Test
   public void testMyBatis() throws IOException {
       //1.加载mybatis的核心配置文件,获取SqlSessionFactory
       InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
       SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
       //2. 获取SqlSession对象,用它来执行sql
       SqlSession sqlsession = sqlSessionFactory.openSession();
       //3. 执行sql
       List<User> users = sqlsession.selectList("test.selectAllUsers");
       System.out.println(users);
       //4.释放资源
       sqlsession.close();
   }
   ```

# Mapper代理开发

## Mapper优势

- 简化操作:直接调用接口就行,不用调用sql语句
- 方便修改:直接修改接口和xml文件

## Mapper步骤

1. 定义与SQL映射文件**同名的Mapper接口**,并且将Mapper接口和SQL映射文件放置在**同一目录(resource文件夹下与编译后的class文件同目录)**下

   ```java
   src->main->java->com->demo->mapper->UserMapper(接口)
   resources->com->demo->mapper->UserMapper.xml
   ```

2. 设置与SQL映射文件的namespace属性为Mapper接口全名(包名.类名)

   ```xml
   <mapper namespace="com.demo.mapper.UserMapper">
       <select id="selectAllUsers" resultType="com.demo.data.User">
           select * from tb_user;
       </select>
   </mapper>
   ```

3. 在Mapper接口中定义方法,方法名就是SQL映射文件中sql语句的id,并保持参数类型和返回值类型一致

   ```java
   public interface UserMapper {
       List<User> selectAllUsers();
   }
   ```

4. 修改mybatis-config配置文件中mapper路径

   ```xml
   <mappers>
       <!--加载sql映射文件-->
   <!--<mapper resource="mybatis/demo/mapper/UserMapper.xml"/>-->
       <!--Mapper代理方式-->
       <package name="mybatis.demo.mapper"/>
   </mappers>
   ```

5. 直接调用接口

   ```java
   //sqlsession.getMapper(UserMapper.class)获得接口
   UserMapper usermapper = sqlsession.getMapper(UserMapper.class);
   List<User> users = usermapper.selectAllUsers();
   System.out.println(users);
   ```

- 总结:**路径和类名一致,方法名和id一致,配置mapper路径**


## 设置

