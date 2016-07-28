title: 简要总结匿名内部类
date: 2015-02-25 09:15:01
tags: [技术,Java]
---

匿名内部类属于java的基础知识，只是我们平时使用几率并不太大。
最近我上手使用java8新特性，其中引入的lambda表达式对简化代码非常有利。而要深入理解java的lambda表达式，必须对匿名内部类有个清晰的认识，因此做个简要的复习。

### 内部类

首先从内部类说起。内部类即定义在类中的类。本文中把外部的类称作外部类，在其内部的类称作内部类。
为什么要使用内部类呢？在《Java核心技术》一书中概括了以下三点：
* 内部类方法可以访问该类定义所在的作用域中的数据，包括私有的数据。
* 内部类可以对同一包中的其他类隐藏起来。
* 当想要定义一个回调函数且不想编写大量代码时，使用匿名内部类比较便捷。

首先通过下面这段代码，演示内部类：

    //外部类
    class Outer {
      private String outStr = "hello outer";

      //内部类：定义在类中
      class Inner1{
        private String innerStr = "hello inner";
        void inner1Method() {
            System.out.println(outer);//内部类中可以直接使用外部类的变量
        }
      }

      void outerMethod() {
        //内部类：定义在方法中
        class Inner2{
            void inner2Method() {
                System.out.println(outer);
            }
        }
        //只能在方法中使用,在方法外无法调用
        Inner2 inner2 = new Inner2();
        inner2.innerMethod();
      }

      public static void main(String[] args){
        //外部类中可以调用inner1中的方法和变量
        Inner inner1 = new Inner1();
        System.out.println(inner1.innerStr);
        System.out.println(inner.inner1Method());
        inner.innerMethod();
        //无法调用Inner2,因为其定义在成员方法内
      }
    }
    //外部类之外的另一个类
    class Demo{
      void test() {
        //可以调用Inner1中的方法
        Outer.Inner1 inner1 = new Outer().new Inner1();
        inner1.inner1Method();
      }
    }
上面代码中展示了内部类的两种使用位置及相应的调用方法：

    1.外部类之内，成员方法之外：可以在外部类中使用，或外部类之外使用。

    2.外部类的成员方法的方法体中：只能在所在方法中使用，外部无法调用。

而除了这两种，还有第三种使用的位置，即：

    3.外部类的成员方法的参数中

第三种位置和匿名内部类放在一起来说。

### 匿名内部类

匿名内部类的目的是继承一个类或实现一个接口，直接实现其中的抽象方法，并创建实例，而不需要使用　class 类名{}　的形式去定义。

举例，首先定义一个接口：

    interface AnonyDemo{
      public abstract void demoInner();
    }

下面我们要实现这个接口，用什么方法呢？

我们先按正常方法实现：

    class AnonyDemoImpl implements AnonyDemo{
      @Override
      public void demoInner() {
          System.out.println("匿名内部类ok");
      }
    }

然后这样调用：

    AnonyDemo demo1 = new AnonyDemoImpl();
    demo1.demoInner();

很明显，这样写非常繁琐，下面改成使用匿名内部类。

匿名内部类的格式:

    new 类名/接口名(){
      覆盖类/接口中的代码，也可以自定义内容
    };

下面使用匿名内部类实现该接口，同时进行调用：

    new AnonyDemo(){
        public void demoInner() {
            System.out.println("匿名内部类ok");
        }
    }.demoInner();

这样明显比上一种方式简洁明了。

那么如果有这种情况，某个方法的参数类型是某个接口类型，比如：

    //规定参数类型为上面的AnonyDemo接口，方法内部使用了接口中的方法
    void outerMethod(AnonyDemo demo) {
        demo.demoInner();
    }

就可以在调用该方法的同时，实现AnonyDemo接口并实例：

    //放在成员方法的参数位置的匿名内部类。这里在参数中实现了AnonyDemo接口，并且赋予接口的demoInner方法以我们想要的功能。
    outerMethod(new AnonyDemo() {
        @Override
        public void demoInner() {
            System.out.println("参数的匿名内部类ok");
        }
    });
