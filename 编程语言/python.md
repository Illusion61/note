#  Anaconda基础操作

## 虚拟环境管理

```conda
#创建虚拟环境
create -n env_name [python=x.x]
#删除虚拟环境
remove -n env_name --all
#激活虚拟环境
conda activate env_name
#退出虚拟环境
conda deactivate
#显示虚拟环境列表
conda env list
```

##  常见错误

- 安装package之后依然提示无法找到:重新选择一遍jupyter-kernel
- 

```conda
任务一：回归任务—地震等级预测
（1） 数据集：quake.dat
• 三个特征值：Focal_depth（震源深度），Latitude（纬度），Longitude（经度）；
• 真实值：Richter（里氏震级）。
（2） 数据集划分（共2178项）：前2000项为训练集，后178项为测试集。
（3） 衡量指标：均方误差（MSE）和均方根误差（RMSE）
在完成这个任务的时候,在测试集上进行预测:
使用线性回归的结果是 均方误差（MSE）: 0.028279088703901542,均方根误差（RMSE）: 0.16816387455069398
使用SVM回归的结果是 均方误差（MSE）: 0.03262782705785472,均方根误差（RMSE）: 0.18063174432489634
使用LSTM的结果是 均方误差（MSE）: 0.03359811358271728,均方根误差（RMSE）: 0.18329788210101414

使用逻辑回归的结果是：
请分析以上三种算法哪种在这个任务中表现最好,以及每种算法在这个任务中表现优劣的原因
```

|           |      逻辑回归      | SVM分类 | 残差神经网络 |
| --------- | :----------------: | :-----: | :----------: |
| precision | 0.9871794871794872 |   1.0   |     1.0      |
| recall    | 0.9871794871794872 |   1.0   |     1.0      |



# Python基础

## 第一部分:python类基础

#### 可变参数args,kwargs

- 注意:调用方法的时候:*args必须在**kwargs前面
- 推荐格式:method(self,arg1,...argn,*args,**kwargs)

```python
# *args
def getmax(*args):
	maxnum = -1e9
	for num in args:
		if num > maxnum:
			maxnum = num
	return maxnum
# **kwargs
def printconf(*args,**kwargs):
	for item in args:
		print(item)
	for key,value in kwargs.items():
		print(f'{key}={value}')

print(printconf('apple',1,'orange',1122,name='harry',age=21))
```

#### python定义类

```python
# 定义构造方法
	# 注意:__init__()的时候被初始化的成员变量(参数)可以在之前没有定义
def __init__(self,参数1...参数N)#构造方法会自动运行

#定义成员方法
def 方法名(self,参数1...,参数N,*args,**kwargs)
	#self关键字是必须要填写的
    #方法内部想要访问成员方法必须使用self
    self.other_methods(...)
    #默认调用成员方法的时候会自动传入self参数

#魔术方法(类内置方法)
__init__(self,param1,...paramn)#初始化方法
__str__(self)#字符串方法(java中的toString()),返回字符串
__lt__(self,other)#less than比较方法,定义对象大于/小于的规则
__le__(self,other)#less than or equal to,定义对象大于等于/小于等于的规则
__eq__(self,other)#equal to,定义等于的规则
__len__(self)#返回对象的长度len(obj)

#定义私有成员变量或者方法
以__开头的变量:__name,__age...
```

```python
class Student:
    __name = None         #记录学生姓名
    __gender = None       #记录学生性别
    __nationality = None  #记录学生国籍
    __age = None          #记录学生年龄
    def __init__(self,name,gender,nationality,age):
        self.__name = name
        self.__gender = gender
        self.__nationality = nationality
        self.__age = age
    def __str__(self):
        return f"Student类对象[姓名:{self.__name},性别:{self.__gender},国籍:{self.__nationality},年龄:{self.__age}]"
    def __lt__(self,other):
        return self.__age < other.__age
    def __le__(self,other):
        return self.__age <= other.__age
    def __eq__(self,a):
        return self.__age == a.__age
    def getName(self):
        return self.__name
    def sayHello(self):
        print(self.__name+"对你说:Hello")
    def sayHello2(self,msg):
        print(self.__name+"对你说:"+msg)
a = Student("Jason","男","中国",31)
b = Student("Harry","男","英国",32)
print(a!=b)
a.sayHello()
a.sayHello2("我叫{},以后多多关照".format(a.getName()))
print(a.name)
```

#### python类的继承

- 单继承:子类名(父类名)
  - 子类默认init方法是无参构造方法并且调用父类的无参构造方法
  - 子类默认继承父类的构造方法
- 多继承:子类名(父类名1,父类名2....)
  - 多个父类优先级:左边优先于右边(父类1最优先)
  - 子类无法访问父类中的私有成员,但是可以访问父类中的get/set方法
- 子类中如何调用父类中被重写的成员变量或者成员方法
  - 父类名().成员变量/成员方法()
  - super().成员变量/成员方法()

```python
class Phone:
    __is_5g_enable:bool
    def __init__(self,fg):
       self.__is_5g_enable = fg
    def __check_5g(self):
        if self.__is_5g_enable:
            print("5g开启")
        else:
            print("5g关闭,使用4g网络")
    def call_by_5g(self):
        self.__check_5g()
        print("正在通话中")
class Phone2022(Phone):
    face_recognition = true
    #重写方法
    def call_by_5g(self):
        super().call_by_5g()
        print("最新5g通话2022")
ph1 = Phone2022(True)
ph1.call_by_5g()
```



## 第二部分 python内置数据类型

#### 匿名函数

- lambda函数:

```python
func = lambda a:a**3
print(func(2))
```



