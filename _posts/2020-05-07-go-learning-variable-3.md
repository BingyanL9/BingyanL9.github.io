---
layout: post
title:  "Go - 变量（三）"
date:   2020-04-24 21:18:54
categories: Go Foundation
tags: Notes Go Variable Pointer
excerpt: Go变量 - 指针、变量逃逸分析等
mathjax: true
---

* content
{:toc}

> Go变量 - 指针、变量逃逸分析等

### **指针**

**指针:** 就是地址，指向内存地址空间，这个地址往往是在内存中存储的另一个变量的值的起始位置

**指针变量**：即存储地址的变量, 变量有时候被称为可寻址的值

Go 语言指针特点：

1. 任何类型的指针的零值都是nil

2. ```&```取变量地址, ```*```取目标对象的值

3. 类型指针的话允许对这个指针指向的值进行修改

4. 传递数据可以直接使用指针，而无须拷贝数据

5. 类型指针**不能进行偏移和运算**, 也就是说 Go 有指针，但没有指针运算

6. 切片，由指向起始元素的原始指针、元素数量和容量组成

eg:

```
func main(){
    a := 10
    b := &a
    fmt.Printf("a:%d ptr:%p\n", a, &a) // a:10 ptr:0xc00001a078
    fmt.Printf("b:%p type:%T\n", b, b) // b:0xc00001a078 type:*int
    fmt.Println(&b)                    // 0xc00000e018
    fmt.Println(*b)                    // 10
}
```

内存中图解：

![](\images\go-variable\pointer.png)

总结： 取地址操作符&和取值操作符```*```是一对互补操作符，```&```取出地址，```*```根据地址取出地址指向的值。

变量、指针地址、指针变量、取地址、取值的相互关系和特性如下：

- 对变量进行取地址```&```操作，可以获得这个变量的指针变量。

- 指针变量的值是指针地址

- 对指针变量进行取值```*```操作，可以获得指针变量指向的原变量的值

### **new 和 make**

Go语言中new和make是内建的两个函数，主要用来分配内存

- new 函数

    ```
    func new(Type) *Type

    Type: 表示类型，new函数只接受一个参数，这个参数是一个基本类型
    *Type: 表示类型指针，new函数返回一个指向该类型内存地址的指针
    ```

    创建变量的另外一种方式。

    表达式```new(T)```要创建一个T类型的匿名变量，初始化为T的类型零值，然后返回变量地址，返回类型为```*T```

    eg:

    ```
    func main(){
        a := new(int)
        b := new(bool)

        fmt.Printf("%T\n", a) // *int
        fmt.Printf("%T\n", b) // *bool
        fmt.Println(*a)       // 0
        fmt.Println(*b)       // false
    }
    ```

- make 函数

    make也是用于内存分配的，区别于 new ，它只用于 slice 、 map 以及 chan 的内存创建，而且它返回的类型就是这三个类型本身，本质来讲，导致这三个类型有所不同的原因是指向数据结构的引用在使用前必须被初始化。

    ```
    func make(t Type, size ...IntegerType) Type
    ```

    eg:

    ```
    func main(){
        var b map[string]int
        b = make(map[string]int, 10)

        b["a"] = 10
        fmt.Println(b)// map[a:10]
    }
    ```

- new 与 make 的区别

    1. 二者都是用来做内存分配的。
    2. make只用于slice、map以及channel的初始化，返回的还是这三个引用类型本身；
    3. 而new用于基本类型的内存分配，并且内存对应的值为类型零值，返回的是指向类型的指针。

### **逃逸分析**

#### 1. 关于堆和栈

栈由编译器进行管理，自动申请、分配、释放，一般不会太大，因此栈的分配和回收速度非常快。可以简单得理解成一次函数调用内部申请到的内存，函数返回直接释放，不会引起垃圾回收，对性能没有影响。

堆是内存的第二区域，除了栈之外，用来存储值的地方。堆适合不可预知大小的内存分配，这也意味着为此付出的代价是分配速度较慢，而且会形成内存碎片。堆无法像栈一样能自清理，使用这部分内存会造成很大的开销（相比于使用栈）。申请到堆上面的内存才会引起垃圾回收，如果这个过程（特指垃圾回收不断被触发）过于高频就会导致 gc 压力过大，程序性能出问题。

#### 2. 什么是逃逸分析

Go 没有像 C 语言那样提供精确的堆与栈分配控制，由于提供了内存自动管理的功能，很大程度上模糊了堆与栈的界限。例如以下代码：

```
package main

func main() {
    str := GetString()
    _ = str
}

func GetString() *string {
    var s string
    s = "hello"
    return &s
}
```
行 10 中的变量 ```s = "hello"``` 尽管声明在了 GetString() 函数内，但是在 main 函数中却仍然能够访问到返回的变量；这种在函数内定义的局部变量，能够突破自身的范围被外部访问的行为称作逃逸，也即通过逃逸将变量分配到堆上，能够跨边界进行数据共享。

逃逸分析技术就是为该场景而存在的；通过逃逸分析技术，编译器会在编译阶段对代码做了分析，当发现当前作用域的变量没有跨出函数范围，则会自动分配在**栈**上，反之则分配在**堆**上。

#### 3. 为什么需要逃逸分析

了解完堆和栈的之后，我们便可以更好的知道逃逸分析的目的了：

- 减少gc压力，栈上的变量，随着函数退出后系统直接回收，不需要gc标记后再清除。

- 因为逃逸分析完后可以确定哪些变量可以分配在栈上，栈的分配比堆快，性能好 (逃逸的局部变量会在堆上分配 ,而没有发生逃逸的则有编译器在栈上分配)。

- 同步消除，如果你定义的对象的方法上有同步锁，但在运行时，却只有一个线程在访问，此时逃逸分析后的机器码，会去掉同步锁运行。

#### 4. 如何逃逸分析

```
终端运行命令查看逃逸分析日志：
go build -gcflags=-m


go run -gcflags "-m -l" file.go

- m 会打印出逃逸分析的优化策略，实际上最多总共可以用 4 个 -m，但是信息量较大，一般用 1 个就可以了。
- l 会禁用函数内联，在这里禁用掉内联能更好的观察逃逸情况，减少干扰。
```

#### 5. 逃逸场景

- 指针逃逸: 返回局部变量指针

    ```
    package main

    type Student struct {
        Name string
        Age  int
    }

    func StudentRegister(name string, age int) *Student {
        s := new(Student) //局部变量逃逸到堆

        s.Name = name
        s.Age = age

        return s
    }

    func main() {
        StudentRegister("Jim", 18)
    }
    ```

- 栈空间不足逃逸（空间开辟过大）

    ```
    package main

    func Slice() {
        s := make([]int, 10000, 10000)//栈空间不足以存放当前对象时，逃逸

        for index, _ := range s {
            s[index] = index
        }
    }

    func main() {
        Slice()
    }
    ```

- 动态类型逃逸（不确定长度大小）

    很多函数参数为interface类型，比如fmt.Println(a …interface{})，编译期间很难确定其参数的具体类型，也能产生逃逸。

    ```
    package main

    import "fmt"

    func main() {
        s := "Escape"
        fmt.Println(s)
    }


    //.\main.go:7: s escapes to heap
    //.\main.go:7: main ... argument does not escape
    ```

- 闭包引用对象逃逸

    ```
    package main

    import "fmt"

    func Fibonacci() func() int {
        a, b := 0, 1
        return func() int {
            a, b = b, a+b
            return a
        }
    }

    func main() {
        f := Fibonacci()

        for i := 0; i < 10; i++ {
            fmt.Printf("Fibonacci: %d\n", f())
        }
    }

    //./main.go:7:9: can inline Fibonacci.func1
    //./main.go:7:9: func literal escapes to heap
    //./main.go:7:9: func literal escapes to heap
    //./main.go:8:10: &b escapes to heap
    //./main.go:6:5: moved to heap: b
    //./main.go:8:13: &a escapes to heap
    //./main.go:6:2: moved to heap: a
    //./main.go:17:34: f() escapes to heap
    //./main.go:17:13: main ... argument does not escape
    ```

#### 6. 逃逸总结

- 栈上分配内存比在堆中分配内存有更高的效率

- 栈上分配的内存不需要GC处理

- 堆上分配的内存使用完毕会交给GC处理

- 逃逸分析目的是决定内存分配地址是栈还是堆

- 逃逸分析在编译阶段完成


### 笔记来源

[Go 语言中文网：9.指针](https://studygolang.com/articles/27710)

[Go语言圣经:变量](https://docs.hacknode.org/gopl-zh/ch2/ch2-03.html)

[掘金:Go 内存逃逸详细分析](https://juejin.im/entry/5af532836fb9a07ac363847b)

[Go 语言中文网:Go 逃逸分析](https://studygolang.com/articles/21880)

[Gopherzhang:Golang内存分配逃逸分析](https://driverzhang.github.io/post/golang%E5%86%85%E5%AD%98%E5%88%86%E9%85%8D%E9%80%83%E9%80%B8%E5%88%86%E6%9E%90/)




