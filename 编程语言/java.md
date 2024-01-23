# JAVA基础语法

## JAVA简介

#### JAVA语言特性

- 所有代码都被包含在类之中
- 在变量被使用之前,必须被声明
- 编译器检查变量类型兼容之后再运行程序
- java中的final类似其它语言的const

#### JVM,JRE,JDK

JDK包含JRE,JRE包含JVM

- JVM(java virtual machine):java虚拟机,java程序在JVM上进行运行.
  - JVM是java跨平台的关键,屏蔽了不同OS之间的差异,可以让相同java程序在不同OS上得到相同结果
  - JVM提供线程管理,内存管理,垃圾回收等众多功能
- JRE(java runtime environment):运行java已编译程序所必须的软件环境
  - 包含JVM,java标准类库
  - 只想运行java就安装JRE,需要开发java再安装JDK
- JDK(java development kit):java开发工具包
  - 包含JRE,编译器,调试分析等工具软件
  - 能够将java源文件编译为字节码文件(.class)再交给JRE运行

#### JAVA的编译

```shell
javac Hello.java #生成Hello.class二进制字节码文件
java Hello #交给JRE运行Hello.class文件,后面不应该加.class后缀
```

- jar包:jar包用于将多个相关的.class文件和资源打包在一起
  - 可以包含多个.class文件,资源文件,配置文件等
  - .jar文件可以用于构建和发布java库,应用程序和模块
  - .jar文件可以被JRE和JVM加载和执行,运行其中包含的JAVA程序

## 面向对象编程

#### 方法

- constructor
- getter/setter
- toString()
- equals()?????????????
- hashCode()?????????????

#### 静态方法VS动态方法

- 静态方法:方法不需要访问实例变量,不依赖于特定对象的状态,并且仅仅对类级别的操作进行定义,选择静态方法
  - Array.sort()对类进行排序
- 动态方法:方法需要访问实例变量,对属性进行修改或者依赖于属性的值
  - String s;s.length();
  - s.setName();

#### 静态变量VS成员变量

- 静态变量:定义类的时候就会初始化静态变量,用于在整个类中共享数据,可以被所有实例访问.
  - 点心类中点心数量(每次初始化一个点心都+1)
- 成员变量:适用于每个类实例都具有不同值的情况,每个实例都拥有自己的成员变量副本.
  - Dog类中weight属性

## 引用数据类型

#### 创建一个类的实例

- `Dog dog = new Dog();`
- `Dog[] dogs = new Dog[2];`
  - `dogs[0] = new Dog();`
  - `dogs[1] = new Dog();`

```java
public class Test{
    public static void main(String[] args){
        Dog dog = new Dog();
    }
}
```

#### 引用数据类型比较

```java
String a = "abc";
String b = "abc";                                                                                                                       
a.equals(b);//正确
a == b;//错误,==比较的是两个变量的地址是否相同
```

## JAVA内置数据类型类

#### String字符串



#### Array数组



#### List列表

- 实现方式:
  - ArrayList数组列表
  - LinkedList链表列表

#### Map字典

- 实现方式:
  - HashMap:哈希表
  - TreeMap:树表

#### Set集合

- 实现方式:
  - HashSet哈希集合