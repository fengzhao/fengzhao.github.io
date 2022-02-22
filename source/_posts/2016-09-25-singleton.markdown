---
layout:     post
title:      "Java设计模式之单例模式"
subtitle:   " \"the singleton pattern is a design pattern that restricts the instantiation of a class to one object.\""
date:       2016-09-25 14:00:00
author:     "FengZhao"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 设计模式
---
单例模式作为最简单的设计模式，也是面试中经常会被问到的，笔者在刚学java没多久的时候，就曾经在面试中被要求手写一个单例模式，于是写下这篇文章，来整理几种单例的写法。

<!-- more -->

## 什么是单例模式？

我们来看看维基百科的定义：

> 单例模式，也叫单子模式，是一种常用的软件设计模式。在应用这个模式时，单例对象的类必须保证只有一个实例存在。许多时候整个系统只需要拥有一个的全局对象，这样有利于我们协调系统整体的行为。比如在某个服务器程序中，该服务器的配置信息存放在一个文件中，这些配置数据由一个单例对象统一读取，然后服务进程中的其他对象再通过这个单例对象获取这些配置信息。这种方式简化了在复杂环境下的配置管理。

简单的概括一下：意思就是我们系统中在实例化一个类的时候，希望这个类的对象是唯一的，不可以重复实例化。

## 单例模式的设计思路

1.给要被实例化的类添加一个private构造函数，防止从外部通过new关键字来实例化该对象。
2.给该类定义一个私有静态变量，创建这个类的唯一实例。
3.在该类中定义一个静态方法，通过类名可以来获取该上面定义的实例。（必须是静态方法，通常使用getInstance这个名称）。

思考：为什么第2步中定义的变量必须是静态的呢？
因为我们在第三步是通过类的静态方法来返回这个变量，获取当前类的实例，如果不声明为静态变量，那么这个变量只有在当前类被实例化后才能被访问，我们就无法在第三步使用静态方法来访问这个变量，并返回了。

## 饿汉式static final field

饿汉式单例作为最简单的实现方式，通过上面的设计思路，我们可以很简单的设计出来。当初我被问到如何实现一个单例模式的时候，我第一反应是写了如下的代码，从Java编程思想中第六章中学到的一段代码。
```java
  public class Singleton{
    private static final Singleton instance= new Singleton();
    private Singleton (){}

    public static Singleton getInstance() {

     return instance;
    }

  }
```
当类被加载时，静态变量instance会被初始化，此时类的私有构造函数会被调用，单例类的唯一实例将被创建。缺点是当类加载时会自动实例化。



## 懒汉式，线程不安全

如何解决上面的自动加载问题呢？我们使用懒汉式单例，在定义静态变量的时候并不初始化。这种技术被成为懒加载技术(Lazy initialization)，只有在需要的时候，才会通过getInstance方法来加载实例。

```java
  public class Singleton{
    private static Singleton instance;
    private Singleton (){}

    public static Singleton getInstance() {
     if (instance == null) {
         instance = new Singleton();
     }
     return instance;
    }

  }
```
这段代码比较简单，但是存在致命问题，当有多个线程并行调用getInstance()的时候，还是会创建多个实例，也就是说在多线程下无法正常工作。

## 懒汉式，线程安全

为了解决上面的问题，避免多个线程同时调用getInstance方法，我们在getInstance方法前面增加一个关键字synchronized，来设置同步。

```java
public static synchronized Singleton getInstance() {
    if (instance == null) {
        instance = new Singleton();
    }
    return instance;
}
```
但是每次调用每次调用getInstance()时都需要进行线程锁定判断，在多线程高并发访问环境中，将会导致系统性能大大降低。

## 双重校验锁

双重检查锁定(Double-Check Locking)，是一种使用同步块加锁的方法。因为会有两次检查 instance == null，一次是在同步块外，一次是在同步块内。

```java
  public class Singleton {   
    //
    private volatile static Singleton instance = null;   

    private Singleton() { }   

    public static Singleton getInstance() {   
        //第一重判断  
        if (instance == null) {  
            //锁定代码块  
            synchronized (Singleton.class) {  
                //第二重判断  
                if (instance == null) {  
                    instance = new Singleton(); //创建单例实例  
                }  
            }  
        }  
        return instance;   
    }  
}
```

需要注意的是，如果使用双重检查锁定来实现懒汉式单例类，需要在静态成员变量instance之前增加修饰符volatile，被volatile修饰的成员变量可以确保多个线程都能够正确处理，且该代码只能在JDK 1.5及以上版本中才能正确执行。由于volatile关键字会屏蔽Java虚拟机所做的一些代码优化，可能会导致系统运行效率降低，因此即使使用双重检查锁定来实现单例模式也不是一种完美的实现方式。

## 静态内部类

懒汉式单例饿汉式单例类不能实现延迟加载，不管将来用不用始终占据内存；懒汉式单例类线程安全控制烦琐，而且性能受影响。我们使用一种更好的办法，被称之为Initialization Demand Holder (IoDH)的技术。这种方法使用内部类来实现，我们在单例类中增加一个静态(static)内部类，在该内部类中创建单例对象，再将该单例对象通过getInstance()方法返回给外部使用。

我比较倾向与使用静态内部类的方法，这种方法也是<<Effective Java>>上推荐的。

```java
//Initialization on Demand Holder  
class Singleton {  
    private Singleton() {  
    }  

    private static class HolderClass {  
            private final static Singleton instance = new Singleton();  
    }  

    public static Singleton getInstance() {  
        return HolderClass.instance;  
    }  
```

由于静态单例对象没有作为singleton的成员变量直接实例化，所以不会在类加载的时候实例化。所以它是懒汉式的，在第一次调用getInstance的时候，将加载内部类，在内部类中定义static类型的变量instance并初始化，由jvm来保证其线程安全，由于getInstance方法没有任何线程锁定，因此其性能不会造成任何影响。通过这种方法，我们既可以实现懒加载，又可以保证线程安全，所以我认为这是实现单例最方便简单的方式。

## 总结

单例模式作为一种目标明确、结构简单、理解容易的设计模式，在软件开发中使用频率相当高，在很多应用软件和框架中都得以广泛应用。
