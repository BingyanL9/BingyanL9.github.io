---
layout: post
title:  "Go - 变量（一）"
date:   2020-04-13 21:18:54
categories: Go Foundation
tags: Notes Go Variable
excerpt: Go变量简介
mathjax: true
---

* content
{:toc}

> Go变量简介

### **Go变量的声明**

Go 使用 var 关键字进行变量声明：

```
var name type 
```

与 javaScrip 和 java 都不一样，Go 在变量声明时将变量的类型放在变量的后面，这样做的好处是可以避免向 C 语言中那样含糊不清的声明形式，如 ```int* a,b;``` 其中只有 a 是指针而 b 不是。 如果想要将两个都声明成指针，则需要将他们分开写。而在 Go 中，则可以一起声明：

```
var a,b *int;
```

变量声明一共有三种格式：

1. 标准格式： 

    ```
    var a int
    ```

2. 批量格式：

    ```
    var (
        a int
        b string
        c []float32
        d func() bool
        e struct {
            f int
        }
    )
    ```
    
3. 简短格式：

    ```
    a,b := 1, "abc"

    c := 3
    ```

    需要注意的是，简短模式（short variable declaration）有以下限制：

    - 定义变量，同时显式初始化

    - 不能提供数据类型

    - 只能用在函数内部

### **Go 命名**

Go语言中的函数名、变量名、常量名、类型名、语句标号和包名等所有的命名，都遵循一个简单的命名规则：一个名字必须以一个字母（Unicode字母）或下划线开头，后面可以跟任意数量的字母、数字或下划线。

- 关键字：

    Go语言中类似if和switch的关键字有25个；关键字不能用于自定义名字，只能在特定语法结构中使用

    ```
    break      default       func     interface   select
    case       defer         go       map         struct
    chan       else          goto     package     switch
    const      fallthrough   if       range       type
    continue   for           import   return      var
    ```

- 预定义的名字

    此外，还有大约30多个预定义的名字，比如int和true等，主要对应内建的常量、类型和函数:

    ```

    内建常量: true false iota nil

    内建类型: int int8 int16 int32 int64
            uint uint8 uint16 uint32 uint64 uintptr
            float32 float64 complex128 complex64
            bool byte rune string error

    内建函数: make len cap new append copy close delete
            complex real imag
            panic recover
    ```

### **Go变量初始化**

Go语言的基本类型有：

- bool

- string

- int、int8、int16、int32、int64

- uint、uint8、uint16、uint32、uint64、uintptr

- byte // uint8 的别名

- rune // int32 的别名 代表一个 Unicode 码

- float32、float64

- complex64、complex128

所有的内存在 Go 中都是经过初始化的，每个变量会初始化其类型的默认值，例如：

- 整型和浮点型变量的默认值为 0 和 0.0

- 字符串变量的默认值为空字符串

- 布尔型变量默认为 false

- 切片、函数、指针变量的默认为 nil

当然，依然可以在变量声明时赋予变量一个初始值。

变量初始化的格式：

1. 标准格式:

    ```
    var a int = 100
    ```

2. 编译器推导类型的格式 - 上面的100和 int 同为 int 类型，int 认为是冗余信息，因此有了进一步简化的初始化写法

    ```
    var a = 100
    ```

3. 短变量声明并初始化

    ```
    a,b := 1, "abc"

    c := 3
    ```

    **注意：**

    - 由于使用了 := ，而不是赋值的 = ，因此推导声明写法的左值变量必须是没有定义过的变量。若定义过，将会发生编译错误。

    - 在多个短变量声明和赋值中，至少有一个新声明的变量出现在左值中，即便其他变量名可能是重复声明的，编译器也不会报错，代码如下：

        ```
        conn, err := net.Dial("tcp", "127.0.0.1:8080")
        conn2, err := net.Dial("tcp", "127.0.0.1:8080")
        ```

        err 不会报错

### **Go变量多重赋值**

```
var a int = 100
var b int = 200
b, a = a, b
fmt.Println(a, b) // 200,100
```

多重赋值时，变量的左值和右值按从左到右的顺序赋值。

### **Go匿名变量**

匿名变量的特点是一个下画线```_```，```_```本身就是一个特殊的标识符，被称为空白标识符。它可以像其他标识符那样用于变量的声明或赋值（任何类型都可以赋值给它），但任何赋给这个标识符的值都将被抛弃，因此这些值**不能在后续的代码中使用**，也不可以使用这个标识符作为变量对其它变量进行赋值或运算。使用匿名变量时，只需要在变量声明的地方使用下画线替换即可。eg:

```
func GetData() (int, int) {
    return 100, 200
}

func main() {
    a, _ := GetData()
    _, b := GetData()
    fmt.Println(a, b)// 100, 200
}
```

匿名变量**不占用内存空间，不会分配内存**。匿名变量与匿名变量之间也不会因为多次声明而无法使用。

### **Go变量的作用域**

根据变量定义位置的不同，可以分为以下三个类型：

- 函数内定义的变量称为局部变量

- 函数外定义的变量称为全局变量

- 函数定义中的变量称为形式参数

1. 局部变量

    局部变量不是一直存在的，它只在定义它的函数被调用后存在，函数调用结束后这个局部变量就会被销毁。

2. 全局变量

    全局变量只需要在一个源文件中定义，就可以在所有源文件中使用。

    **全局变量声明必须以 var 关键字开头**，如果想要在外部包中使用全局变量的，这个局部变量的首字母必须大写。

3. 形式参数

    在定义函数时函数名后面括号中的变量叫做形式参数（简称形参）。形式参数只在函数调用时才会生效，函数调用结束后就会被销毁，在函数未被调用时，函数的形参并不占用实际的存储单元，也没有实际值。

    形式参数会作为函数的局部变量来使用。


### 笔记来源

[C语言中文网：Go语言简介](http://c.biancheng.net/golang/intro/)

[Go语言圣经:命名](https://docs.hacknode.org/gopl-zh/ch2/ch2-01.html)








