# Go

## workspace

`GOPATH`搜索包的路径

```shell
# export env
export GOPATH=~/workspace
# go inside the workspace directory
cd ~/workspace
```

## Variables

普通

```go
var a int
a = 1
```

快捷

```go
message := "hello world"
```

单行多变量赋值

```go
var b, c int = 2, 3
```

## 数据类型

### Number, String, and Boolean

Number:
    int, int8, int16, int32, int64,
    uint, uint8, uint16, uint32, uint64, uintptr…

String:

Bool:

Complex number:

```go
var a bool = true
var b int = 1
var c string = 'hello world'
var d float32 = 1.222
var x complex128 = cmplx.Sqrt(-5 + 12i)
```

### Arrays, Slices, and Maps

- Array

特征: 相同数据类型元素，拥有固定长度
缺陷：在运行时改变数组的值有限制，不提供获取子数组的能力。

```go
var a [5]int
```

多维数组

```go
var multiD [2][3]int
```

- Slices

定义：

无容量定义

```go
var b []int
```

拥有长度和容量的slice

```go
numbers := make([]int,5,10)
```

## 