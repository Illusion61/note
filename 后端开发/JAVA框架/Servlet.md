# Tomcat

## 环境配置

1. 去[Apache的tomcat官网](https://tomcat.apache.org/)下载tomcat安装包并解压安装

2. tomcat目录结构:

   ```
   ./bin:存放运行时候的二进制文件:start.bat和shutdown.bat分别是开启tomcat和关闭tomcat(提前配置好JAVA_HOME)
   ./conf:配置文件,注意server.xml(配置服务器)和web.xml(配置网页文件)
   ./lib:存放jar包
   ./logs:存放日志信息
   ./webapps:存放web应用程序的页面
   ```

3. 运行start.bat并访问127.0.0.1:8080测试tomcat是否成功开启

4. 修改端口号和协议

   ```xml
   <Connector port="8080" protocol="HTTP/1.1"
                  connectionTimeout="20000"
                  redirectPort="8443"
                  maxParameterCount="1000"
                  />
   ```

5. 解决乱码问题:在logging.properties中配置

   ```properties
   java.util.logging.ConsoleHandler.encoding = GBK
   ```

## IDEA配置tomcat

- Settings中找到Application-Servers选项添加tomcat
- 在Run/Debug配置中添加tomcat-server
- deployment中添加目标模块/项目,进行部署(**可以设置虚拟目录**)

## IDEA中手动配置环境(不用创建JavaEE)

- 复制webapp文件夹
- **在project-structure中的artifacts中手动添加web-application**

## 配置IO多路复用(NIO)

```xml
<Connector port="8080" protocol="org.apache.coyote.http11.Http11NioProtocol" 
         connectionTimeout="20000" redirectPort="8443"/>
```



# Servlet简介

## Servlet是什么

- servlet = server+applet(运行在服务器端的小程序)
- tomcat是一个相对独立的运行环境,作为servlet的容器来托管和运行servlet(一个tomcat中可以有很多个servlet)
- servlet就是一个类,定义了**java类被tomcat识别的规则(被浏览器访问的规则)**
- web资源:静态资源(HTML+CSS),动态资源(nodejs,servlet...)

## Servlet快速入门

1. 创建Jakarta EE项目(选择web)

2. 定义一个类,实现servlet接口

3. 实现接口中的抽象方法

4. 配置servlet(web.xml)

   ```xml
   <servlet>
        <servlet-name>demo1</servlet-name>
        <servlet-class>demo.servlet.ServletDemo1</servlet-class>
   </servlet>
   <servlet-mapping>
       <servlet-name>demo1</servlet-name>
       <url-pattern>/demo1</url-pattern>
       <url-pattern>/test1</url-pattern>
   </servlet-mapping>
   ```

## Servlet的方法

```java
init方法:servlet被创建之后执行,只会执行一次
service方法:servlet每次被调用的时候都会执行,执行多次
destroy方法:servlet正常终止(服务器正常关闭)的时候执行,只会执行一次
ServletConfig方法:获取ServletConfig对象
```
## Servlet的生命周期

- 被创建:

  - 默认**第一次被访问**的时候启动:load-on-startup值为负数
  - 服务器启动的时候创建:load-on-startup的值为0或正数

  ```xml
  <load-on-startup>0</load-on-startup>
  ```

  - 注意:**一个servlet在内存中只存在一个对象**,servlet是单例的.多个用户同时访问的时候(多线程),会存在线程安全问题
  - 解决:尽量不要在Servlet中创建成员变量,转而使用局部变量(在service方法内部定义变量).即使定义了成员变量,也不要修改它的值

- 提供服务:每次servlet被调用的时候都会执行service
- 被销毁:destroy在servlet被销毁之前执行,一般用于释放资源

## Servlet3.0注解配置

注解配置:不需要web.xml

```java
@WebServlet(value={"/demo2/*","/demoo2","/demo2"},loadOnStartup = 0)
public class ServletDemo1 implements Servlet{...}
```

## Servlet体系结构

- servlet接口:实现servlet接口并重写五种方法
- GenericServlet抽象类:GenericServlet抽象类**提供了生命周期方法init和destroy的简单实现**,要编写一般的servlet只需重写抽象方法service即可
- HttpServlet:判断浏览器请求的方法,然后调用不同的方法(doGet/doPost...)
  - HttpServlet类中**默认返回"方法不支持"的页面**,需要自己重写覆盖HttpServlet中的方法
  - 只需要重写doXxx方法(doGet/doPost...)

# Servlet HTTP

## Get和Post对比

- Get请求特点:
  - 参数在url之后,形如/?key1=value1&key2=value2&key3=value3
  - 参数用明文传递,不安全
  - 参数数据量小,传输速度快
- Post请求特点:
  - 参数在请求体中
  - 参数加密传输,比较安全
  - 参数数据量小,传输速度快

## Request接收数据

```java
String username = req.getParameter("username");//GET和POST都行
```

## Response返回数据

```java
resp.setCharacterEncoding("utf-8");//设置服务器端的编码格式
resp.setContentType("text/html;charset=utf-8");//设置返回的内容类型
resp.setHeader("Content-Type","text/html;charset=utf-8");//设置响应头信息
```

- 内容类型补充:[菜鸟教程](https://www.runoob.com/http/http-content-type.html),常见内容类型:

  ```
  text/html ： HTML格式
  text/plain ：纯文本格式
  application/json： JSON数据格式
  ```

## 目录结构

```
entity:实体类,存放实体类(private对象和public的getter和setter以及toString和constructor)
utils:工具类,存放诸如创建连接池,手机号正则表达式之类的类
dao:DataAccessobjects数据存取对象,定义了一套方法供CRUD数据库中的数据使用
service:存放后端逻辑代码
```

