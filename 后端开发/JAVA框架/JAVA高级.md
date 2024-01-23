# 泛型

## 泛型产生的背景

默认类型为Object,所有类型的数据都能往里塞,**运行的时候会导致错误**

```java
ArrayList list = new ArrayList();
list.add("hello");list.add(1024);
System.out.println((String)list.get(0));
System.out.println((String)list.get(1));//运行时报错
```

泛型的作用:

1. 提供**编译期间的类型检查**,让潜在的错误提前被检测出来
2. 减少**数据类型的转换**

```java
ArrayList<String> strlist = new ArrayList<>();
strlist.add("hello");
strlist.add(1024);//编译时报错
```

## 泛型类的使用

常用的泛型标识:T,E,K,V

```java
class 类名称<泛型标识1,泛型标识2...>{
    private 泛型标识 变量名;
}
```

构造时以下三种方式均可:

```java
Generic<String> generic = new Generic<String>();
Generic<String> generic = new Generic<>();
Generic<String> generic = new Generic();
```

不定义泛型的类型就默认为Object

```java
Generic generic = new Generic();
```

泛型类不支持基本数据类型,只支持包装类的类型

```java
Generic<int> generic = new Generic();//报错
```

## 泛型类派生子类

- 子类中的标识必须要和父类中的一致

# 序列化

## 序列化

- 序列化:以一种形式(二进制字节流,json,String...)持久化地把对象从JVM中转化为其他数据格式进行存储

- 目的:方便对象的数据持久化存储或者网络传输

- 实例:

  ```java
  User(name=Alice, age=13)//序列化前
  ```

  ```json
  "{
      "@class":"redis.demo.data.User",
  	"name":"Alice",
  	"age":13
  }"//序列化后
  ```

## 反序列化

- 反序列化:以一种形式把对象从其他数据格式转化为java对象
- 注意:反序列化**必须要提前知道对应的是哪个java类**

## JSON转换

- 序列化:使用ObjectMapper类中的writeValueAsString()成员方法
- 反序列化:使用ObjectMapper类中的readValue(String,类名.class)方法

```java
private static final ObjectMapper mapper = new ObjectMapper();
public static void main(){
    String json = mapper.writeValueAsString(new User("Alice",17));//序列化为JSON字符串
    User res = mapper.readValue(val,User.class);//反序列化为对象,User.class告诉目标类为User
}
```



# 注解

## 什么是注解(annotation)

- 形如**@Override**,写在类或者属性,方法的上方
- 内置注解:
  - @Override:用于修辞方法,重写方法的声明(不是重写的方法会报错)
  - @Deprecated:用于修辞方法,属性,类;表示不推荐程序员使用这样的元素(已废弃,有删除线)
  - @SupressWarnings("all"):用于修辞方法,属性,类;去除警告线或者代码提示

## 元注解(meta annotation)

- 元注解用来注解其他注解,用来定义设置其他注解

#### JAVA中有四个元注解:
  - **@Target:用于描述注解的使用范围**
      - METHOD:注解方法
      - TYPE:注解类,接口,枚举
      - PARAMETER:注解方法参数
      - FIELD:成员变量
  - **@Retention:用于描述该注解的生命周期(~~SOURCE~~<~~CLASS~~<RUNTIME)**
      - SOURCE:只在源代码中有效,不会出现在字节码中.例如:@Test用于标记测试方法,但是运行时不会对测试方法造成任何影响
      - CLASS:只在编译和加载阶段有效,例如@Autowired在编译时进行依赖注入,但在运行时就不会影响
      - **RUNTIME**:运行时保留,在源码编译加载运行时都有效
  - @Document:说明该注解将被包含在javadoc中
  - @Inherited:说明子类可以**继承**父类中的该注解

## 自定义注解

```java
@Target(value= {ElementType.METHOD,ElementType.TYPE,ElementType.FIELD,ElementType.PARAMETER})
@Retention(value = RetentionPolicy.RUNTIME)
@Documented
@Inherited
public @interface MyAnnotation {
    //注解的参数
    String value() default "";
    int id() default -1;
}
```
- 注解本身不提供任何功能,只是做了一个标记
- 可以通过**反射机制读取**,并根据注解的信息**执行特定的逻辑**

# 反射概述

## 动态语言VS静态语言

- 动态语言可以在运行的时候改变其结构,比较灵活:

	```python
obj.newAttribute = {newValue}#向对象中添加新的属性和方法
  ```

- 静态语言在编译之后结构就已经确定,性能较好

- JAVA虽然是静态语言,但是可以根据反射赋予一定动态性

## 反射(Reflection):class对象==>类信息

- 反射机制允许程序在**执行期间**借助于ReflectionAPI**取得任何类的内部信息(类名,接口,方法,属性...)**,并**能直接操作任意对象内部属性以及方法**
- 加载完类之后,堆内存的方法区中就会产生一个Class类型的对象(一个类只有一个Class对象),这个对象就包含了完整的类的结构信息,我们可以通过这个对象看到类的结构(这个对象就像一面镜子通过镜子看到类的结构,所以称之为"反射")
- 反射对性能有负面影响,new出来的变量比反射快几十倍

## 获得反射对象

Class类(用于描述类的类):

- Object类中定义了**getClass()方法**会被所有子类继承
- getClass()方法的返回值是一个**Class类对象**

```java
//通过反射获取类的Class对象
Class c1 = Class.forName("org.example.User");
//output: class org.example.User
System.out.println(c1);
```

获取Class类对象

- 若已知具体的类,通过类的class属性获取

  ```java
  Class clazz = Person.class;
  ```
  
- 已知某个类的实例,调用该实例的getClass()方法获取Class对象

  ```java
  Class clazz = person.getClass();
  ```

- 已知一个类的全名,且该类在类路径下,可以通过Class类的静态方法forName()获取

  ```java
  Class clazz = Class.forName("com.demo.Person");
  ```

**按住ALT再选中,可以跨行选中**

## 同一个Class

只要元素类型和维度一样,就是同一个Class对象

```java
@Test
public void test02(){
    int[] a = new int[10];
    int[] b = new int[1000];
    //输出结果相同
    System.out.println(a.getClass().hashCode());
    System.out.println(b.getClass().hashCode());
}
```

# 反射底层原理

## JAVA内存分析

- 堆:存放**new创建的对象和数组**(引用类型变量),可以被所有线程共享
- 栈:存放**基本变量类型**,存放堆中引用类型变量的**地址**
- 方法区:包含了所有class和static变量,可以被所有线程共享

## 类的加载过程

- 加载:将**类的class文件**加载到**内存的方法区**中,并生成一个**java.lang.Class对象**(static)
- 链接:
  - 验证:确保加载的类信息符合JVM规范(格式验证和加载同步进行,其他的验证在方法区中进行)
  - 准备:正式为类变量(static)分配内存并设置类当中的变量**默认初始值(eg:null,0...)**
  - 解析:将符号引用(**类,接口,属性和方法**的符号)转化为直接引用(变量内存地址)
- 初始化:
  - 先进行父类的初始化
  - 然后初始化类的静态(static)部分

## 什么时候会发生类初始化

类的主动引用(一定会发生类的初始化):

- 当虚拟机启动,先初始化main方法所在的类
- new一个类的对象的时候会初始化类
- 调用类的静态成员(除了final常量)和静态方法
- 使用java.lang.reflect方法对类进行反射调用
- 当初始化一个类,如果其父类没有被初始化,则会先初始化它的父类

类的被动引用(不会发生类的初始化):

- 通过子类访问父类的静态变量或者静态方法

- 创建类的数组

  ```java
  Son[] sons = new Son[5];//不会触发类的初始化,因为只是预留一块空间
  sons[0] = new Son();//会触发类的初始化
  ```

- 访问类中的常量(在常量池中):不会触发类的初始化(常量早就在常量池中了) 

## 类加载器

- 

# 反射使用

## 获取类的运行时结构

```java
//获取类名
System.out.println(c1.getName());//获得类的包名+类名
System.out.println(c1.getSimpleName());//获得类的包名+类名

//获取属性信息
Field[] fs1 = c1.getFields();//获取所有公开的属性信息
Field[] fs2 = c1.getDeclaredFields();//获取所有属性信息
Field field1 = c1.getDeclaredField("name");//获取指定的属性信息
Field field2 = c1.getField("name");//获取指定的属性信息

//获取方法信息
Method[] ms1 = c1.getMethods();//获取所有公开的方法信息(包括继承的方法)
Method[] ms2 = c1.getDeclaredMethods();//获取所有定义的方法信息(不包括继承)
Method method = c1.getDeclaredMethod("setName",String.class);//获取指定的方法信息(需要指定参数类型)

//获取构造器
Constructor[] constructors = c1.getDeclaredConstructors();
Constructor constructor = c1.getDeclaredConstructor(String.class,int.class,int.class);
```

## 运行方法

```java
//创建类的实例
User user = (User)cls.getDeclaredConstructor().newInstance();
//让私有成员方法可以访问
method.setAccessible(true);
//执行method对象
method.invoke(类的实例,参数);
```

完整过程示例:

```java
@Test
    public void test03() throws ClassNotFoundException, NoSuchMethodException, InstantiationException, IllegalAccessException, InvocationTargetException {
        Class clx = Class.forName("org.example.User");
        Method getname = clx.getDeclaredMethod("getName");
        Method setname=  clx.getDeclaredMethod("setName", String.class);
        getname.setAccessible(true);
        setname.setAccessible(true);
        User user = (User)clx.getDeclaredConstructor(String.class,int.class,int.class).newInstance("Alice",309,17);
        setname.invoke(user,"Jack");
        String name = (String)getname.invoke(user);
        System.out.println(name);
    }
```

# 反射性能

