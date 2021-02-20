---
layout: post
title:  "Java classloader"
date:   2021-02-17 21:18:54
categories: Java
tags: Java
excerpt: Java classloader
mathjax: true
---

* content
{:toc}

> JAVA classloader

## 类的生命周期

加载 -> 连接（验证-> 准备 -> 解析）-> 初始化 -> 使用 -> 卸载

1. 加载 二进制字节流（.class）从磁盘读到内存。通过类全限定名称读取(package + class name eg:java.lang.String)

2. 连接

    · 验证： 验证字节码文件的正确性

    · 准备: 给类的静态变量分配内存并赋予默认值（未初始化）

    · 解析： 类装载器（user classloader -> application classloader -> extensions classloader -> bootstrap classloader）装入类所引用的其他所有类（静态链接）

3. 初始化

为类的静态变量赋予正确的初始值，真正初始化。 执行静态代码块。

4. 卸载

类结束生命周期：

    · 程序正常执行结束/异常结束

    · 操作系统异常，进程结束

    . system.exit()

## 类加载器

1. bootstrap classloader (启动类加载器)

加载JRE核心类库。 如JRE下的 rt.jar,charsets.jar

2. extensions classloader (扩展类加载器)

加载JRE扩展目录下(ext)中jar类包

3. application classloader (系统类加载器)

加载classPath路径下的类

4. user classloader (用户自定义加载器)

加载用户自定义路径下的类包

## 类加载机制

1. 全盘负责委托机制

当一个classloader加载一个类的时候，除非显示的使用另外一个classloader，该类所依赖和引用的类也有这个classloader加载。

在一个类里面引用了其他的类，其他类由该类加载器加载。

2. 双亲委派机制

双亲委派的意思是如果一个类加载器需要加载类，那么首先它会把这个类请求委派给父类加载器去完成，每一层都是如此。一直递归到顶层，当父加载器无法完成这个请求时，子类才会尝试去加载。这里的双亲其实就指的是父类，没有mother。

优势： 

    · 沙箱安全机制： 防止JRE核心类库被随意篡改

    · 避免类的重复加载： 当父classloader已经加载了该类的时候，子classloader就不需要再次加载了。