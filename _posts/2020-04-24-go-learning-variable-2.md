---
layout: post
title:  "Go - 变量（二）- 基础数据类型"
date:   2020-04-24 21:18:54
categories: Go Foundation
tags: Notes Go Variable
excerpt: Go变量 - 基础数据类型
mathjax: true
---

* content
{:toc}

> Go变量 - 基础数据类型

### **整型**

Go语言中有符号整数采用 2 的补码形式表示，也就是最高位用来表示符号位，一个 n 位的有符号数的取值范围是从 ```-2^(n-1)``` 到 ```2^(n-1)-1```。无符号整数的所有位都用于表示非负数，取值范围是 0 到 ```2^n-1```

1. 有符号整型：

    ```
    int8 int16 int32 int64

    int: 实际开发中由于编译器和计算机硬件的不同，int 所能表示的整数大小会在 32bit 或 64bit 之间变化
    ``` 

2. 无符号整型： 

    ```
    uint8 uint16 uint32 uint64 

    uint: 同int
    ```

3. uintptr: 无符号整型， 没有指定具体位数，但是足以容纳指针

### **浮点类型**

Go语言提供了两种精度的浮点数 float32 和 float64

这些浮点数类型的取值范围可以从很微小到很巨大。浮点数取值范围的极限值可以在 math 包中找到：

- 常量 math.MaxFloat32 表示 float32 能取到的最大数值，大约是 3.4e38；
- 常量 math.MaxFloat64 表示 float64 能取到的最大数值，大约是 1.8e308；
- float32 和 float64 能表示的最小值分别为 1.4e-45 和 4.9e-324。

一个 float32 类型的浮点数可以提供大约 6 个十进制数的精度，而 float64 则可以提供约 15 个十进制数的精度，通常应该优先使用 float64 类型，因为 float32 类型的累计计算误差很容易扩散，并且 float32 能精确表示的正整数并不是很大。

用 Printf 函数打印浮点数时可以使用```%f```来控制保留几位小数

### **复数**

复数的值由三部分组成 RE + IMi，其中 RE 是实数部分，IM 是虚数部分，RE 和 IM 均为 float 类型，而最后的 i 是虚数单位。

1. complex128 ： 64 位实数和虚数， 为复数的默认类型

2. complex64 : 32 位实数和虚数

声明如下：
```
var name complex128 = complex(x,y)

// complex 为Go语言的内置函数用于为复数赋值

// 可通过 real(z) 获取实部的值x

// 可通过 imag(z) 获取虚部的值y
```

可简写成：
```
name := complex(x,y)
```

复数也可以用```==```和```!=```进行相等比较，只有两个复数的实部和虚部都相等的时候它们才是相等的。

### **布尔类型**

一个布尔类型的值只有两种：true 或 false

Go语言对于值之间的比较有非常严格的限制，只有两个相同类型的值才可以进行比较，如果值的类型是接口（interface），那么它们也必须都实现了相同的接口。

如果其中一个值是常量，那么另外一个值可以不是常量，但是类型必须和该常量类型相同。

如果以上条件都不满足，则必须将其中一个值的类型转换为和另外一个值的类型相同之后才可以进行比较。

**Go语言中不允许将整型强制转换为布尔型**


### **字符串**

一个字符串是一个**不可改变**的字节序列，字符串可以包含任意的数据，但是通常是用来包含可读的文本，字符串是 UTF-8 字符的一个序列.

UTF-8 是一种被广泛使用的编码格式，是文本文件的标准编码，其中包括 XML 和 JSON 在内也都使用该编码。由于该编码对占用字节长度的不定性，在Go语言中字符串也可能根据需要占用 1 至 4 个字节，这与其它编程语言如 C++、Java 或者 Python 不同（Java 始终使用 2 个字节）。Go语言这样做不仅减少了内存和硬盘空间占用，同时也不用像其它语言那样需要对使用 UTF-8 字符集的文本进行编码和解码

字符串是一种值类型，且值不可变，即创建某个文本后将无法再次修改这个文本的内容，更深入地讲，字符串是字节的定长数组。

字符串所占的字节长度可以通过函数 len() 来获取，例如 len(str)。

获取字符串中某个字节的地址属于非法行为，例如 &str[i]。

- 定义多行字符串

    ```
    const str = ` first line
    second line
    third line
    \r\n
    `
    fmt.Println(str)
    ```

    运行结果：
    ```
    first line
    second lind
    third line
    \r\n
    ```

    在这种方式下，反引号间换行将被作为字符串中的换行，但是**所有的转义字符均无效**，文本将会原样输出。

- len()和RuneCountInString()

    len(): ASCII 字符串长度
    
    RuneCountInString(): Unicode 字符串长度 

    ```
    tip1 := "genji is a ninja"
    fmt.Ptintln(len(tip1))

    tip2 := "忍者"
    fmt.Println(len(tip2))
    ```

    程序输出结果：
    ```
    16
    6
    ```

    len() 函数的返回值的类型为 int，表示字符串的 ASCII 字符个数或字节长度。
    - 输出中第一行的 16 表示 tip1 的字符个数为 16。
    - 输出中第二行的 6 表示 tip2 的字节长度为 6。由于 Go 语言的字符串都以 UTF-8 格式保存，每个中文占用 3 个字节，因此使用 len() 获得两个中文文字对应的 6 个字节。

    ```
    fmt.Println(RuneCountInString("忍者"))
    fmt.Println(RuneCountInString("我是忍者,fight!"))

    //2
    //11
    ```

- 遍历字符串

    ASCII 字符串遍历直接使用下标。

    Unicode 字符串遍历用 for range。

    - 遍历每一个 ASCII 字符

        ```
        theme := "狙击 start"
        for i := 0; i < len(): i++ {
            fmt.Println("ascii: %c %d\n", theme[i], theme[i])
        }

        //ascii: ? 231
        //ascii:   139
        //ascii:   153
        //ascii: ? 229
        //ascii:   135
        //ascii: ? 187
        //ascii:   32
        //ascii: s 115
        //ascii: t 116
        //ascii: a 97
        //ascii: r 114
        //ascii: t 116
        ```

    - 按Unicode字符遍历字符串

        ```
        theme := "狙击 start"
        for _,s := range theme{
            fmt.Println("Unicode: %c %d\n", s, s)
        }
        
        //Unicode: 狙  29401
        //Unicode: 击  20987
        //Unicode:    32
        //Unicode: s  115
        //Unicode: t  116
        //Unicode: a  97
        //Unicode: r  114
        //Unicode: t  116
        ```

- 字符串截取

    字符串索引比较常用的有如下几种方法：
    - strings.Index：正向搜索子字符串。
    - strings.LastIndex：反向搜索子字符串。
    - 搜索的起始位置可以通过切片偏移制作

    ```
    tracer := "死神来了, 死神bye bye"
    comma := strings.Index(tracer, ", ") // 12

    pos := strings.Index(tracer[comma:], "死神")// 逗号为中文 占三个字符 因此输出的结果为 3

    fmt.Println(comma, pos, tracer[comma+pos:]) // 12 3 死神bye bye
    ```

- 修改字符串

    Go 语言的字符串是不可变的。
    修改字符串时，可以将字符串转换为 []byte 进行修改。
    []byte 和 string 可以通过强制类型转换互转。

    ```
    angel := "Heros never die"
    angleBytes := []byte(angel)
    for i := 5; i <= 10; i++ {
        angleBytes[i] = ' '
    }
    fmt.Println(string(angleBytes)) //Heros       die
    ```

- 高效字符串拼接

    连接字符串这么简单，还需要学吗？确实，Go 语言和大多数其他语言一样，使用```+```对字符串进行连接操作，非常直观。
    但问题来了，好的事物并非完美，简单的东西未必高效。除了加号连接字符串，Go 语言中也有类似于 StringBuilder 的机制来进行高效的字符串连接，例如：

    ```

    s1 := "first string"

    s2 := "second string"

    var stringBuilder bytes.Buffer// bytes.Buffer 是可以缓冲并可以往里面写入各种字节数组的

    stringBuilder.WriteString(s1)

    stringBuilder.WriteString(s2)

    fmt.Println(stringBuilder.String())// 将需要连接的字符串，通过调用 WriteString() 方法，写入 stringBuilder 中，然后再通过 stringBuilder.String() 方法将缓冲转换为字符串。
    ```

- Go语言fmt.Sprintf

    ```
    fmt.Sprintf(占位符, 参数列表…)

    占位符：字符串形式，格式化动词以%开头。
    参数列表：多个参数以逗号分隔，个数必须与格式化样式中的个数一一对应，否则运行时会报错。

    ```

    占位符：

    |占位符| 功能 |
    |----|------|
    |%v| 相应值的默认格式 |
    |%+v| 打印结构体时，会添加字段名 |
    |%#v| 相应值的Go语法表示  |
    |%T| 相应值的类型的Go语法表示  |
    |%%| 输出 % 本身|
    |%b| 整型以二进制方式显示 |
    |%o| 整型以八进制方式显示 |
    |%d| 整型以十进制方式显示 |
    |%x| 整型以十六进制显示 |
    |%X| 整型以十六进制显示，字母大写方式显示 |
    |%c| 相应Unicode码点所表示的字符|
    |%q| 单引号围绕的字符字面值，由Go语法安全地转义 or 双引号围绕的字符串，由Go语法安全地转义|
    |%U| Unicode 字符|
    |%f| 浮点数，有小数点而无指数 |
    |%b| 无小数部分的，指数为二的幂的科学计数法，与 strconv.FormatFloat 的 'b' 转换格式一致 |
    |%e| 科学计数法 |
    |%E| 科学计数法 |
    |%g| 根据情况选择 %e 或 %f 以产生更紧凑的（无末尾的0）|
    |%G| 根据情况选择 %E 或 %f 以产生更紧凑的（无末尾的0）|
    |%s| 输出字符串表示（string类型或[]byte) |
    |%p| 指针， 以十六进制方式显示|

    ```
    var progress = 2
    var target = 8
    // 两参数格式化
    title := fmt.Sprintf("已采集%d个药草, 还需要%d个完成任务", progress, target) 
    fmt.Println(title)// 已采集2个草药，还需要8个完成任务
    pi := 3.14159
    // 按数值本身的格式输出
    variant := fmt.Sprintf("%v %v %v", "月球基地", pi, true)
    fmt.Println(variant)// "月球基地" 3.14159 true 
    // 匿名结构体声明, 并赋予初值
    type Human struct {
        Name string
    }

    var people = Human{Name:"zhangsan"}
    fmt.Printf("使用'%%+v' %+v\n", people)// 使用%+v {Name:zhangsan}
    fmt.Printf("使用'%%#v' %#v\n", people)// 使用%#v main.Human{Name:"zhangsan"}
    fmt.Printf("使用'%%T' %T\n", people)// 使用%T main.Human

    fmt.Printf("%q", 0x4E2D)// '中'
    fmt.Printf("%e", 10.2)// 1.020000e+01
    fmt.Printf("%E", 10.2)// 1.020000e+01
    fmt.Printf("%g", 10.20)   //10.2
    fmt.Printf("%G", 10.20+2i) //(10.2+2i)

    fmt.Printf("%G", 10.20+2i) //(10.2+2i)
    ```

### **字符类型**

1. byte: 相当于 unit8 , 代表了 ASCII 码的一个字符。

2. rune：等价于 int32 类型, 代表一个 UTF-8 字符，当需要处理中文、日文或者其他复合字符时，则需要用到 rune 类型

3. Unicode 包中内置了一些用于测试字符的函数，这些函数的返回值都是一个布尔值，如下所示（其中 ch 代表字符）：

    - 判断是否为字母：unicode.IsLetter(ch)
    - 判断是否为数字：unicode.IsDigit(ch)
    - 判断是否为空白符号：unicode.IsSpace(ch)

### **数据类型转换**

在必要以及可行的情况下，一个类型的值可以被转换成另一种类型的值。由于Go语言不存在隐式类型转换，因此所有的类型转换都必须显式的声明：

```
valueOfTypeB = typeB(valueOfTypeA)

a := 5.0 

b := int(a) // 浮点数在转换为整型时，会将小数部分去掉，只保留整数部分。

fmt.Ptinrln("%d", b) // 5
```

类型转换只能在定义正确的情况下转换成功，例如:

- 支持从一个取值范围较小的类型转换到一个取值范围较大的类型（将 int16 转换为 int32）

- 当从一个取值范围较大的类型转换到取值范围较小的类型时（将 int32 转换为 int16 或将 float32 转换为 int），会发生精度丢失（截断）的情况。

- 只有相同底层类型的变量之间可以进行相互转换（如将 int16 类型转换成 int32 类型），不同底层类型的变量相互转换时会引发编译错误（如将 bool 类型转换为 int 类型）

### **UTF-8 和 Unicode 有何区别**

1. Unicode 与 ASCII 类似，都是一种字符集

    字符集为每个字符分配一个唯一的 ID，我们使用到的所有字符在 Unicode 字符集中都有一个唯一的 ID, 在不同国家的字符集中，字符所对应的 ID 也会不同.

2. UTF-8 是编码规则

    将 Unicode 中字符的 ID 以某种方式进行编码，UTF-8 的是一种变长编码规则，从 1 到 4 个字节不等。编码规则如下：

    - 0xxxxxx 表示文字符号 0～127，兼容 ASCII 字符集。

    - 从 128 到 0x10ffff 表示其他字符。

    - 根据这个规则，拉丁文语系的字符编码一般情况下每个字符占用一个字节，而中文每个字符占用 3 个字节

广义的 Unicode 指的是一个标准，它定义了字符集及编码规则，即 Unicode 字符集和 UTF-8、UTF-16 编码等。

### 笔记来源

[Go 语言中文网：golang fmt格式“占位符”](https://studygolang.com/articles/2644)

[C 语言中文网：go语言基本语法](http://c.biancheng.net/golang/syntax/)