# IDEA

## IDEA四大分级:

- **项目(Project)**:项目名称作为文件夹名称

- **模块(Module)**:模块名称显示在左边

- **软件包(Package)**:存放类的文件夹
- **类(Class)**:单个代码文件

## 分级示例:

想要做一个电商网站项目;包括前端网页展示,后端数据返回,安全验证等模块;后端服务器模块里面有mysql,redis,restful api等软件包

**一般都是先新建一个空白项目,然后向里面添加不同框架的模块**

## IDEA企业版

- IDEA企业版创建模块的时候才会有SpringBoot的初始化工具
- 企业版破解连接:[点击这里打开网站](https://www.exception.site/essay/idea-reset-eval)

## 设置@Test测试方法

- 修改maven配置添加dependency

  ```xml
  <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-api</artifactId>
      <version>5.9.3</version>
  </dependency>
  ```
  
- ~~(添加JUnit插件)~~

- 在测试方法上面写@Test,之后导入包:

  ```java
  import org.junit.jupiter.api.Test;
  ```

## 常用技巧

导入包:

- **ALT+ENTER:自动import-package**
- 输入包的时候会有提示,按照提示去走,就会import包

查看参数类型:

- 输入方法的时候观察提示
- 鼠标放在方法参数的括号上面,会显示有哪些参数分别是什么类型

省略自己写实体类的toString(),getter和setter方法以及构造方法:

```java
//需要引入lombok
@Data//省略getter和setter
@NoArgsConstructor//无参构造器
@AllArgsConstructor//有参构造器
public class User {
    private String name;
    private Integer age;
}
```

创建常量:final==>JAVA中没有const关键字

- ctrl+h:查看子类
- alt+7:查看方法

# SpringBoot项目搭建



## 创建SpringBoot项目

- group模块所在的文件夹
- artifact是模块名称

- group和artifact必须要唯一
- 选择maven作为包管理工具
- package会作为项目整体运行文件的路径

- SpringBoot需要联网运行

![屏幕截图 2023-09-01 235742](https://sciform.one/pictures/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202023-09-01%20235742.png)

## JAVA目录问题

- 在运行文件同目录或者子目录下面的JAVA类才会被加载

- /src/main存放主要业务代码,/src/test当中存放测试代码

- Settings>Editor>FileTypes当中设置ignoredFilesAndFolders可以屏蔽显示文件

- **/src/main/resources/application.properties可以设置项目的属性(有提示)**

  ```
  server.port=80
  ```
  
## MAVEN依赖项飘红

- 原因:网络连接不顺畅
- 解决:
  1. 添加http_proxy,https_proxy
  2. 删除本地仓库中原来的残留:C:\Users\nightmare00\\.m2\repository\\{指定目录}
  3. 刷新MAVEN

# Spring&nbsp; MVC



## REST &nbsp;API

1. 同一类操作用一个url(复数)来控制:/users
2. 不会明显地展示出来操作的目的:/users/add==>/users
3. 用不同的方法来区分不同的操作:GET/POST/DELETE

## 路由基础

- **@Controller**:标识一个类是 Spring MVC 框架中的路由器(接受请求返回数据)
- **@RequestMapping(value="/api",method=RequestMethod.GET)**:表明网页url,可以上下层累加
- **@ResponseBody**:表明return的String值应该返回给网页/客户端显示(res.send)

```java
@Controller
public class Users {
    @RequestMapping(value = "/users",method = RequestMethod.GET)
    @ResponseBody
    public String getAllUsers(Integer id){
        return "get user "+id;
    }
    @RequestMapping(value = "/users/{id}",method = RequestMethod.GET)
    @ResponseBody
    public String getUserById(@PathVariable Integer id){
        return "get user "+id;
    }
    @RequestMapping(value = "/users",method = RequestMethod.POST)
    @ResponseBody
    public String userRegister(@RequestBody String json){
        return json;
    }
}
```

## 处理输入参数

- GET参数:

  - 类型1:/users?id=2021091203003:

    ```java
    @RequestMapping(value="/users",method=RequestMethod.GET)
    public String getUserById(Integer id)
    ```

  - 类型2:/users/{id}:通过处理函数的参数直接获取

    ```java
    @RequestMapping(value="/users/{id}",method=RequestMethod.GET)
    public String getUserById(@PathVariable Integer id)
    ```
  
- POST参数:

  - 请求体参数:

  ```java
  @RequestMapping(value="/users",method=RequestMethod.POST)
  public String addUser(String userInfo)//得到用户信息的JSON对象
  ```
## 输入信息

1.  **@PathVariable**：用于将 URL 中的**路径参数(/users/653)**绑定到方法参数上。例如，@PathVariable("id") 将会将 URL 中的 "id" 部分的值绑定到方法参数上。
2. **@RequestParam**：用于将查询**字符串参数(?id=653)**绑定到方法参数上。例如，@RequestParam("name")将会将查询字符串参数中名为 "name" 的值绑定到方法参数上。
3. **@RequestHeader**：用于将**请求头信息**绑定到方法参数上。例如，@RequestHeader("User-Agent") 将会将请求头中的 "User-Agent" 值绑定到方法参数上。
4. **@CookieValue**：用于将**请求中的 Cookie 值**绑定到方法参数上。例如，@CookieValue("sessionId") 将会将名为 "sessionId" 的 Cookie 值绑定到方法参数上。
5. @RequestAttribute：用于将请求属性绑定到方法参数上。请求属性是在请求的生命周期内存储和传递的对象。例如，@RequestAttribute("userId")将会将名为 "userId" 的请求属性值绑定到方法参数上。

6. **@RequestBody**:用于**从请求体中获取数据**。例如，@RequestBody将会将请求体中的数据绑定到方法参数上。(不支持指定参数)

## 简化路由

- **@RestController=@Controller+@ResponseBody**
- **@GetMapping=RequestMapping(method=RequestMethod.GET)**
- **可以设置多级路由减少重复**

```java
@RestController
@RequestMapping("/users")
public class Users{
    @GetMapping
    public String getUserById(@RequestParam("id") Integer id){
        return "get user "+id;
    }
    @GetMapping(value="/{id}/{name}")
    public String getUserById(@PathVariable("name") String name,@PathVariable("id") Integer id){
        return "get user "+name+" "+id;
    }
    @PostMapping
    public String createUser(@RequestBody String userInfo){
        return userInfo;
    }
}
```
# SpringBoot优势

## Parent

- SpringBoot的pom.xml中的parent中**定义了包版本标准(properties)中**,然后**依赖项中使用了事先定义好的版本**,让不同的包之间版本不会发生冲突而且同一个包使用同一个版本,极大程度上方便包管理
- SpringBoot中不需要自己指定包的版本,而是自动使用parent中定义好的坐标版本
- SpringBoot的版本不同,依赖坐标版本就会有差别!(2.5.4的包版本和2.5.3的包版本就会不一样)

## Starter

- **依赖传递**:依赖一个模块,这个模块又会依赖很多配置好的子模块:不用再自己一个一个写依赖了

- starter定义了当前项目使用的所有依赖坐标,以达到减少依赖配置的目的

## 实际开发中:

- 只写groupId和artifactId
- 如果版本出错或者未定义,再写version