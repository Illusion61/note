# Pytorch

## 加载数据集(预处理)

#### torchvision中的数据集

``` python
from torchvision import datasets

```



## 定义神经网络

#### 通用语法

- 步骤:
  1. 自定义子类继承nn.Module类
  2. 重写init方法(用来定义子模型)
     - 重写init方法要**先调用super的init方法**
     - 定义后续将会用到的网络(计算**维度**)
     - 定义激活函数
  3. 重写forward方法(用来进行前进)
- 注意:
  - **设计网络之前要先计算好每一层的维度(数据集维度,中间层维度,输出维度)**
  - nn和F的区别:
    - nn是定义并创建网络对象,之后重载()运算来forward
    - F是直接就是function,直接运算
    - nn对象有可学习的参数,F函数没有

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
class ImageClassifier(nn.Module):
    def __init__(self):
        super().__init__()#必须要有,用来初始化nn.Module中很多参数(训练/预测,cpu/gpu,...)
        self.net = nn.Sequential(
            nn.Linear(xx,yy)
            ......
        )
    def forward(self,x):
        out = F.relu(self.net1(x))
        out = self.net2(out)
        x = self.net3(x)
model = ImageClassifier()
model.cuda()
criterion = nn.CrossEntropyLoss()
optimizer = torch.optim.SGD(model.parameters(),learning_rate)
```

#### CNN神经网络

参考:https://zhuanlan.zhihu.com/p/251068800

![v2-8e2add9b9d3422019dfec3048e66e1a9_1440w](C:\Users\nightmare00\Desktop\capture\v2-8e2add9b9d3422019dfec3048e66e1a9_1440w.webp)

- CNN机制:

  - 每个filter和上一层n个channel的**每一层进行卷积操作**,之后将结果**相加**,并在新矩阵的每一位**加上偏置单元b(一个数字)**
  - 总共有n_new个filter,将这些filter计算出来的二维矩阵拼接在一起构成新的一层
  - 每一层参数总共有`n_new*n*k*k+n_new`个(加上每个卷积核带有的偏置单元)
  - **新的一层维度**:
    - 卷积层:`m'=(m+2p-k)/s`(矩阵尺寸+padding*2-卷积核尺寸)/步长+1
    - 池化层:`m'=(m-k)/s+1`(矩阵尺寸-池化核尺寸)/步长+1

  ![20240101133024_waifu2x_256x256_1n_jpg](C:\Users\nightmare00\Desktop\capture\20240101133024_waifu2x_256x256_1n_jpg.png)

- CNN注意点:

  - 前面卷积核要大,频道数小;后面卷积核尽量小,频道数多
  - padding策略:
  - 池化层分为最大池化和平均池化:

- 代码:
  - nn.Conv2d(in_channel,out_channel,kernel_size,stride,padding)
  - nn.MaxPool2d(kernel_size,stride,padding)
  - nn.BatchNorm2d(channel)
  - nn.Flatten()

```python
class ImageClassfier(nn.Module):
    def __init__(self):
        super().__init__()
        self.net = nn.Sequential(
            #[3,32,32]
            nn.Conv2d(3,64,3,1,1),#[64,32,32]
            nn.BatchNorm2d(64),
            nn.ReLU(),
            nn.Conv2d(64,128,3,1,1),#[128,32,32]
            nn.BatchNorm2d(128),
            nn.ReLU(),
            nn.MaxPool2d(2,2,0),#[128,16,16]
            nn.Conv2d(128,256,3,1,1),#[256,16,16]
            nn.BatchNorm2d(256),
            nn.ReLU(),
            nn.MaxPool2d(2,2,0),#[256,8,8]
            nn.Conv2d(256,512,3,1,1),#[512,8,8]
            nn.BatchNorm2d(512),
            nn.ReLU(),
            nn.MaxPool2d(2,2,0),#[512,4,4]
            nn.Flatten(),
            nn.Linear(512*4*4,1024),
            nn.ReLU(),
            nn.Linear(1024,256),
            nn.ReLU(),
            nn.Linear(256,10),
            nn.Sigmoid()
        )
    def forward(self,x):
        return self.net(x)
```

#### RNN神经网络

```python

```

## 定义损失函数和优化方法

#### 常见损失函数

- 交叉熵:nn.CrossEntropyLoss()

#### 优化方法

- torch.optim.SGD(model.parameters(),learning_rate)

```python
model = ImageClassfier()
model.cuda()
criterion = nn.CrossEntropyLoss()
optimizer = torch.optim.SGD(model.parameters(),learning_rate)
```



## 训练

