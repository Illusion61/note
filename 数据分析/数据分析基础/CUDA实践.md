# NVIDIA显卡

## 1.1 显卡命名规则

- NVIDIA产品:
  - GeForce:面向消费者市场
    - GTX:提供强大的游戏图形渲染能力
    - RTX:在GTX的基础上加入实时光线追踪和深度学习支持,适合跑模型
  - TITAN:面向高端消费者市场
  - Quadro:面向专业设计站和设计领域
  - Tesla:面向高性能计算和数据中心

- 型号:一般意义上,对于同一个系列(都是GTX),数字序号越大性能越好
- 后缀:Ti表示加强版,SUPER表示加强版

## 1.2 CUDA兼容性检测

- 查看显卡型号

```bash
lspci | grep -i nvidia #对于linux设备查看显卡型号
```

- 进入[nvidia官网](https://developer.nvidia.com/cuda-gpus)查看是否兼容cuda
- 进入[gpu数据库](https://www.techpowerup.com/gpu-specs/)查看gpu性能数据和软件支持

## 1.3 GPU的管理检测

```bash
nvidia-smi #cuda附带的工具,查看gpu
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 537.42                 Driver Version: 537.42       CUDA Version: 12.2     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                     TCC/WDDM  | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  NVIDIA GeForce GTX 1650      WDDM  | 00000000:01:00.0 Off |                  N/A |
| N/A   65C    P0              16W /  40W |   3816MiB /  4096MiB |     33%      Default |
|                                         |                      |                  N/A |
+-----------------------------------------+----------------------+----------------------+
GPU:多个显卡有自己不同的编号,当前gpu显卡编号为0
Name:显卡名称NVIDIA GeForce GTX 1650
Temp:显卡温度65C
Perf:显卡性能等级P0最高最耗电,最小好像能到P12
Memory-Usage:内存占用
GPU-Util:显卡利用率
+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|    0   N/A  N/A     22616    C+G   ...apps\common\Stellaris\stellaris.exe    N/A      |
+---------------------------------------------------------------------------------------+
显示当前占用GPU的进程的信息

nvidia-smi -L #查询显卡型号
GPU 0: NVIDIA GeForce GTX 1650 (UUID: GPU-fbd80071-53ac-fcac-5ad3-e570f6427a8b)

nvidia-smi -q #查询显卡详细信息
```

## 1.4 nvcc的使用

- nvcc是cuda/c++的编译器

#### 工作过程

- 预处理:处理#include的相关内容
- 解析:构建语法树,检查语义错误,生成中间代码
- 依赖分析:确定哪些函数在GPU上执行,哪些函数在CPU上面执行
- **设备代码生成**:nvcc会把GPU(device)上运行函数代码转化为适用于NVIDIA-GPU的底层**PTX代码**
- **主机代码生成**:nvcc会把在CPU(host)上运行的代码用**编译器(gcc)**进行编译
- 链接:nvcc把生成的设备代码与主机代码和其他库文件代码进行链接,以生成可执行文件

#### 参数

- 参数类型:

  - 长命名参数(--include-path):一般用在脚本中,方便开发者阅读知道含义
  - 短命名参数(-i):一般直接用于交互式命令,方便打字

- 查看[官网教程](https://docs.nvidia.com/cuda/archive/12.2.1/cuda-compiler-driver-nvcc/index.html#command-option-types-and-notation)

  - 设置文件和路径:

  ```nvcc
  --output-file [filename] (-o):指定输出的文件名
  --pre-include [file1,file2...] (-include):指定引入的文件名称
  ```

  - 指定编译过程相关的参数:

  ```nvcc
  
  ```

  - 指定编译,链接过程的参数

  ```nvcc
  
  ```

  - 传递编译过程相关的参数
  - 指定nvcc行为相关的参数
  - 控制CUDA编译过程相关的参数
  - 控制GPU代码生成相关的参数

# CUDA编程上手

## 2.1 CUDA线程模型

- CUDA代码分类:
  - host-program:运行在CPU上面的代码
  - kernel-program:运行在GPU上面的代码(以成千上万条线程来执行)
- 线程模型分级:
  - **GRID**(网格):内核(kernel)执行时所产生的所有线程称为grid
  - **Block**(块):一个Grid有多个Block
    - 一般Block运行在流多处理器(Stream-MultiProcessor)上
    - **Block内部**的**线程**可以进行**数据交换**
    - 位于不同Block之间的线程很难做数据交换
  - **Thread**(线程):最小的执行单位,在SP上面执行
- GRID和Block可以是多维(最多三维?)的
- 线程标识:
  - 每一个线程是由**blockIdx**和**threadIdx**唯一标识
  - blockIdx和threadIdx是三维向量,通过下标x,y,z访问(blockIdx.z)
- 线程模型维度:
  - 线程模型维度由**blockDim**和**gridDim**标识
  - blockDim和gridDim也是三维向量
- 线程管理
  - threadId由threadIdx计算得到:threadId = (i+j*Dx)

## 2.2 CUDA内存模型

- **Register**:**每一个线程**都由自己**独有的Register**,是最快的内存,内核代码中没有任何修饰符的自动变量通常存放在寄存器中
- **LocalMemory**:本地内存是每个线程**独有的**,内核函数中无法存放到寄存器中的变量会被存放到本地内存中
- **sharedMemory**:`__shared__`修饰的变量存放在共享内存中,**由一个Block内的线程共享**,具有低延迟和高带宽

- constantMemory:`__constant__`修饰的内存,常量内存可以被**所有内核代码访问**
- globalMemory:
  - 全局内存数量最大,使用最多,延迟最大,通常用于与CPU做**数据交换**
  - 静态分配:使用`__device__`关键字
  - 动态分配:使用主机中的内存管理函数cudaMalloc,cudaMemcpy,cudaMemset,cudaFree

## 2.3 CUDA内核函数

#### CUDA内核函数基础

- CUDA函数分类:

|      Qualifiers      | Execution  |        Callable         |      Notes       |
| :------------------: | :--------: | :---------------------: | :--------------: |
|     `__global__`     | **device** | **host**/certain-device |   必须返回void   |
|     `__device__`     | **device** |     **device**-only     |       N/A        |
|      `__host__`      |  **host**  |      **host**-only      | 默认host可以省略 |
| `__device____host__` | **device** |     **device/host**     |                  |

- 内核函数:kernel-function在GPU线程上并发执行
  - 只能访问GPU内存
  - 必须返回void
  - 不能使用变长参数
  - 不能使用函数指针
  - 不能使用静态变量
  - 内核函数具有异步性:内核函数的线程之间具有异步性,内核函数本身和CPU函数本身也不同步

#### CUDA HelloWorld

- 内核执行配置(kernel-execution-configuration):<<<grid,block>>>
- 释放所有与GPU相关的GPU资源:cudaDeviceReset();

```c++
#include<stdio.h>
__global__ void helloFromGPU(){//定义核函数为全局类型
    printf("Hello World From GPU\n");
}
int main(){
    printf("Hwllo World From CPU\n");
    helloFromGPU<<<4,10>>>();//调用核函数
    cudaDeviceReset();//释放GPU资源
    return 0;
}
```

#### CUDA 获取线程索引

- grid(gridDim)和block(blockDim)的变量类型都是**dim3**,blockIdx和threadIdx的类型是uint3(一种三个uint组合的数据类型)
- blockDim和gridDim用来获取全局的block和grid维度信息
- blockIdx和threadidx是当前线程的编号

```c++
#include<stdio.h>
__global__ void helloFromGPU(){
    printf("blockDim:x=%d,y=%d,z=%d; gridDim:x=%d,y=%d,z=%d; Current blockIdx:x=%d,y=%d,z=%d; Current threadIdx:x=%d,y=%d,z=%d;\n",blockDim.x,blockDim.y,blockDim.z,gridDim.x,gridDim.y,gridDim.z,blockIdx.x,blockIdx.y,blockIdx.z,threadIdx.x,threadIdx.y,threadIdx.z);
}
int main(){
    printf("HelloFromCPU\n");
    dim3 grid;grid.x=2;grid.y=2;//grid.x,grid.y,grid.z都默认为1
    dim3 block;block.x=2;block.y=2;
    helloFromGPU<<<grid,block>>>();
    cudaDeviceReset();
    return 0;
}
```

## 2.4 CUDA信息获取

#### 错误信息检查

#### 运行时GPU信息查询

- 获取GPU数量:**cudaGetDeviceCount**
- 设置需要使用的GPU:**cudaSetDevice**
- 获取GPU信息:**cudaGetDeviceProperties**(cudaDeviceprop)

```c++
#include<cuda_runtime.h>
#include<stdio.h>
int main(int argc,char **argv){
	printf("%s Starting...\n",argv[0]);

	///判断GPU数量,如果为0直接退出
	int deviceCount = 0;
	cudaGetDeviceCount(&deviceCount);
	if(deviceCount == 0){
		printf("There is no available device(s) that support CUDA\n");
		return 1;
	}
	else{
		printf("Detected %d CUDA Capable device(s)\n",deviceCount);
	}

	///设置默认查询的GPU
	int device=0,driverVersion=0,runtimeVersion=0;
	cudaSetDevice(device);
	
	///查询显卡相关信息
	cudaDeviceProp deviceProp;
	cudaGetDeviceProperties(&deviceProp,device);
	cudaDriverGetVersion(&driverVersion);
	cudaRuntimeGetVersion(&runtimeVersion);
    //deviceProp.maxBlocksPerMultiProcessor,deviceProp.maxThreadsPerBlock,deviceProp.maxThreadsPerMultiProcessor,deviceProp.multiProcessorCount
}
```

## 2.5 CUDA性能提升

#### warp

- Block中的thread在执行的时候是以warp为SIMD的单位进行执行的,如果thread数量不是deviceProp.warpSize(32)的**整数倍**,就会浪费一部分性能
- Block在唯一一个SM上运行,一个SM上面可以有多个Block
- Block的作用是里面的thread共享内存,warp则是整数倍提升性能

#### PTX兼容性

- 较低GPU计算能力:PTX兼容性设置的比较低,就能在更多终端上运行;PTX兼容性设置的版本高,就能更充分发挥高级版本GPU的性能

```bash
nvcc ./helloworld.cu -ptx#输出同名ptx文件,查看默认ptx兼容性
nvcc ./helloworld.cu -arch=compute_75#编译的时候指定兼容性版本,发挥更高架构等级的实力
```

#### 二进制兼容性

#### CUDA运行时库

## 2.6 CUDA编程框架

#### 程序结构

- 设置GPU设备:多个GPU用哪个
- 初始化矩阵
- **定义CUDA内核函数**
- **分配GPU内存**(主机运行程序分配GPU内存)
- 将**数据传入GPU内存**并计算 
- 在主机中获取计算结果

#### CUDA同步方法

- GPU内核和主机线程以**异步方式执行**
- 个别GPU运行库API包含隐式的同步处理(cudaMemcpy...)
- GPU内核与主机线程同步接口:cudaDeviceSynchronize()等待device上面的计算结束后再继续运行

#### 检测内核执行时间

- CPU计时器
- nvprof:分析执行时间工具(**需要使用root权限**)

```bash
nvprof [options] [application] [application arguments]
#注意:nvprof需要管理员或者root权限
nvprof .\add.exe
==8940== Profiling application: .\add.exe
==8940== Profiling result:
            Type  Time(%)      Time     Calls       Avg       Min       Max  Name
 GPU activities:   69.44%  479.58us         3  159.86us  159.17us  161.05us  [CUDA memcpy HtoD]
                   22.94%  158.46us         1  158.46us  158.46us  158.46us  [CUDA memcpy DtoH]
                    7.62%  52.607us         1  52.607us  52.607us  52.607us  sumArrayOnGPU(float*, float*, float*, int)
#可以看到主要的时间都花费在了主机和设备之间的数据传递上面,真正用于计算的时间很少
```

## 2.7 索引

```c++
__global__ void matrixAdd(int* A,int* B,int* C){
    int ix = blockIdx.x*blockDim.x + threadIdx.x;//块ID*块宽度+线程ID
    int iy = blockIdx.y*blockDim.y + threadIdx.y;
    int id = iy*nx + ix;//GPU中的多维矩阵按照线性存储,相当于matrix[z][y][x]
}
```

```c++
//设置block数量:向上取整
dim3 block;
grid.x = (xsize+block.x-1)/block.x
```

## 2.8 线程束分支优化

- 类似这种的程序就会产生分支

```c++
__global__ void matrixGenerate(int* M){
    int x = blockIdx*blockDim.x + threadIdx.x;
    int type = x%2;
    if(type){
        M[x] = 100;
    }
    else{
        M[x] = 200;
    }
}
```

- warp当中如果有一部分的线程要执行分支,一部分的不要执行,就会导致每次只能执行一部分,需要执行多次
- 如果能控制好每个warp内部的线程都只执行**同一个分支**,就能极大提升效率