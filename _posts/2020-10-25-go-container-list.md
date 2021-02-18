---
layout: post
title:  "Go - 容器 - 列表"
date:   2020-10-25 21:18:54
categories: Go Foundation
tags: Notes Go List
excerpt: Go - 容器 - 列表
mathjax: true
---

* content
{:toc}

> 列表是一种非连续的存储容器，由多个节点组成，节点通过一些变量记录彼此之间的关系，列表有多种实现方法，如单链表、双链表等。 在Go语言中，列表使用 container/list 包来实现，内部的实现原理是双链表，列表能够高效地进行任意位置的元素插入和删除操作。

### **初始化列表**

list 的初始化有两种方法：分别是使用 New() 函数和 var 关键字声明，两种方法的初始化效果都是一致的。

1. 通过 container/list 包的 New() 函数初始化 list

    ```
    variable_name := list.New()
    ```

2. 通过 var 关键字声明初始化 list

    ```
    var variable_name list.List
    ```

    Note: 列表与切片和 map 不同的是，列表并没有具体元素类型的限制，因此，列表的元素可以是**任意类型**，这既带来了便利，也引来一些问题，例如给列表中放入了一个 interface{} 类型的值，取出值后，如果要将 interface{} 转换为其他类型将会发生宕机.

### **在列表中插入元素**

双链表支持从队列前方或后方插入元素，分别对应的方法是 PushFront 和 PushBack。这两个方法都会返回一个 *list.Element 结构，如果在以后的使用中需要删除插入的元素，则只能通过 *list.Element 配合 Remove() 方法进行删除，这种方法可以让删除更加效率化，同时也是双链表特性之一。

```
l := list.New()
l.PushFront(67)
l.PushBack("First")
```

同时也支持在某个元素之前或者之后插入元素：InsertAfter 和 InsertBefore

```
l := list.New()

element := l.PushFront(67)

l.InsertAfter("high", element)

l.InsertAfter("non", element)

for i := l.Front(); i != nil; i = i.Next() {
    fmt.Println(i.Value)
}

// non 67 high

```

### **从列表中删除元素**

列表插入函数的返回值会提供一个 *list.Element 结构，这个结构记录着列表元素的值以及节点之间的关系等信息，从列表中删除元素时，需要用到这个结构进行快速删除。

```
l := list.New()

element := l.PushFront(67)

l.Remove(element)
```

### **遍历列表**

遍历双链表需要配合 Front() 函数获取头元素，遍历时只要元素不为空就可以继续进行，每一次遍历都会调用元素的 Next() 函数

```
l := list.New()

l.PushBack("cannot")

l.PushFront(67)

for i := l.Front(); i != nil; i = i.Next() {
    fmt.Ptintln(i.Value)
}

// 67 cannot
```


