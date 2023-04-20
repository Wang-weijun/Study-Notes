# MVC设计模式：

M：Model 模型：一个功能。用 JavaBean实现。

V：View 视图：用于请求，以及与用户交互。使用html	js	css	jsp	jquery等前端

C：Controller 控制器：接受请求，将请求跳转到模型进行处理；模型处理完毕后，再将处理的结果返回给 请求处。可以用jsp实现，但一般建议使用Servlet实现控制器。



JSP->Java(Servlet)->JSP



![](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220405170256075.png)



![image-20220405171222546](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220405171222546.png)





# 三层架构：

与MVC设计模式的目标一致：都是为了 解耦合、提高代码复用；

区别：二者对项目理解的角度不同。



## 三层组成：

**表示层（USL，User Show Layer；视图层）**

​			-前台：对应于MVC中的View：用于和用户交互、界面的显示

​				jsp	js	html	css	jquery等web前端技术

​				代码位置：web

​			-后台：对应于MVC中Controller，用于 控制跳转、调用业务逻辑层

​				Servlet（SpringMVC  Struts2）

​				代码位置：一般位于xxx.servlet包中

**业务逻辑层（BLL，Business Logic Layer；Service层）**

​			-接受表示层的请求，调用

​			-组装数据访问层，逻辑性的操作（增删改查，删：查+删）

​			一般位于	xxx.service包（也可以成为：	xxx.manager，xxx.bll）

**数据访问层（DAL，Data Access Layer；Dao层）**

​			-直接访问数据库的操作，原子性的操作（增删改查）

​			一般位于	xxx.dao包



三层间的关系：

​	上层 将请求传递给下层，下层处理后返回给上层

​	上层依赖于下层，依赖：代码的理解，就是持有成员变量



![image-20220405172620891](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220405172620891.png)

数据库是由数据访问层去访问的。



## 三层优化：

### 1.加入接口

建议面向接口开发：先接口-再实现类

--service、dao加入接口

--接口与实现类的命名规范

​	接口：interface					起名	I实体类Service						IStudentService

​																IStudentDao

​	实现类：implements		   起名	实体类ServiceImpl				StudentServiceImpl

​																StudentDaoImpl

​	接口：I实体类层所在的包名			IStudentService、IStudentDao

​				接口所在的包：xxx.service	xx.dao

​	实现类：实体类层所在包名Impl     StudentServiceImpl、StudentDaoImpl

​				实现类所在的包：xxx.service.impl		xx.dao.impl

以后使用接口/实现类时，推荐写法：

接口 x = new 实现类（）；

```java
IStudentDao studentDao = new StudentDaoImpl();
```



### 2.DBUtil

通用的数据库帮助类，可以简化Dao层的代码量

帮助类 一般建议写在 xxx.util包







# MVC与三层架构的区别：



![image-20220405173616501](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220405173616501.png)