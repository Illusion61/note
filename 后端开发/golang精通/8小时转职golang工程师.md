# 8小时转职golang工程师

## golang基础语法

#### 变量

- 变量声明

```go
var 变量名 变量类型 = 值
```

```go
package main;
import "fmt"
func main(){
    //三种方式:
    //第一种 var var_name var_type;赋予默认值为0
    var a int//a==0
    //第二种 var var_name = value;根据value的类型自动判断变量类型
    var b = 5
    //第三种 var var_name var_type = value;给出value和type
    var c int = 10
    //第四种 var_name:=5;省略var关键字
    d := 3.14
}
```

- 多变量声明(不需要写类型)

```go
var a1,b1 = 123;
c1,d1 := 789,"liubang"
```

- 常量const

```go
const (
    Byte = iota
    KB = 1<<(10*iota)
    MB
    GB
)
const LENGTH int = 10
const a,b,c = 1,false,"str"
```

#### 函数

```go
func swap(x,y string)(string string){
    return y,x
}

```

## go运行环境配置

#### Go PATH

- GOPATH:早期处理版本依赖的机制

```go
go env//输出所有go环境变量,其中有GOPATH
GOPATH目录:bin所有依赖版本二进制文件
```

- 缺点:
  - 没有版本控制机制
  - 没法同步第三方版本号
  - 没法指定当前项目引用的第三方版本号

#### GO Modules

```go
go env//输出所有go环境变量
go mod help//查看go mod命令指南
```

- 需要配置的go环境变量

```go
GO111MODULE:on/off/auto//控制是否开启go module模式,auto是如果项目目录中有go.mod就启动
GOPROXY:url,direct//先从哪个第三方平台去下载库,direct设置再从模块版本的源地址去抓取
GOSUMDB:sum.golang.org,direct//checksum database:验证下载的库有没有被篡改

```

