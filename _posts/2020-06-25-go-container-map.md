---
layout: post
title:  "Go - 容器 - 哈希表"
date:   2020-06-25 21:18:54
categories: Go Foundation
tags: Notes Go Map
excerpt: Go - 容器 - 哈希表
mathjax: true
---

* content
{:toc}

> map 是一种特殊的数据结构，一种元素对（pair）的无序集合，pair 对应一个 key（索引）和一个 value（值），所以这个结构也称为**关联数组或字典**，这是一种能够快速寻找值的理想结构，给定 key，就可以迅速找到对应的 value。

### 声明和初始化

1. 声明

    ```
    map 是引用类型，可以使用如下方式声明：
    var mapname map[keytype]valuetype

    [keytype] 和 valuetype 之间允许有空格。

    在声明的时候不需要知道 map 的长度，因为 map 是可以动态增长的，未初始化的 map 的值是 nil，使用函数 len() 可以获取 map 中 pair 的数目
    ```

2. 初始化

    - 字面量
        ```
        hash := map[string]int{
            "1":2,
            "3":4,
            "5":6,
        }
    ```

    - make
        ```
        和数组不同，map 可以根据新增的 key-value 动态的伸缩，因此它不存在固定长度或者最大限制，但是也可以选择标明 map 的初始容量 capacity
        make(map[keytype]valuetype, cap)

        make(map[string]float, 100)
        当 map 增长到容量上限的时候，如果再增加新的 key-value，map 的大小会自动加 1，所以出于性能的考虑，对于大的 map 或者会快速扩张的 map，即使只是大概知道容量，也最好先标明。

        ```

    - 使用切片作为map的值

        ```
        mp1 := make(map[int][]int)
        mp2 := make(map[int]*[]int)
        ```

### 遍历

```
//遍历键值对
iteratorMap := make(map[string]int)

iteratorMap["route"] = 66
iteratorMap["brazil"] = 4
iteratorMap["china"] = 960

for k,v := range iteratorMap {
    fmt.Println(k,v)
}

//route 66
//brazil 4
//china 960

//遍历值
for _,v := range iteratorMap {
    fmt.Println(v);
}

//4
//960
//66

//遍历键
for k := range iteratorMap {
    fmt.Println(k);
}

//china
//route
//brazil
```

**遍历输出元素的顺序与填充顺序无关，不能期望 map 在遍历时返回某种期望顺序的结果**, 如果需要特定顺序的遍历结果，正确的做法是先排序。

```
iteratorMap := make(map[string]int)

iteratorMap["route"] = 66
iteratorMap["brazil"] = 4
iteratorMap["china"] = 960

var sortList []string

// 将map数据遍历复制到切片中
for k := range iteratorMap {
    soutList = append(iteratorMap, k)
}

sout.Stirngs(soutList)

fmt.Println(sceneList) //[brazil china route]
```

### 删除和清空

#### 删除

Go语言提供了一个内置函数 delete()，用于删除容器内的元素。

```
//delete key from a map
delete(map, key)

iteratorMap := make(map[string]int)

iteratorMap["route"] = 66
iteratorMap["brazil"] = 4
iteratorMap["china"] = 960

delete(iteratorMap, "brazil")

for k,v := range iteratorMap {
    fmt.Println(k, v);
}

//route 66
//china 960
```

#### 清空

Go语言中并没有为 map 提供任何清空所有元素的函数、方法，清空 map 的唯一办法就是重新 make 一个新的 map，不用担心垃圾回收的效率，Go语言中的并行垃圾回收效率比写一个清空函数要高效的多。

### 多键索引 - 多个数值条件可以同时查询

```
// 人员档案
type Profile struct {
    Name    string   // 名字
    Age     int      // 年龄
    Married bool     // 已婚
}

//需求： name 和 age 作为组合键 去索引数据

// 查询键
type queryKey struct {
    Name string
    Age  int
}

//构建查询键到数据的索引
var mapper = make(map[queryKey] *Profile)

//构建查询索引
func buildIndex(list []*Profile) {

    for _, profile := range list {
        key := queryKey {
            Name : profile.Name
            Age : profile.Age
        }

        mapper(key, profile)
    }
}


//根据条件查询数据
func queryData(name string, age int){

    key := queryKey{name, age}

    // 根据键值查询数据
    result,status := mapper[key]

    if (status){
        fmt.Println(result)
    } else {
        fmt.Println("no found")
    }
}
```

**Notes:**
能够构建哈希值的类型必须是非动态类型、非指针、函数、闭包

- 非动态类型：可用数组，不能用切片

- 非指针：每个指针数值都不同，失去哈希意义

- 函数、闭包不能作为 map 的键

### sync.Map

**Go语言中的 map 在并发情况下，只读是线程安全的，同时读写是线程不安全的**

以下代码是会报错的：
```
m := make(map[int]int)

//开启一段并发代码
go func() {

    //不断对map进行写入
    for {
        m[1] = 1
    }
}()

go func() {
    
    //不断对map进行读取
    for {
        _ = m[1]
    }
}()

// 无限循环, 让并发程序在后台执行
for {
}

//运行报错
//fatal error: concurrent map read and map write
```
错误信息显示，并发的 map 读和 map 写，也就是说使用了两个并发函数不断地对 map 进行读和写而发生了竞态问题，map 内部会对这种并发操作进行检查并提前发现。

像这种并发的读写操作一般的做法是加锁，但是这样会影响性能。这个时候，就可以考虑使用 sync.Map 了，sync.Map 效率较高且并发安全。

#### sync.Map 特性：

- 无须初始化，直接声明即可

- sync.Map 不能使用 map 的方式进行取值和设置等操作，而是使用 sync.Map 的方法进行调用，**Store** 表示存储，**Load** 表示获取，**Delete** 表示删除

- 使用 Range 配合一个回调函数进行遍历操作，通过回调函数返回内部遍历出来的值，Range 参数中回调函数的返回值在需要继续迭代遍历时，返回 true，终止迭代遍历时，返回 false

#### 用例

```
import(
    "fmt"
    "sync"
)

func main() {

    var scene sync.Map

    //将键值对保存到sync.Map
    scene.Store("greece", 97)
    scene.Store("london", 100)
    scene.Store("egypt", 200)

    fmt.Println(scene.Load("london"))

    scene.Delete("london")

    scene.Range(func(k v interface{}) bool {

        fmt.Println("iterate:", k, v)
        return true
    })

    //100 true
    //iterate: egypt 200
    //iterate: greece 97
}
```

sync.Map 没有提供获取 map 数量的方法，替代方法是在获取 sync.Map 时遍历自行计算数量，sync.Map 为了保证并发安全有一些性能损失，因此在非并发情况下，使用 map 相比使用 sync.Map 会有更好的性能。

### 笔记来源

[C 语言中文网：Go语言map（Go语言映射）](http://c.biancheng.net/view/31.html)

[C 语言中文网:Go语言遍历map（访问map中的每一个键值对）](http://c.biancheng.net/view/32.html)

[C 语言中文网:Go语言map的多键索引——多个数值条件可以同时查询](http://c.biancheng.net/view/vip_7306.html)

[Go 语言中文网:Go语言sync.Map（在并发环境中使用的map）](http://c.biancheng.net/view/34.html)
