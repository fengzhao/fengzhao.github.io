---
layout:     post
title:      "Java设计模式之工厂模式"
subtitle:   " \"最常用的java设计模式.\""
date:       2016-10-04 16:00:00
author:     "FengZhao"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 设计模式
---
工厂模式作为最常用的java设计模式，在Java和Android的设计中，很多地方都有体现出来，相信对于经常调用各种API的朋友来说，对通过各种factory方法来生成一个对象的方法都非常熟悉了，今天我们就来研究一下这种设计模式。

<!-- more -->


## 工厂模式的种类

通过网络上的一些资料，我了解到工厂模式一般包括三种模式：
1. 简单工厂模式：
2. 工厂方法模式
3. 抽象工厂模式

## 简单工厂模式（SimpleFactory）

简单工厂模式（SimpleFactory）：定义一个工厂类，它可以根据参数的不同返回不同类的实例，被创建的实例通常都具有共同的父类。因为在简单工厂模式中用于创建实例的方法是静态(static)方法，因此简单工厂模式又被称为静态工厂方法(Static Factory Method)模式。

通过上面的定义，我们基本上也可以概括出它的实现思路：

1. 将要创建的各种对象封装到不同的类中，这些类被成为具体产品类。
2. 把具体产品的公共代码，进行抽象后封装到一个抽象类中，这个类被成为抽象产品类，每一个具体产品类都是抽象产品类的子类。
3. 创建一个工厂类，里面提供一个工厂方法用于创建各种产品，该方法可以通过传入的不同参数，来判断并返回不同的具体产品对象。
4. 客户端要想创建产品的时候，只需要通过调用工厂类中的工厂方法并传入相应的参数，即可返回相应的对象。

下面是一个简单的代码实现：

```java
//声明抽象产品类，在设计的时候，需要对具体产品类进行抽象，里面封装了各种产品对象的公有方法。
abstract class Product {  
    //所有产品类的公共业务方法  
    public void methodSame() {  
        //公共方法的实现  
    }  

    //声明抽象业务方法，每个类独有的实现，必须在子类中被实现  
    public abstract void methodDiff();  
}

//声明具体产品类A，每一个具体产品类都继承自抽象产品类，需要重写在父类中声明的抽象方法。
class ProductA extends Product {  
    //实现业务方法  
    public void methodDiff() {  
        //业务方法的实现  
    }  
  }
//声明一个具体产品类B
  class ProductB extends Product {  
        //实现业务方法  
        public void methodDiff() {  
            //业务方法的实现  
    }  
}
//工厂类，静态方法Product接收一个参数，根据传入不同的参数，来返回不同类型的对象。即不同的产品
public class ProductFactory{
      public static Product getProuduct(String args){
        switch(args)
        {
          case A:
              return new ProductA();
              break;

          case B:
              return  new ProductB();  
              break;
          default:break;
        }
  }

}
```
简单工厂模式的核心在于，可以通过类名直接调用其静态工厂方法，传入一个正确的参数，来返回需要的对象，在实际开发中，还可以将参数保存在xml格式的文件中，通过静态方法来读取，修改参数时无需直接修改源代码。

核心问题：通过这种模式，最大的问题在于工厂方法承担的职责过重，里面继承了所有常见的创建逻辑，二是可拓展性不高，试想如果要增加新的产品类，则必须要修改工厂方法，当产品类型较多时，逻辑过于复杂，难以维护。




## 工厂方法模式

工厂方法模式，是在简单工厂模式中，根据具体产品类，来定义多个工厂类，每个工厂类，只负责生产单一的具体产品对象。这也符合了单一职责原则。

工厂方法模式（Factory Pattern）：定义一个用于创建对象的接口，让子类决定将哪一个类实例化。工厂方法模式让一个类的实例化延迟到其子类。工厂方法模式又简称为工厂模式(Factory Pattern)，又可称作虚拟构造器模式(Virtual Constructor Pattern)或多态工厂模式(Polymorphic Factory Pattern)。

通过上面的定义，我们基本上也可以概括出它的实现思路：

1. 定义多个具体产品类，并抽象出其父类。
2. 定义多个具体工厂类，每一个工厂类负责生产对应的具体产品对象，定义抽象工厂类，里面声明工厂方法，具体的工厂方法，由继承自它的具体工厂类自己去实现。
3. 客户端通过调用不同的工厂方法，来返回不同的具体产品对象。

与简单工厂相比，工厂方法模式也增加了工厂类的抽象与继承层次结构，需要注意的一点是，工厂父类中的工厂方法，不能被声明为static，因为工厂方法实际都是由其子类去动态调用，所以这里需要注意。

下面是一个简单的工厂类代码示例，产品类代码不变：

```java
class abstract  Product{
    public abstract  Product();
}
class ProductA extends Product{
   private  String name;
   private  int age;
   public ProductA(String name){
     this.name=name;
   }
   public ProductA(int age){
     this.age=age;
   }
}
//抽象工厂类
 abstract class absFactory{
      public Product getProuduct(){

  }        
}

//抽象工厂类A，返回产品A
public  class ProductAFactory extends absFactory{
      public static  Product getProuduct(Sting name){
        return  new ProductA(name);      
      }        
   public static Product getProuduct(int age){
     return  new ProductA(age);
   }
}
```
通过代码可以发现，我们还可以重载工厂方法，来返回不同的对象，当然，对于一个具体工厂，只能返回同一个类型的产品对象。有的时候，我们还可以对客户端隐藏工厂方法，既在工厂方法中直接实例一个产品对象，并直接调用该对象的方法。

核心问题：在添加新的产品的时候，需要添加新的产品类和对应的工厂类，系统中类的个数双倍增长，会带来额外的开销。

## 抽象工厂模式

在工厂方法模式中，每一个具体的工厂类都返回一个具体产品对象，这样造成了巨大的浪费，我们想想，能否在在每一个工厂类中，返回多个产品呢，当然，这是可以的。这就是我们要讲的抽象工厂模式。

抽象工厂模式(Abstract Factory Pattern)：提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类。抽象工厂模式又称为Kit模式。

在抽象工厂模式中，每一个具体工厂都提供了多个工厂方法用于产生多种不同类型的产品，这些产品构成了一个产品族。与工厂方法模式中工厂类与产品类都是一对一的关系不同。抽象工厂是多对多的关系，笔者曾经尝试用文字来描述这种关系，最终宣告失败，下面我们用代码来描述一下它的具体实现方式：

```java
//抽象工厂
abstract class AbstractFactory {  
public abstract AbstractProductA createProductA(); //工厂方法A只生产A产品
public abstract AbstractProductB createProductB(); //工厂方法B只生产B产品
……  
}


//工厂1
class ConcreteFactory1 extends AbstractFactory {  
    //工厂1中的方法A，生产A1
    public AbstractProductA createProductA() {  
    return new ConcreteProductA1();  
  }  
    //工厂1中的方法B，生产B1
    public AbstractProductB createProductB() {  
    return new ConcreteProductB1();  
  }  
}
//工厂2
class ConcreteFactory2 extends AbstractFactory {  
    //工厂2中的方法A，生产A2
    public AbstractProductA createProductA() {  
    return new ConcreteProductA2();  
  }  
    //工厂2中的方法B，生产B2  
    public AbstractProductB createProductB() {  
    return new ConcreteProductB2();  
  }  
}
//抽象产品A
abstract class AbstractProductA{  
}
//由1工厂生产的产品A
class ConcreteProductA1 extends AbstractProductA {}
//由2工厂生产的产品A  
class ConcreteProductA2 extends AbstractProductA {}
//抽象产品B
abstract class AbstractProductB{  
}
//由1工厂生产的产品B
class ConcreteProductB1 extends AbstractProductB{}
//由2工厂生产的产品B
class ConcreteProductB2 extends AbstractProductB{}  
```

通过这段代码，相信你可以很快理解抽象工厂模式的使用方法，在抽象工厂模式中，最最重要的就是抽象工厂中的抽象方法，它的设计决定了整个代码的关键。
抽象工厂模式是工厂方法模式的进一步延伸，由于它提供了功能更为强大的工厂类并且具备较好的可扩展性，在软件开发中得以广泛应用，尤其是在一些框架和API类库的设计中，例如在Java语言的AWT（抽象窗口工具包）中就使用了抽象工厂模式，它使用抽象工厂模式来实现在不同的操作系统中应用程序呈现与所在操作系统一致的外观界面。抽象工厂模式也是在软件开发中最常用的设计模式之一。
