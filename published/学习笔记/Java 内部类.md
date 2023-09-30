---
description: "之前关于 Java 的内部类的具体使用，以及外部类和内部类的数据互访的问题上总有一些模糊，这里对 CSDN 上写的比较好的一篇文章做一个简单的记录。。"
time: 2022-11-21 16:32:55+08:00
---

之前关于 Java 的内部类的具体使用，以及外部类和内部类的数据互访的问题上总有一些模糊，这里对 CSDN 上写的比较好的一篇文章做一个简单的记录。Java 内部类可以分为 4 类：

* 普通内部类
* 静态内部类
* 匿名内部类
* 局部内部类

## 普通内部类

直接作为一个类的字段声明一个内部类。

```java
public class OuterClass {

    public class InnerClass {
        
    }
}
```

在这种定义方式下，普通内部类对象依赖外部类对象而存在，即在创建一个普通内部类对象时 **首先需要创建其外部类对象**，在外部类的函数中才可以创造内部类实例。  
这样生成的内部类对象就像是外部类对象的一个属性，可以访问外部类的 **所有访问权限** 的属性。

```java
// OuterClass.java
public class OuterClass {
    public int i;
    public OuterClass(int i) {
        this.i = i;
    }
    public class InnerClass {
        public void print() {
            System.out.println(i);
        }
    }
}

// Main.java
public class Main {
    public static void main(String[] args) {
        OuterClass o = new OuterClass(23);
        OuterClass.InnerClass i = o.new InnerClass();
        i.print();  // 23
    }
}
```
在上面这个例子中，Main 类可以通过先创建一个 OuterClass 对象，然后再通过这个对象创建一个 InnerClass 对象，最后通过这个 InnerClass 对象访问 OuterClass 对象的属性。要注意，这里的 InnerClass 对象必须依赖 OuterClass 对象而存在的，因此在创建 InnerClass 对象之前必须先创建 OuterClass 对象。  
同时，如果 InnerClass 的修饰符是 private 的话，那么在 Main 类中就无法创建 InnerClass 对象，如果 print 方法的修饰符是 private 的话，那么在 Main 类中可以是礼花 InnerClass，但无法调用 print 方法。  
这里的 InnerClass 对象就像是 OuterClass 对象的一个属性，可以访问 OuterClass 对象的所有访问权限的属性，但是 OuterClass 对象却无法访问 InnerClass 对象的属性，因为 InnerClass 对象的定义域只在 OuterClass 对象中，出了这个定义域，InnerClass 对象就不存在了，因此 OuterClass 对象无法访问 InnerClass 对象的属性。

```java


## 静态内部类

静态内部类作为一个外部类的静态成员而存在，创建一个类的静态内部类对象 **不需要依赖其外部类对象** 。但同时，静态内部类中也无法访问外部类的 **非静态成员**。

```java
public class OuterClass {

    /**
     * 静态内部类
     */
    static class InnerClass{
        private int number;
        public InnerClass(int num){
            this.number = num;
        }
        public void print(){
            System.out.println("num = " + this.number);
        }
    }

    public static void main(String[] args) {
        new InnerClass(1).print();
    }
}
```

## 匿名内部类

匿名内部类有多种形式，其中最常见的一种形式莫过于在方法参数中新建一个接口 / 类对象，并且实现这个接口 / 类中原有的方法了：

```java
public class OuterClass {

    public static void main(String[] args) {
        Square[] list = new Square[3];
        // 这里直接 new 一个接口并给出实现
        Arrays.sort(list, new Comparator<Square>() {
            @Override
            public int compare(Square o1, Square o2) {
                return 0;
            }
        });
    }
}
class Square {
    double width;
    double length;
}
```

上面的代码中展示了常见的两种使用匿名内部类的情况：  
1. 直接 new 一个接口，并实现这个接口声明的方法。  
   在这个过程其实会 **创建一个匿名内部类** 实现这个接口，并重写接口声明的方法，然后再创建一个这个匿名内部类的对象并赋值给声明的对象。
2. new 一个已经存在的类 / 抽象类，并且选择性的实现这个类中的一个或者多个非 final 的方法，这个过程会创建一个匿名内部类对象**继承对应的类 / 抽象类**，并且重写对应的方法。

在匿名内部类中可以使用外部类的属性，但是外部类却不能使用匿名内部类中定义的属性，因为是匿名内部类，因此在外部类中无法获取这个类的类名，也就无法得到属性信息。

## 局部内部类

局部内部类使用的比较少，其**声明在一个方法体 / 一段代码块的内部**，而且不在定义类的定义域之内便无法使用，其提供的功能使用匿名内部类都可以实现，而本身匿名内部类可以写得比它更简洁，因此局部内部类用的比较少。

```java
public class InnerClassTest {

    public int field1 = 1;
    protected int field2 = 2;
    int field3 = 3;
    private int field4 = 4;

    public InnerClassTest() {
        System.out.println("创建 " + this.getClass().getSimpleName() + " 对象");
    }
    
    private void localInnerClassTest() {
	    // 局部内部类 A，只能在当前方法中使用
        class A {
	        // static int field = 1; // 编译错误！局部内部类中不能定义 static 字段
            public A() {
	            System.out.println("创建 " + A.class.getSimpleName() + " 对象");
                System.out.println("其外部类的 field1 字段的值为: " + field1);
                System.out.println("其外部类的 field2 字段的值为: " + field2);
                System.out.println("其外部类的 field3 字段的值为: " + field3);
                System.out.println("其外部类的 field4 字段的值为: " + field4);
            }
        }
        A a = new A();
        if (true) {
	        // 局部内部类 B，只能在当前代码块中使用
            class B {
                public B() {
	                System.out.println("创建 " + B.class.getSimpleName() + " 对象");
                    System.out.println("其外部类的 field1 字段的值为: " + field1);
                    System.out.println("其外部类的 field2 字段的值为: " + field2);
                    System.out.println("其外部类的 field3 字段的值为: " + field3);
                    System.out.println("其外部类的 field4 字段的值为: " + field4);
                }
            }
            B b = new B();
        }
//        B b1 = new B(); // 编译错误！不在类 B 的定义域内，找不到类 B，
    }

    public static void main(String[] args) {
        InnerClassTest outObj = new InnerClassTest();
        outObj.localInnerClassTest();
    }
}
```

在局部内部类里面可以访问外部类对象的所有访问权限的字段，而外部类却不能访问局部内部类中定义的字段，因为局部内部类的定义只在其特定的方法体 / 代码块中有效，一旦出了这个定义域，那么其定义就失效了。

## 内部类的嵌套

内部类的嵌套，即内部类中再定义内部类，这个问题从内部类的分类角度去考虑比较合适：
* 普通内部类：  
  在这里我们可以把它看成一个外部类的普通成员方法，在其内部可以定义普通内部类（嵌套的普通内部类），但是无法定义 static 修饰的内部类，*就像你无法在成员方法中定义 static 类型的变量一样*，当然也可以定义匿名内部类和局部内部类；
* 静态内部类：  
  因为这个类独立于外部类对象而存在，我们完全可以将其拿出来，去掉修饰它的 static 关键字，他就是一个完整的类，因此在静态内部类内部可以定义普通内部类，也可以定义静态内部类，同时也可以定义 static 成员；
* 匿名内部类：  
  和普通内部类一样，定义的普通内部类只能在这个匿名内部类中使用，定义的局部内部类只能在对应定义域内使用；

* 局部内部类：  
  和匿名内部类一样，但是嵌套定义的内部类只能在对应定义域内使用。

## 参考

[CSDN：详解 Java 内部类](https://blog.csdn.net/Hacker_ZhiDian/article/details/82193100?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166901405816800182763306%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=166901405816800182763306&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-82193100-null-null.142^v66^control,201^v3^control_2,213^v2^t3_control2&utm_term=%E5%86%85%E9%83%A8%E7%B1%BB&spm=1018.2226.3001.4187)