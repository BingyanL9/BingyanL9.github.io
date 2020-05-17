---
layout: post
title:  "Go - 变量（四）"
date:   2020-05-07 21:18:54
categories: Go Foundation
tags: Notes Go Variable
excerpt: Go变量 - 常量、枚举和strconv包
mathjax: true
---

* content
{:toc}

> Go变量 - 常量、枚举和strconv包

### **常量**

使用关键字 const 定义， 用于存储不会改变的数据，常量是在编译时被创建的。只能是**布尔型、数字型（整数型、浮点型和复数）和字符串型**。

常量的值必须是能够在编译时就能够确定的，可以在其赋值表达式中涉及计算过程，但是所有用于计算的值必须在编译期间就能获得。

```
显式类型定义： const b string = "abc"
隐式类型定义： const b = "abc"

批量定义：
const (
    e  = 2.7182818
    pi = 3.1415926
)
```

常量间的所有算术运算、逻辑运算和比较运算的结果也是常量，对常量的类型转换操作或以下函数调用都是返回常量结果：len、cap、real、imag、complex 和 unsafe.Sizeof。

如果是批量声明的常量，除了第一个外其它的常量右边的初始化表达式都可以省略，如果省略初始化表达式则表示使用前面常量的初始化表达式，对应的常量类型也是一样的。

```
const (
    a = 1
    b
    c = 2
    d
)
fmt.Println(a, b, c, d) // "1 1 2 2"
```

iota 常量生成器的实现就基于这一特性。

#### iota 常量生成器

Go 没有提供枚举类型的定义， 但可以通过const + iota 来模拟枚举类型。

```
type EventAppendStatus int // 需要 int32 和 int64 的枚举，也是可以的

const {
    _ EventAppendStatus = iota  //0
    Success  //1
    Failed  //2
    DuplicateEvent  //3
    DuplicateCommand  //4
}

func main() {
    fmt.println(DuplicateEvent)  //3
}
```

一个 const 声明内的每一行常量声明，将会自动套用前面的 iota 格式，并自动增加，类似于电子表格中单元格自动填充的效果，只需要建立好单元格之间的变化关系，拖动右下方的小点就可以自动生成单元格的值。

当然，iota 不仅可以生成每次增加 1 的枚举值。还可以利用 iota 来做一些强大的枚举常量值生成器。

```
const (
    FlagNone = 1 << iota
    FlagRed
    FlagGreen
    FlagBlue
)
fmt.Printf("%d %d %d\n", FlagRed, FlagGreen, FlagBlue)// 2 4 8
```

当然也可手动编写常量值。

#### 将枚举值转换为字符串

```
type ChipType int

const (
    None ChipType = iota
    CPU
    GPU    
)

func (c ChipType) String() string {
    switch c {
    case None:
        return "None"
    case CPU:
        return "CPU"
    case GPU:
        return "GPU"
    }
    return "N/A"
}

//String() 方法的 ChipType 在使用上和普通的常量没有区别。当这个类型需要显示为字符串时，Go语言会自动寻找 String() 方法并进行调用

func main() {
    
    fmt.Printf("%s %d", CPU, CPU)//CPU 1
}

```

### **Go语言type关键字（类型别名）**

类型别名是 Go 1.9 版本添加的新功能，主要用于解决代码升级、迁移中存在的类型兼容性问题

#### 区分类型别名与类型定义

```
类型别名：
type TypeAlias = Type
TypeAlias 只是 Type 的别名，本质上 TypeAlias 与 Type 是同一个类型

类型定义:
type TypeAlias Type
将 TypeAlias 定义为 Type 类型， Type 和 TypeAlias不是一个类型
```

eg:
```
package main

import (
    "fmt"
)

type NewInt int

type IntAlias = int

func main(){

    var a NewInt

    fmt.Println("a type: %T\n", a)// a type: main.NewInt

    var a2 IntAlias

    fmt.Println("a2 type: %T\n", a2)// a2 type: int
}
```

### **字符串和数值类型的相互转换**

#### string 与 int 类型之间的转换

- Itoa()：整型转字符串
    ```
    func Itoa(i int) string

    func main() {
        num := 100
        str := strconv.Itoa(num)
        fmt.Printf("type:%T value:%#v\n", str, str)//type:string value:"100"
    }
    ```

- Atoi()：字符串转整型
    ```
    func Atoi(s string) (i int, err error)

    Atoi() 函数有两个返回值，i 为转换成功的整型，err 转换失败时相应的错误信息。


    func main(){
        str1 := "100"
        str2 := "s100"

        num1, err := Atoi(str1)

        if (err != nil){
            fmt.Printf("%v 转换失败！", str1)
        } else {
            fmt.Printf("type:%T value:%#v\n", num1, num1)
        }

        num2, err := Atoi(str2)   

        if (err != nil){
            fmt.Printf("%v 转换失败！", str1)
        } else {
            fmt.Printf("type:%T value:%#v\n", num2, num2)
        }
    }

    //type:int value:110
    //s100 转换失败！
    ```

#### Parse 系列函数

Parse 系列函数用于将字符串转换为指定类型的值，其中包括 ParseBool()、ParseFloat()、ParseInt()、ParseUint()。

```

func ParseBool(str string) (value bool, err error)

func ParseInt(s string, base int, bitSize int) (i int64, err error)

func ParseUint(s string, base int, bitSize int) (n uint64, err error)

- bitSize 指定结果必须能无溢出赋值的整数类型，0、8、16、32、64 分别代表 int、int8、int16、int32、int64。
- base 指定进制，取值范围是 2 到 36。如果 base 为 0，则会从字符串前置判断，“0x”是 16 进制，“0”是 8 进制，否则是 10 进制。

func ParseFloat(s string, bitSize int) (f float64, err error)

- bitSize 表示参数 f 的来源类型（32 表示 float32、64 表示 float64），会据此进行舍入。

- 返回的 err 是 *NumErr 类型的，如果语法有误，err.Error = ErrSyntax，如果结果超出类型范围 err.Error = ErrRange。

```

#### Format 系列函数

Format 系列函数实现了将给定类型数据格式化为字符串类型的功能，其中包括 FormatBool()、FormatInt()、FormatUint()、FormatFloat()

```
func FormatBool(b bool) string

func FormatInt(i int64, base int) string

func FormatUint(i uint64, base int) string

- bitSize 指定结果必须能无溢出赋值的整数类型，0、8、16、32、64 分别代表 int、int8、int16、int32、int64。
- base 指定进制，取值范围是 2 到 36。如果 base 为 0，则会从字符串前置判断，“0x”是 16 进制，“0”是 8 进制，否则是 10 进制。

func FormatFloat(f float64, fmt byte, prec, bitSize int) string

- fmt 表示格式，可以设置为“f”表示 -ddd.dddd、“b”表示 -ddddp±ddd，指数为二进制、“e”表示 -d.dddde±dd 十进制指数、“E”表示 -d.ddddE±dd 十进制指数、“g”表示指数很大时用“e”格式，否则“f”格式、“G”表示指数很大时用“E”格式，否则“f”格式。
- prec 控制精度（排除指数部分）：当参数 fmt 为“f”、“e”、“E”时，它表示小数点后的数字个数；当参数 fmt 为“g”、“G”时，它控制总的数字个数。如果 prec 为 -1，则代表使用最少数量的、但又必需的数字来表示 f。
- bitSize 表示参数 f 的来源类型（32 表示 float32、64 表示 float64），会据此进行舍入。

```

#### Append 系列函数

Append 系列函数用于将指定类型转换成字符串后追加到一个切片中，其中包含 AppendBool()、AppendFloat()、AppendInt()、AppendUint()。

Append 系列函数和 Format 系列函数的使用方法类似，只不过是将转换后的结果追加到一个切片中。

