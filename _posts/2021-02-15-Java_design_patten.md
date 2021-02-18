---
layout: post
title:  "Java 设计模式"
date:   2021-02-16 21:18:54
categories: Java Design-patten
tags: Java
excerpt: Java 设计模式
mathjax: true
---

* content
{:toc}

> 为了看懂源码，先学习设计模式

## 单例模式

意图：保证一个类仅有一个实例，并提供一个访问它的全局访问点。

主要解决：一个全局使用的类频繁地创建与销毁。

何时使用：当您想控制实例数目，节省系统资源的时候。

如何解决：判断系统是否已经有这个单例，如果有则返回，如果没有则创建。

关键代码：构造函数是私有的。

应用实例：

1、一个班级只有一个班主任。
2、Windows 是多进程多线程的，在操作一个文件的时候，就不可避免地出现多个进程或线程同时操作一个文件的现象，所以所有文件的处理必须通过唯一的实例来进行。
3、一些设备管理器常常设计为单例模式，比如一个电脑有两台打印机，在输出的时候就要处理不能两台打印机打印同一个文件。
优点：

1、在内存里只有一个实例，减少了内存的开销，尤其是频繁的创建和销毁实例（比如管理学院首页页面缓存）。
2、避免对资源的多重占用（比如写文件操作）。
缺点：没有接口，不能继承，与单一职责原则冲突，一个类应该只关心内部逻辑，而不关心外面怎么样来实例化。

使用场景：

1、要求生产唯一序列号。
2、WEB 中的计数器，不用每次刷新都在数据库里加一次，用单例先缓存起来。
3、创建的一个对象需要消耗的资源过多，比如 I/O 与数据库的连接等。
注意事项：getInstance() 方法中需要使用同步锁 synchronized (Singleton.class) 防止多线程同时进入造成 instance 被多次实例化。

- 饿汉式
    ```
    /**
    /* 类加载到内存后，直接创建该类的实例，JVM 保证线程安全
    /* 简单实用
    /* 缺点： 不管用与否，实例在类加载时都会被创建
    */
    public class NewInstance {
        private static final NewInstance INSTANCE= new NewInstance();

        private Instance() {};

        public static NewInstance getInstance() {
            return INSTANCE;
        }
    }

    // 饿汉式的另外一种写法
    public class NewInstance {
        private static final NewInstance INSTANCE
        
        static {
            INSTANCE= new NewInstance();
        }

        private Instance() {};

        public static NewInstance getInstance() {
            return INSTANCE;
        }
    }
    ```

- 懒汉式
    ```
    /**
    /* 缺点：不是线程安全。
    */
    public class NewInstance {
        private static NewInstance INSTANCE;

        private Instance() {};

        public static NewInstance getInstance() {
            if (INSTANCE == null){
                INSTANCE = new NewInstance();
            }
            return INSTANCE;
        }
    }
    ```

    ```
    /**
    /* 线程安全的饿汉式
    */
    public class NewInstance {
        private static NewInstance INSTANCE;

        private Instance() {};

        public static sychronized NewInstance getInstance() {
            if (INSTANCE == null){
                INSTANCE = new NewInstance();
            }
            return INSTANCE;
        }
    }
    ```

    ```
    /**
    /* 线程不安全的饿汉式， 试图减少同步代码块来实现
    */
    public class NewInstance {
        private static NewInstance INSTANCE;

        private Instance() {};

        public static NewInstance getInstance() { 
            if (INSTANCE == null){
                sychronized (this) {
                    INSTANCE = new NewInstance();
                }  
            }
            return INSTANCE;
        }
    }
    ```

    ```
    /**
    /* 线程安全的饿汉式， 减少同步代码块来实现
    /* 双判断, 没必要
    */
    public class NewInstance {
        //volatile：防止JIT优化后，因为指令重排返回 NullpointException
        private static volatile NewInstance INSTANCE;

        private Instance() {};

        public static NewInstance getInstance() { 
            if (INSTANCE == null){
                sychronized (this) {
                    if (INSTANCE == null){
                        INSTANCE = new NewInstance();
                    }
                }  
            }
            return INSTANCE;
        }
    }
    ```
- 静态内部类的方式 - 懒汉式

    ```
    /**
    /* JVM保证线程安全
    /* 加载外部类时不会加载内部类，从而实现懒加载
    */
    public class NewInstance {

        private Instance() {};

        private static class NewInstanceInterval {
            private final static NewInstance INSTANCE = new NewInstance();
        }

        // 在这个方法调用的时候interval的类才会加载
        public static NewInstance getInstance() { 
            return NewInstanceInterval.INSTANCE;
        }
    }
    ```

- 枚举单例 完美的方法

    ```
    /**
    /* 不仅可以解决线程安全，还可以防止反序列化
    /* 枚举类没有构造方法，因此可以防止方序列化。
    */
    public enum NewInstance {

        INSTANCE;

        // 使用：

        public static void main(String[] args) {
            new Thread(() -> {
                System.out.println(NewInstance.INSTANCE.hashCode());
            }).start();
        }
    }
    ```
    
## 策略模式

在策略模式（Strategy Pattern）中，一个类的行为或其算法可以在运行时更改。这种类型的设计模式属于行为型模式。

主要解决：在有多种算法相似的情况下，使用 if...else 所带来的复杂和难以维护。

何时使用：一个系统有许多许多类，而区分它们的只是他们直接的行为。

如何解决：将这些算法封装成一个一个的类，任意地替换。

关键代码：实现同一个接口。

应用实例： 1、旅行的出游方式，选择骑自行车、坐汽车，每一种旅行方式都是一个策略。 2、JAVA AWT 中的 LayoutManager。3. 同一个类的不同比较算法和不同类的不同比较算法

优点： 1、算法可以自由切换。 2、避免使用多重条件判断。 3、扩展性良好。

缺点： 1、策略类会增多。 2、所有策略类都需要对外暴露。

使用场景： 1、如果在一个系统里面有许多类，它们之间的区别仅在于它们的行为，那么使用策略模式可以动态地让一个对象在许多行为中选择一种行为。 2、一个系统需要动态地在几种算法中选择一种。 3、如果一个对象有很多的行为，如果不用恰当的模式，这些行为就只好使用多重的条件选择语句来实现。

注意事项：如果一个系统的策略多于四个，就需要考虑使用混合模式，解决策略类膨胀的问题。

eg: 要比较不同的类。java提供comparaotr的接口，不需要自己写

```
pulic class Dog{
   int food;

   public Dog(int food) {
       this.food = food;
   }
```

```
pulic class DogComparator implements Comparor<Dog> {

   @Override
   public int compare(Dog o1, Dog o2) {
        if (o1.food < o2.food>){
            return -1;
        } else if (o1.food > o2.food>) {
            return 1;
        } else {
            return 0;
        }
   } 
}
```

```
pulic class Cat {
   int weight;
   int height;

   public Cat(int weight, int height) {
       this.weight = weight;
       this.height = height
   }
}
```

```
pulic class CatWeightComparator implements Comparor<Cat> {

   @Override
   public int compare(Cat o1, Cat o2) {
        if (o1.weight < o2.weight>){
            return -1;
        } else if (o1.weight > o2.weight>) {
            return 1;
        } else {
            return 0;
        }
   } 
}
```

```
pulic class CatHeightComparator implements Comparor<Cat> {

   @Override
   public int compare(Cat o1, Cat o2) {
        if (o1.height < o2.height>){
            return -1;
        } else if (o1.height > o2.height>) {
            return 1;
        } else {
            return 0;
        }
   } 
}
```

```
pulic class Sorter<T> {
    public void sort(T[] arr, Comparator<T> comparator){
        int minPos = -1;
        for (int i=0; i<arr.length -1; i++>){
            minPos = i;
            for (int j =i+1; j<arr.length; j++) {
                minPos = comparator.compare(arr[j], arr[minPos]) == -1 ? j : minPos;
            }
            swap(arr, i, minPos);
        }
    }

    void swap(T[] arr, int i, int minPos) {
        T temp = arr[i];
        arr[i] = arr[minPos];
        arr[minPos] = temp;
    }
}
```

```
pulic class Main {
    public static void main(String[] args){
        Dog[] a = {new Dog(5), new Dog(1), new Dog(3)};
        Sorter<Dog> sorter = new Sorter<>();
        souter.sort(a, new DogComparator());
    }
}
```

## 责任链模式

这种模式在结构上由多个部件、基于引用组成一串链条，行为上请求从链条头部的头部传递到各个节点，从而触发执行各个节点所需要的业务逻辑。

- 经典责任链模式：遍历链条上的节点，直到找到相对于的节点，然后处理。 - 结构上遍历数组。

- 变种责任链模式：各个节点依次处理，共享负担责任的一部分。- 引用组成链表。
