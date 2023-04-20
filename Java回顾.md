# Java回顾

从2022年12月20日开始





## Java基础





### JDK JRE JVM

**JDK**是Java Development Kit的缩写，是Java的开发工具包。

**JRE**（Java Runtime Environment，Java运行环境），包含JVM标准实现及Java核心类库。

**JVM**(Java Virtual Mechinal)，Java虚拟机，是JRE的一部分。

![1](https://img.php.cn/upload/image/327/663/584/1578906122900620.png)







### Java程序运行机制

编译型：C语言 、C++

解释型：网页、python、JavaScript



Java：编译加解释

![图片1](https://pic2.zhimg.com/80/v2-b0a738a35720507de04012242d8187b5_720w.webp)



### 数据类型

>  **强类型语言**：要求变量的使用要严格符合规定，所有变量都必须先定义后才能使用



Java的数据类型分为两大类：

- **基本类型**：整数类型（int byte short long）、浮点类型（float double）、字符类型（char）、boolean类型（true false）

- **引用类型**：类、接口、数组

![图片](http://c.biancheng.net/uploads/allimg/190909/5-1ZZZ91512493.jpg)

###  运算符  





### Scanner对象



~~~java
import java.util.Scanner; 
 
public class ScannerDemo {
    public static void main(String[] args) {
        Scanner scan = new Scanner(System.in);
        // 从键盘接收数据
 
        // next方式接收字符串
        System.out.println("next方式接收：");
        // 判断是否还有输入
        if (scan.hasNext()) {
            String str1 = scan.next();
            System.out.println("输入的数据为：" + str1);
        }
        scan.close();
    }
}
~~~

>next方式接收：
>runoob com
>输入的数据为：runoob



~~~java
import java.util.Scanner;
 
public class ScannerDemo {
    public static void main(String[] args) {
        Scanner scan = new Scanner(System.in);
        // 从键盘接收数据
 
        // nextLine方式接收字符串
        System.out.println("nextLine方式接收：");
        // 判断是否还有输入
        if (scan.hasNextLine()) {
            String str2 = scan.nextLine();
            System.out.println("输入的数据为：" + str2);
        }
        scan.close();
    }
}
~~~

> ```
> nextLine方式接收：
> runoob com
> 输入的数据为：runoob com
> ```

**next() 与 nextLine() 区别**

next():

- 1、一定要读取到有效字符后才可以结束输入。
- 2、对输入有效字符之前遇到的空白，next() 方法会自动将其去掉。
- 3、只有输入有效字符后才将其后面输入的空白作为分隔符或者结束符。
- next() 不能得到带有空格的字符串。

nextLine()：

- 1、以Enter为结束符,也就是说 nextLine()方法返回的是输入回车之前的所有字符。
- 2、可以获得空白。



### Java方法

原子性：一个方法只完成一个功能。



~~~java
public static 返回类型 方法名称([参数类型 变量, ......]) {
	方法体代码;
	[return [返回值];]
}
~~~



**方法的重载**

> 重载就是在一个类中，有相同的函数名称，但形参不同的函数。

方法重载的规则：

方法名称必须相同。

参数列表必须不同（个数不同、或类型不同、参数排列顺序不同等）。

方法返回类型可以相同也可以不相同。

仅仅返回类型不同不足以成为方法的重载。





## 面向对象

> 本质：以类的方式组织代码，以对象的组织（封装）数据。



抽象



### 三大特性：

+ 封装

- 继承

  ==super==注意点：

  	1. super调用父类的构造方法，必须在构造方法的第一个
  	1. super必须只能出现在子类的方法或者构造方法中！
  	1. super和this不能同时调用构造方法！

  Vs ==this==：

  ​	代表的对象不同：

  ​		this：本身调用者这个对象

  ​		super：代表父类对象的应用

  ​	前提

  ​		this：没有继承也可以使用

  ​		super：只能在继承条件才能使用

  ​	构造方法

  ​		this（）：本类的构造

  ​		super（）：父类的构造！

  重写：需要有继承关系，子类重写父类的方法！

      1. 方法名必须相同
      1. 参数列表必须相同
      1. 修饰符：范围可以扩大：public > protected > default > private
      1. 抛出的异常：范围，可以被缩小，但不能扩大； 

- 多态



### 接口





## 异常

![图片](C:\Users\Admin\Desktop\笔记\img\exception-hierarchy-1674230717608-3.png)

### Error（错误）

> 通常是灾难性的致命错误，是程序无法控制和处理的，当出现这种异常时，Java虚拟机（JVM）一般会选择终止程序



Error类对象由Java虚拟机生成并抛出，大多数错误与代码编写者所执行的操作无关。





### Exception（异常）

> 通常情况下是可以被程序处理的，并且在程序中应该尽可能的去处理这些异常



在Excepion分支中有一个重要的子类RuntimeException（运行时异常）

+ ArrayIndexOutOfBoundsException（数组下标越界）
+ NullPointerException（空指针异常）
+ ArithmeticException（算术异常）
+ MissingResourceException（丢失资源）
+ ClassNotFoundException（找不到类）



### 异常处理机制



**try catch  finally throw throws**









# GUI编程

GUI：图形用户接口

GUI核心技术：**Swing  AWT**



## AWT

包含了很多类和接口





## Swing







## 贪吃蛇

> 帧：如果时间片足够小，就是动画，一秒30帧







# 多线程





















