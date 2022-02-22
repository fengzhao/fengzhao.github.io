---
layout:     post
title:      "Java注解"
subtitle:   " "
date:       2017-11-12 14:41:00
author:     "FengZhao"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Java
---
记录Java注解的学习过程，通过自己的语言描述我对注解的认识。
<!-- more -->

# 概述

很多文章都把注解定义为元数据，它提供程序数据，但不是作为程序本身的组成部分，注解不会直接影响他们所注解的代码的行为。实际上，他们是用来修饰代码的，Java语言中的类、方法、变量、参数和包等都可以被注解标注。和Javadoc不同，Java注解可以通过反射获取标注内容。在编译器生成类文件时，标注可以被嵌入到字节码中。Java虚拟机可以保留注解内容，在运行时可以获取到注解内容。当然它也支持自定义Java注解。

## 元注解
    
元注解实际上就是修饰注解的注解，他们是专门在定义注解的时候使用，在java.lang.annotation中定义，负责标注其他注解的一些属性，分别是：
 +  Retention   标记当前注解的生命周期
 +  Target  标记当前注解应该可以标记哪种Java成员。
 +  Documented  标记这些注解是否包含在用户文档中。  
 +  Inherited  标记这个注解是继承于哪个注解类(默认 注解并没有继承于任何子类)
 +  Repeatable  Java 8 开始支持，标识某注解可以在同一个声明上使用多次。

### Retention注解
Retention注解标注当前Annotation被保留的时间长短：某些注解仅出现在源代码中，而被编译器丢弃；而另一些却被编译在class文件中；编译在class文件中的Annotation可能会被虚拟机忽略，而另一些在class被装载时将被读取（请注意并不影响class的执行，因为Annotation与class在使用上是被分离的）。使用这个meta-Annotation可以对 Annotation的“生命周期”限制。

取值范围：取值（RetentionPoicy）有：
+ SOURCE:被标注的注解只保留到源码级，会被编译器忽略
+ CLASS:被标注的注解在编译时会被编译器识别，但会被Java虚拟机忽略。
+ RUNTIME:被标注的注解会被Java虚拟机识别，因此可用于运行时环境。

### Target注解

Target注解限制了其注解的注解可用于其他哪些Java元素，目标注解指定以下元素类型作为其值：

+ ANNOTATION_TYPE	注解类型声明
+ CONSTRUCTOR	构造方法声明
+ FIELD	字段声明（包括枚举常量）
+ LOCAL_VARIABLE	局部变量声明
+ METHOD	方法声明
+ PACKAGE	包声明
+ PARAMETER	参数声明
+ TYPE	类、接口（包括注解类型）或枚举声明

### Documented注解
Document注解表示只要它制定的注解的元素都应该用Javadoc工具文档化。

### Repeatable注解
Repeatable注解过的注解可多次用于同一个声明或类声明。




## 注解实例

看了这么多概念，下面上代码来理解一下。


### 声明注解

``` java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.ANNOTATION_TYPE,ElementType.METHOD,ElementType.TYPE,ElementType.FIELD})
public @interface AnnotationTest {
    String author() default "fengzhao";
    int age()default 18;
}
```
这段代码声明了一个AnnotationTest注解，限定了注解的声明周期与使用范围，声明注解与普通接口类似，在interface关键字前加上@符号就可以声明注解。这个注解有两个属性:autor和age,且设置了默认值。

### 使用注解

```java
@AnnotationTest(author = "fengzhaoadmin", age=13 )
public class AnnotationUse {

    @AnnotationTest
    public static void classAnnotationTEST(){
        Class cl =AnnotationUse.class;
        if(cl.isAnnotationPresent(AnnotationTest.class)){
            AnnotationTest at =(AnnotationTest) cl.getAnnotation(AnnotationTest.class);
            System.out.print(at.author());  //打印fengzhaoadmin
            System.out.print(at.age()); //打印13
        }
        else {
            System.out.print("this class isn't annotationed by AnnotationTest ");
        }
    }
    public static void methodAnnotationTEST(){
        try {
            Class cl =AnnotationUse.class;
            Method method = cl.getDeclaredMethod("classAnnotationTEST",(Class [])null );
            method.setAccessible(true);
          AnnotationTest at=(AnnotationTest)  method.getAnnotation(AnnotationTest.class);
            System.out.print(at.author());
            System.out.print(at.age());
        }
        catch (NoSuchMethodException e){
            e.printStackTrace();
            System.out.println(e.getMessage());
        }
    }

    public static  void main(String []argss){
        classAnnotationTEST();
        methodAnnotationTEST();
    }
}
```

在代码中，使用AnnotationTest去注解类，并设置其注解属性，在classAnnotationTEST中通过反射，打印其注解。继续使用该注解去注解classAnnotationTEST方法，在methodAnnotationTEST中通过反射，打印classAnnotationTEST方法的注解。



## 内置注解

Java已经内置了一些注解，我们平时经常会见到这些注解。

### Deprecated

这个注解是用来标记过时的元素，想必大家在日常开发中经常碰到。编译器在编译阶段遇到这个注解时会发出提醒警告，告诉开发者正在调用一个过时的元素比如过时的方法、过时的类、过时的成员变量。使用idea自动补全方法的时候，经常会看到一些方法被加上横线，这就是说明该方法已经被废弃了。 

```java
public class Person(){
    @Deprecated
    public void eating(){
        System.out.print("Eating");
    }
     public void drinking(){
        System.out.print("Dinking");
    }
}
```
在ide中调用Eating方法的时候，就会提示该方法被弃用。

### Ovveride
这个注解的作用是标识某一个方法是否覆盖了它的父类的方法。

```java
public class Fruit {

    public void displayName(){
        System.out.println("水果的名字是：*****");
    }
}

class Orange extends Fruit {
    @Override
    public void displayName(){
        System.out.println("水果的名字是：桔子");
    }
}

class Apple extends Fruit {
    @Override
    public void displayname(){
        System.out.println("水果的名字是：苹果");
    }
} 
```
Orange类编译不会有问题，Apple类在编译的时候，就会提示相关错误。


## SuppressWarnings

SuppressWarnings用来抑制编译器生成警告信息。可以修饰的元素为类，方法，方法参数，属性，局部变量，例如在泛型中使用原生数据类型。
它的定义如下：
```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}
```
该注解有方法value(）,可支持多个字符串参数，使用方法如下：
```java
@SupressWarning(value={"uncheck","deprecation"}).
```

# 参考

>  http://www.importnew.com/10294.html
> http://blog.csdn.net/briblue/article/details/73824058