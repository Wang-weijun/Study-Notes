# 一、JDBC原理及使用Statement访问数据库

## 1. JDBC

JDBC：Java DataBase Connectivity 可以为多种关系型数据库DBMS 提供统一的访问方式，用Java操作数据库

JDBC实现原理：Java通过JDBC操作JDBC DriverManager管理不同数据驱动。通过JDBC的**API**（包括接口、方法、类）



## 2.JDBC API 主要功能：

三件事，具体是通过以下类/接口实现：

DriverManager：管理jdbc驱动

Connection：连接（通过DriverManager产生）



Statement （PreparedStatement）：增删改查 （通过Connection产生）

CallableStatement：调用数据库中的 存储过程/ 存储函数 （通过Connection产生）

Statement、PreparedStatement、CallableStatement都是由Connection产生的



Result：返回的结果集 （上面的Statement等产生）



ResultSet：保存结果集 select * from xxx

next ()：光标下移，判断是否有下一条数据；true/false

previous ()：true/false

getXxx (字段值|位置)：获取具体的字段值



### 2.1 Connection产生操作数据库的对象：

Connection产生Statement对象：createStatement ()

Connection产生PreparedStatement对象：prepareStatement ()

Connection产生CallableStatement对象：prepareCall ()；



### 2.2 Statement操作数据库：

增删改：executeUpdate（）

查询：executeQuery（）；



### 2.3 PreparedStatement操作数据库：

public interface PreparedStatement extends Statement

因此

增删改：executeUpdate（）

查询：executeQuery（）；

赋值操作 setXxx（）；



### 2.4 PreparedStatement与Statement在使用时的区别：

1. Statement：

   sql

   executeUpdate（sql）

2. PreparedStatement:

   sql（可能存在占位符 ? ）

   在创建PreparedStatement 对象时，将sql预编译 prepareStatement(sql)

   executeUpdate（）

   setXxx（）替换占位符

   

推荐使用prepareStatement ：原因如下：

1. 编码更加简便（避免了字符串的拼接）

2. 提高性能（因为有预编译操作，预编译只执行一次）

3. 安全（可以有效防止sql注入）

   sql注入：将客户端输入的内容  和  开发人员的SQL语句  混为一体



### 2.5 CallableStatement：调用 存储过程、存储函数

connection.prepareCall（存储过程或存储函数名）

参数格式：

存储过程（无返回值return，用Out参数代替）：

​				{call	存储过程名（参数列表）}

存储函数（有返回值）：

​				{ ? = call	存储函数名（参数列表）}



### 2.5.1 存储过程

构造数据库存储过程

create or replace procedure addTwoNum ( num1 in number, num2 in number, result out number)

as

begin

​		result:=num1+num2;

end;

/

```java
cstmt = connection.prepareCall("call addTwoNum(?,?,?)");
cstmt.setInt(1, 10);
cstmt.setInt(2, 10);
cstmt.execute(); // num1 + num2,execute（）之前处理输入参数，之后处理输出参数
// 设置输出参数的类型
cstmt.registerOutParameter(3, Types.INTEGER);
int result = cstmt.getInt(3);// 获取计算结果
System.out.println(result);
```

JDBC调用存储过程的步骤：

a. 产生 调用存储过程的对象（CallableStatement）  cstmt = connection.prepareCall("call addTwoNum(?,?,?)");

b. 通过 setXxx（）处理 输出参数值cstmt.setInt（1，..）;

c. 通过 registerOutParameter（）;处理输出参数类型

d. cstmt.execute（）执行

e. 接收输出值（返回值）getXxx（）



### 2.5.2 存储函数

调用存储函数

create or replace function addTwoNumfunction ( num1 in number, num2 in number, result reutnr number)

return number

as

​		result number;

begin

​		result:=num1+num2;

​		return result;

end;

/

JDBC调用存储函数与调用存储过程的区别：

在调用时，注意参数 "? = call addTwoNumfunction(?,?)"



## 3. JDBC访问数据库的具体步骤：

a.  导入驱动，加载具体的驱动类

b. 与数据库建立连接

c. 发送sql，执行

d. 处理结果集（查询）



## 4. 数据库驱动						  			 

| 数据库驱动 | 驱动jar                    | 具体驱动类            | 连接字符串                               |
| :--------- | -------------------------- | --------------------- | ---------------------------------------- |
| MySQL      | mysql-connector-java-x.jar | com.mysql.jdbc.Driver | jdbc:mysql://localhost:3306/数据库实例名 |

使用jdbc操作数据库时，如果对数据库进行了更换，只需要替换：驱动、具体驱动类、连续字符串、用户名、密码



## 5. 用java实现对数据库的增删改、查

```java
public class JDBCDemo {
    private static final String URL = "jdbc:mysql://localhost:3306/bookdb";
    private static final String USERNAME = "root";
    private static final String PWD = "159357";
    public static void update() { // 增删改
        Connection connection = null;
        Statement stmt = null;
        try {
            //        a.  导入驱动，加载具体的驱动类
            Class.forName("com.mysql.cj.jdbc.Driver"); // 加载具体的驱动类
//        b. 与数据库建立连接
            connection = DriverManager.getConnection(URL, USERNAME, PWD);
//        c. 发送sql，执行（增删改、查）
            stmt = connection.createStatement();
//            String sql = "insert into user values(4,'df','26262')";
//            String sql = "update user set username='low' where id=3";
            String sql = "delete from user where id=4";
            // 执行SQL语句
            int count = stmt.executeUpdate(sql);// 返回值表示 增删改 几条数据
            // d.处理结果
            if (count > 0) {
                System.out.println("操作成功！");
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
            if (stmt != null) stmt.close();
                if (connection != null) connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

    public static void query() { // 查
        Connection connection = null;
        Statement stmt = null;
        ResultSet rs = null;
        try {
            //        a.  导入驱动，加载具体的驱动类
            Class.forName("com.mysql.cj.jdbc.Driver"); // 加载具体的驱动类
//        b. 与数据库建立连接
            connection = DriverManager.getConnection(URL, USERNAME, PWD);
//        c. 发送sql，执行（查）
            stmt = connection.createStatement();
            String sql = "select * from user";
            // 执行SQL语句(增删改executeUpdate（），查executeQuery（）)
             rs = stmt.executeQuery(sql);
            // d.处理结果
            while (rs.next()) {
                int id = rs.getInt("id");
                String username = rs.getString("username");
                String pwd = rs.getString("pwd");

//                int id = rs.getInt(1);    下标：从1开始计数
//                String username = rs.getString(2);
//                String pwd = rs.getString(3);
                System.out.println(id+"--"+username+"--"+pwd+"--");

            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (rs != null) rs.close();
                if (stmt != null) stmt.close();
                if (connection != null) connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
//        update();
        query();
    }
}
```



## 6. 总结JDBC（模板、八股文）

try{

a. 导入驱动包、加载具体驱动类 Class.forName（“具体驱动类”）

b. 与数据库建立连接 connection = DriverManager.getConnection(...);

c. 通过connection，获取操作数据库的对象（Statement\preparedStatement\callablestatement）

d.（查询）处理结果集 rs = pstmt.executeQuery（）

​	while(rs.next()) { rs.getXxx(...) };	

} catch (ClassNotFoundException e) {

} catch（SQLException e）{

} catch（Exception e）{

} finally {

​		// 打开顺序与关闭顺序相反

​		if (rs!= null)  rs.close();

​		if (stmt != null)  stmt.close();

​		if (connection!= null)  connection.close();

}

--jdbc中，除了Class.forName（）抛出ClassNotFoundException，其余方法全部抛SQLException 



## 7. 处理CLUB[Text]/BLOB类型

处理稍大型数据：

a. 存储路径 E:\

​		通过JDBC存储文件路径，然后根据IO操作处理

b. 

CLOB：大文本类型 （小说-> 数据）

BLOB：二进制



clob:

存

1. 先通过pstmt 的 ? 代替小说内容（占位符）
2. 再通过pstmt.setCharacterStream（2, reader , (int) file.length() ）;将上一步的 ? 替换为小说流，注意第三个参数需要是Int类型

取

1. 通过Reader reader = rs.getCharacterStream("NOVEL"); 将cloc类型的数据保存到 Reader对象中
2. 将Reader通过Writer输出即可。

blob与clob操作类似

把CharacterStream变为BinaryStream



# 二、JSP访问数据库

JSP就是在html中嵌套java代码，因此java代码可以写在jsp中（<%......%>）

导包操作：java项目：1. Jar复制到工程中 2. 右键该 Jar ：build path

​				   Web项目：jar复制到WEB-INF的lib里面

核心：将java代码中的JDBC代码，复制到 JSP 中的 <% ... %>



注意：如果jsp出现错误：The import Xxx cannot be resolved...

尝试解决步骤：

​						a. （可能是JDK、tomcat版本不一样）

​						b. 清空各种缓存

​						c. 删除之前的tomcat，重新配置tomcat、重启电脑等

​						d. 如果类之前没有包，则将该类加入包中

## 1. JavaBean

我们将jsp中登录操作的代码 转移到了LoginDao.java；其中LoginDao类 就称之为JavaBean。

JavaBean的作用：

​	a. 减轻了jsp的复杂度

​	b. 提高了代码的复用率（以后任何地方的 登录操作，都可以通过调用LoginDao实现）



JavaBean（就是一个类）的定义：满足以下两点，就可以称为JavaBean

​	a. public 修饰的类，public无参构造

​	b. 所有属性（如果有）都是private，提供set/get （如果boolean 则get可以替换 is）

使用层面，Java分为2大类：

​	a. 封装业务逻辑的JavaBean (Login.java 封装了登录逻辑)				---**逻辑**

​						可以将jsp中的JDBC代码，封装到Login.java中（Login.java）

​	b. 封装数据的JavaBean （实体类，Student.java Person.java）	---**数据**

​						对应于数据库中的一张表

​						Login login = new Login(username , upwd); 即用Login对象 封装了2个数据（用户名 和 密码）

封装数据的JavaBean 对应于数据库一张表（Login(name, pwd)）

封装业务逻辑的JavaBean用于操作 一个封装数据的JavaBean

可以发现使用，JavaBean可以简化 代码 （jsp->jsp+java）、提供代码复用（LoginDao.java）



# 三、MVC设计模式：

M：Model 模型：一个功能。用 JavaBean实现。

V：View 视图：用于请求，以及与用户交互。使用html	js	css	jsp	jquery等前端

C：Controller 控制器：接受请求，将请求跳转到模型进行处理；模型处理完毕后，再将处理的结果返回给 请求处。可以用jsp实现，但一般建议使用Servlet实现控制器。



JSP->Java(Servlet)->JSP



![](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220405170256075.png)



![image-20220405171222546](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220405171222546.png)



# 四、Servlet：

Java类必须符合一定的	规范：
	a. 必须继承	javax.servlet.http.HttpServlet

​	b. 重写其中的 doGet()或doPost()方法

doGet():	接受	并处	所有get提交方式的请求

doPost():  接受	并处	所有post提交方式的请求



Servlet要想使用，必须配置

Servlet2.5: web.xml

Servlet3.0: @WebServlet



Servlet2.5: web.xml	

项目的根目录：web、src

```jsp
<a href="WelcomeServlet">WelcomeServlet</a> 所在的jsp是在web目录中
```

因此	发出的请求WelcomeServlet	是去请求项目的根目录。



Servlet流程：

请求 -> <url-pattern> -> 根据 <servlet-mapping> 中的 <servlet-name> 去匹配 <servlet> 中的 <servlet-name> ，然后寻找到 <servlet-class>，最终将请求交由该 <servlet-class> 执行

```jsp
<servlet>
    <servlet-name>WelcomeServlet</servlet-name>
    <servlet-class>servlet.WelcomeServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>WelcomeServlet</servlet-name>
    <url-pattern>/WelcomeServlet</url-pattern>
</servlet-mapping>
```



Servlet3.0与Servlet2.5的区别：

Servlet3.0不需要在web.xml中配置，但需要在Servlet类的定义处之上编写	

注释	@WebServlet("url-pattern的值")

匹配流程：请求地址	与 @WebServlet中的值 进行匹配，如果匹配成功，则说明请求的就是该注解所对应的类



## 根路径：/：

项目根目录：web	src（所有的构建路径）

![image-20220327203200989](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220327203200989.png)

例如：在web中有一个文件index.jsp

src中有一个Servlet.java

如果：index.jsp中请求 <a href="abc” >...</a>，则 寻找范围：既会在src根目录中寻找 也会在web中寻找

如果：index.jsp中请求 <a href="a/abc” >...</a>，寻找范围：现在src或web中找a目录，然后再在a目录中找abc



/:

web.xml中的 / ：代表的是项目路径 http://localhost:8888/Servlet4Project_war_exploded/

jsp中的  / ：代表服务器根路径 http://localhost:8888/



构建路径、web：根目录



## Servlet生命周期：5个阶段

加载

初始化：init()，该方法会在Servlet被加载并实例化的以后 执行

服务：service() ->doGet()  doPost

销毁：destroy()，Servlet被系统回收时执行

卸载



init()：

​			a. 默认第一次访问 Servlet时会被执行 （只执行一次）

​			b. 可以修改为Tomcat启动时自动执行

​						Ⅰ.Servlet2.5: web.xml中

```jsp
<servlet>
    <servlet-name>a</servlet-name>
    <servlet-class>servlet.WelcomeServlet</servlet-class>
    <load-on-startup>1</load-on-startup>  其中'1'代表第一次
</servlet>
```

​						Ⅱ.Servlet3.0

​						@WebServlet(value="/WebcomeServlet" , loadOnStartup=1 )

service() -> doGet（） doPost（）：调用几次，则执行几次

destroy()：关闭tomcat服务时，执行一次。 



## Servlet API：

由两个软件包组成：对应于HPPT协议的软件包、对应于除了HPPT协议以外的其他软件包即Servlet API 可以适用于	任何	通信协议

我们学习的Servlet，是位于javax.servlet.http包中的类和接口，是基础HTTP协议。



## Servlet继承关系：

ServletConfig：接口

getServletContext（）：获取Servlet上下文对象	applicatio

String	getInitParameter（String name）：在当前Servlet范围内，获取名为name的参数值（初始化参数）



a. ServletContext中的常见方法（application）：

getContextPath()：相对路径

getRealPath()：绝对路径

setAttribute（）、getAttribute（）

--->

String  getInitParameter（String name）：在当前web范围内，获取名为name的参数值（初始化参数）



HttpServletRequest中的方法：（同request），例如setAttrite()、getCookies()、getMethod()

HttpServletResponse中的方法：同response



## 使用建议：

Servlet：一个Servlet对应一个功能，因此 如果有增删改查4个功能，则需要创建4个Servlet











# 五、三层架构：

与MVC设计模式的目标一致：都是为了 解耦合、提高代码复用；

区别：二者对项目理解的角度不同。



三层组成：

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





# 六、MVC与三层架构的区别：



![image-20220405173616501](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220405173616501.png)



# 七、Servlet中使用jsp功能

## out

out对象在servlet中使用

```java
PrintWriter out = response.getWriter();
```



## session

```java
request.getSession();
```



## application

```java
request.getServletContext();
```



# 八、分页SQL

## 1、分页

要实现分页，必须要知道  某一页的  数据 从哪里开始 到哪里结束

页面大小：每页显示的数据量



假设每页显示10条数据

sqlserver/oracle：从1开始计数

第n页					开始					结束

1							1						10

2							11					  20

3							21					  30

n							(n-1)×10+1		n×10



mysql：从0开始计数

0							0						9

1							10					  19

2							20					  29

n							n×10			   （n+1）×10-1

结论：

分页：	第n页的数据：	第(n-1)×10+1条	--	第n×10条



MySQL实现分页的sql：

limit	从第几页开始，每页显示多少条

第0页

select * from student limit 0, 10;

第1页

select * from student limit 10, 10;

第2页

select * from student limit 20, 10;

第n页

select * from student limit n×10, 10;

**MySQL的分页语句**

**select * from Student limit 页数×页面大小,页面大小;**



oracle分页：

sqlserver/oracle：从1开始计数

第n页					开始					结束

1							1						10

2							11					  20

3							21					  30

n							(n-1)×10+1		n×10

select * from student where sno >=(n-1)×10+1 and sno <= n×10;	--此种写法的前提：必须是Id值连续，否则无法显示

**oracle分页查询语句：**

select * from

(

select rownum r, t.* from (select s.* from student s order by sno asc) t

)

where r >(n-1)×10+1 and r <= n×10;

优化：

select * from

(

select rownum r, t.* from (select s.* from student s order by sno asc) t

where rownum <= n×10

)

where r >(n-1)×10+1 and r <= n×10;



**SQLServer分页：**

row_number() over(字段)



select * from

(

select row_number()  over (sno order by sno asc) as r, * from student

where  r <= n×10

)

where r >(n-1)×10+1 and r <= n×10;



SQLServer分页sql与oralce分页sql的区别：1.rownum, row_number()	2.oracle需要排序



## 分页实现：

  5个变量（属性）

1.数据总数		100		103															（查数据库，select count(*)...）

2.页面大小（每页显示的数据条数）20											（用户自定义）

3.总页数

​				总页数 = 100 / 20 = 数据总数 / 页面大小

​				页面数 = 103 / 20 = 数据总数 / 页面大小 + 1

​				-->

​				总页数 = 数据总数 % 页面大小 == 0 ？ 数据总数 / 页面大小：数据总数 / 页面大小+1；

4.当前页（页码）																					（用户自定义）

5.当前页的对象集合（实体类集合）：每页所显示的所有数据（10个人信息）

List<Student>																						（查数据库，分页sql）



```
// 分页帮助类
public class Page {
    // 当前页 currentPage
    private int currentPage;
    // 页面大小 pageSize
    private int pageSize;
    // 总数据 totalCount
    private int totalCount;
    // 总页数 totalPage
    private int totalPage;
    // 当前页的数据集合 students
    private List<Student> students;
    }
```



# 九、文件上传

## 1.上传

apache：从 https://mvnrepository.com 下载

commons-fileupload.jar组件

commons-fileupload.jar组件 依赖于 commons-io.jar



a.引入两个jar包

b.

代码：

前台jsp：

```jsp
<form action="UploadServlet" method="post" enctype="multipart/form-data">
上传照片：<input type="file" name="spicture"/>
```

​			表单提交方式必须是post

​			在表单中必须增加一个属性	enctype="multipart/form-data"

后台servlet：



注意的问题：

​				上传到目录	upload;

​				1.如果修改代码，则在tomcat重新启动时 会被删除

​								原因：当修改代码的时候，tomcat会重新编译一份class并且重新部署（重新创建各种目录）

​				2.如果不修改代码，则不会删除

​								原因：没有修改代码，class仍然是之前的class

因此，为了防止	上传目录丢失：a.虚拟路径	b.直接更换上传目录 到非tomcat目录



限制上传：类型、大小

类型：

```
String fileName = item.getName(); // a.txt  a.docx  a.png
String ext = fileName.substring(fileName.indexOf(".")+1);
if (!(ext.equals("png")||ext.equals("gif")||ext.equals("jpg"))) {
    System.out.println("图片类型有误！ 格式只能是png gif jpg");
    return; // 终止
}
```

进行判断文件后缀名，如果不是指定照片格式，则终止。



大小：

```
DiskFileItemFactory factory = new DiskFileItemFactory();
ServletFileUpload upload = new ServletFileUpload(factory);

// 设置文件上传时 用到的临时文件的大小（缓冲区）DiskFileItemFactory设置
factory.setSizeThreshold(10240); // 设置临时的缓存文件大小为10KB
factory.setRepository(new File("C:\\Users\\Wang\\Desktop\\uploadtemp")); // 设置临时文件的目录
// 控制上传单个文件的大小 20KB ServletFileUpload
upload.setSizeMax(20480); // 字节B

// 通过parseRequest解析form中的所有请求字段，并保存到item集合中（即前台传递到的sno sname spicture此时就保存在了item中）
                List<FileItem> items = upload.parseRequest(request);
```

注意：对文件的限制条件  写在 parseRequest 之前



## 2.下载

不需要依赖任何jar包

a.请求（地址a  form），请求Servlet	b.Servlet通过文件的地址 将文件转为输入流 读到Servlet中

c.通过输出流  将 刚才 已经转为输入流的文件	输出给用户

 

注意：下载文件	需要设置	两个响应头

```Java
// 下载文件：需要设置  消息头
response.addHeader("contentType", "application/octet-stream"); // MIME类型：二进制文件
response.addHeader("content-Disposition", "attachement;filename="+fileName); // fileName包含了文件后缀：abc.txt
```

![image-20220422000606168](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220422000606168.png)



下载时，文件名乱码问题：

Edge解决方法

```
URLEncoder.encode(fileName,"UTF-8")
```

FireFox、等其他浏览器需用if判断处理

```java
// 对于不同浏览器，进行不同的处理
// 获取客户端的User-Agent信息
String agent = request.getHeader("User-Agent");
if (agent.toLowerCase().indexOf("edge") != -1) {
    // Edge下载 文件乱码问题
    response.addHeader("content-Disposition", "attachment;filename="+ URLEncoder.encode(fileName,"UTF-8")); // fileName包含了文件后缀：abc.txt
}
```



# 十、EL表达式语言

EL：Expression Language，可以替代JSP页面中的 JAVA代码

servlet（增加数据）-》jsp（显示数据）

传统的 在JSP中用 java代码显示数据的弊端：类型转化、需要处理null、代码参杂 -》EL



## EL操作符：

**点操作符**	**.**										---使用方便

```
${requestScope.student.sno}
${域对象.域对象中的属性.属性.级联属性}
```

**中括号操作符**	**["  "]**	或	**[' ']	**	---功能强大：可以包含特殊字符（.	、-），获取变量值，获取数组的元素

```
${requestScope["student"]["sno"]}
${requestScope['student']['sno']}
```

当获取变量值时，不加引号，例如存在变量name，则可以用

```
${requestScope[name]}
```

可以获取数组的元素，例如定义一个数组String[] hobbies = new String[] {"football","pingpang","basketball"};

```
${requestScope.hobbies[0]}
${requestScope.hobbies[2]}
```



**获取map属性**

```
Map<String,Object> map = new HashMap<>();
map.put("cn", "中国");
map.put("us", "美国");
request.setAttribute("map", map);
```

**关系运算符	逻辑运算符**

```
${3>2}、${3 gt 2}<br/>				// 输出结果 true、true
${3>2 || 3<2 }、${3>2 or 3<2 }		// 输出结果 true、true
```

**Empty运算符**：判断一个值null、不存在 --> true

```
${empty requestScope["my-name"]}<br/>					 //输出结果	false
不存在的值：${empty requestScope["helloworld"]}<br/>		// 输出结果 true
为空的值：${empty requestScope.nullVal}<br/>				// 输出结果 true
```



## EL表达式的隐式对象： 

**隐式对象**：（不需要new	就能使用的对象，自带的对象）

a. 作用域的访问对象（EL域对象）：

pageScope、requestScope、sessionScope、applicationScope

如果不指定域对象，则会默认根据  从小到大的顺序	依次取值

b. 参数访问对象：获取表单数据（超链接中传的值、地址栏的值）

（request.getParameter()、			request.getParamterValues() ）

​			${param}											${paramValues}

c. JSP隐式对象：pageContext

在JSP中可以通过pageContext 获取其他的jsp隐式对象；

因此如果要在EL中使用JSP隐式对象，就可以通过pageCount间接获取

可以使用此方法，级联获取对象

```
${pageContext.request.serverPort}
```

**使用方法**：	${pageContext.方法名去掉() 和 get 并且将首字母小写}

例如要拿	getRequset( )		${pageContext.request}





# 十一、JSTL：

比EL更加强大，但需要两个jar包：jstl-1.2.jar、standard-1.1.2. jar

![image-20220509225302807](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220509225302807.png)

引入tablib:

```
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
```

其中prefix="c" ：前缀

核心标签库：通用标签库、条件标签库、迭代标签库

### **a. 通用标签库：**

**< c:set>** 赋值 

1. 在某个作用域之中（4个范围对象），给某个变量赋值

```
<%--
    request.setAttribute("name", "zhangsan");
--%>
<c:set var="name" value="zhangsan" scope="request" />
${requestScope.name}
```

<c:set var="变量名" value="变量值" scope="4个范围对象的作用域" />

2. 给普通对象赋值

   在某个作用域之中（4个范围对象），给某个对象的属性赋值（此种方法，不能指定scope属性）

​         <c:set target="${requestScope.student}" property="sname"  value="zxs" />

​		给map对象赋值

​		<c:set target="${requestScope.map}" property="cn" value="China" />

<c:set target="对象" property="对象的属性" value="赋值" />

注意 < c:set> 可以给不存在的变量赋值（但不能给不存在的对象赋值）



**< c:out>**

1. 显示（default="当value为空时，显示的默认值"）

```
传统EL：${requestScope.student} <br/>
c:out方式<c:out value="${requestScope.student}" />  <br/>
c:out显示不存在的数据：<c:out value="${requestScope.stu}" default="不存在" />  <br/>
```

2. 根据escapexml选择显示超链接方式

```
true:<c:out value='<a href="https://www.baidu.com">百度</a>' escapeXml="true" />
false:<c:out value='<a href="https://www.baidu.com">百度</a>' escapeXml="false" />
```

![image-20220510000959869](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220510000959869.png)

**< c:remove>**  删除属性

```
<c:remove var="a" scope="request" />
```



### **b. 条件标签库：选择**

if（boolean）

单重选择

< c:if test="" >

```
<c:if test="${10>2}" var="result" scope="request" >
    真
    ${requestScope.result}
</c:if>
```

多重选择

if else if...  esle if ....

< c:choose >

​					< c:when test="...">  < /c:when>

​					< c:when test="...">  < /c:when>

​					< c:otherwise>   < /c:otherwise>

< /c:choose >



**注意：在使用 test=“” 一定要注意是否有空格**

例如：test=“${10>2}”		true

​			test=“${10>2}  ”	  非true



### **c. 迭代标签库：循环**

for（int i=0; i<5; i++ ）

```
<c:forEach begin="0" end="5" step="1" varStatus="status">	// 0-5，varStatus显示下标次数
    ${status.index}	
    test...
</c:forEach>
```

![image-20220510225324021](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220510225324021.png)

for（String str : names）

```
<c:forEach var="name" items="${requestScope.names}" >
    ${name}-
</c:forEach>
```

![image-20220510231142989](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220510231142989.png)



# 十二、过滤器与监听器

## 过滤器：

实现一个Filter接口

init（）、destroy（）原理、执行时机同Servlet

配置过滤器，类似Servlet

通过doFilter（）处理拦截，并且通过	chain.doFilter(request, response);   放行请求



```
public class MyFilter implements Filter { // 过滤器
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("filter...init...");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        System.out.println("拦截请求。。。。。。");
        chain.doFilter(request, response); // 放行
        System.out.println("拦截响应。。。。。。");
    }

    @Override
    public void destroy() {
        System.out.println("filter...destroy...");
    }
}
```



filter映射

只拦截 访问Myservlet的请求

```
<url-pattern>/MyServlet</url-pattern>
```

拦截一切请求（每一次访问，都会被拦截）

```
<url-pattern>/*</url-pattern>
```



通配符

dispatcher请求方式：

REQUEST：拦截HTTP请求	get post						常用

FORWARD：只拦截  通过  请求转发方式的请求		常用

INCLUDE：只拦截通过 request.getRequestDispatcher("").include()、通过< jsp:include page="..."/>发出的请求

ERROR：只拦截< error-page>发出的请求



过滤器中doFilter方法参数：ServletRequest

在Servlet中的方法参数：HttpServletRequest



过滤器链

可以配置多个过滤器，过滤器的先后顺序 是由web.xml中<filter-mapping>的位置决定的





## 监听器：

开发步骤：

1. 编写监听器，实现接口
2. 配置web.xml



监听对象：request	session	application

a. 监听对象的创建和销毁

request：ServletRequestListener

session：HttpSessionListener

application：ServletContextListener

每个监听器	各自提供了2个方法：监听开始、监听结束的方法



b. 监听对象中属性的变更 

request：ServletRequestAttributeListener

session：HttpSessionAttributeListener

application：ServletContextAttributeListener

每个监听器	各自提供了3个方法：新增、替换、删除属性

例如：ServletRequest

```
@Override
public void attributeAdded(ServletRequestAttributeEvent srae) {
    String attrName = srae.getName(); // 目前正在操作的属性名
    Object attrValue = srae.getServletRequest().getAttribute(attrName);
    System.out.println("ServletRequest【增加】属性：属性名"+attrName+",属性值："+attrValue);
}

@Override
public void attributeRemoved(ServletRequestAttributeEvent srae) {
    System.out.println("ServletRequest【删除】属性：属性名"+srae.getName());
}

@Override
public void attributeReplaced(ServletRequestAttributeEvent srae) {
    String attrName = srae.getName();
    Object attrValue = srae.getServletRequest().getAttribute(attrName);
    System.out.println("ServletRequest【替换】属性：属性名"+attrName+",属性值："+attrValue);
}
```



## session对象的四种状态：绑定解绑、钝化活化

监听绑定与解绑：HttpSessionBindingListener										**不需要配置web.xml**

a. session.setAttribute（“a”,xxx） 对象a【绑定】到sesssion中

b. session.removeAttribute（“a”）将对象a从session中【解绑】 

监听session对象的钝化、活化：HttpSessionActivationListener			**不需要配置web.xml**

c. 钝化：内存-》硬盘

d. 活化：硬盘-》活化



如何钝化、活化：配置 tomcat安装目录/conf/context.xml

钝化、活化的本质  就是序列化、反序列化：序列化、反序列化需要实现Serializable



总结：钝化、活化 实际执行 是通过context.xml中进行配置 而进行。

​			活化：session中获取某一个对象时，如果该对象不存在，则直接尝试从之前钝化的文件中获取（活化）

​			HttpSessionActivationListener只是负责 在session钝化和活化时  予以监听.

​			需要实现Serializable





# 十三、Ajax，JQuery 与JSON 

## Ajax：

异步 js  和 xml

异步刷新:如果网页中某个地方需要修改，异步刷新可以使:只刷新需要修改的地方，而页面中其他地方保持不变。

例如：百度搜索框、视频的点赞

实现：

### js：XMLHttpReques对象

**XMLHttpReques对象的方法：**

open（方法名（提交方式get|post），服务器地址，true）：与服务端建立连接

send（）：

​					get：send（null）

​					post：send（参数值）

setRequestHeader（header,value）：

​					get：不需要设置此方法

​					post：需要设置：

​												a. 如果请求元素中包含了	文件上传：

​															setRequestHeader（"Content-Type","multiparty/form-data"）;

​												b. 不包含了	文件上传

​													setRequestHeader（"Content-Type","application/x-www-form-urlencoded"）;



**XMLHttpReques对象的属性：**

readyState：请求状态	只有状态为4  代表请求完毕

status：响应状态	只有200代表响应正常

onreadystatechange：回调函数

responseTest：响应格式为String

responseXML：响应格式为XML



## jquery：推荐

$.ajax（{

url：服务器地址，

请求方式：get|post

data：请求数据，

success：function（result,testStatus）

{

}，

error：function(xhr,errorMessage,e) {

}

}）;

```
<script type="text/javascript" src="js/jquery.js"></script>
<script type="text/javascript" >
    function register() {
        var $mobile = $("#mobile").val();
        $.ajax({
            url:"MobileServlet",
            请求方式:"post",
            data:"mobile="+$mobile,
            success:function (result,testStatus) {
                if (result == "true") {
                    alert("已存在!注册失败！");
                } else {
                    alert("注册成功！");
                }
            },
            error:function (xhr,errorMessage,e) {
                alert("系统异常！");
            }
        })
    }
</script>
```



$.get（

服务器地址，

请求数据，

function(result) {
},

预期返回值类型（string\xml）

）；



$.post（

服务器地址，

请求数据，

function(result) {
},

"xml" 或 "json" 或 "text"

）；







































# 十四、JNDI与tomcat连接池

## 1. JNDI：

java命名与目录接口



范围对象：4大作用域对象、4大范围对象

pageContext < request < session < application

pageContext：定义的变量有效期为当前页面，页面跳转后失效

request：一次请求有效

session：一次会话有效

application：一次项目运行期间都有效（项目打开后，只有不关闭，永远有效）



当我们需要多个项目都有效，除了数据库等，还可以使用 JDNI

JNDI：将某一个资源（对象），以配置文件（tomcat / conf / context.xml ）的形式写入；



在配置文件（tomcat / conf / context.xml ）中加入：

<Environment name="jndiName" value="jndiValue" type="java.lang.String" />

```java
String jndiName = "jndiValue";
```

String：写在type中，需要写包，java.lang.String

jndiName：元素名写在name中

jndiValue：赋的值写在value中



## 2. JNDI实现步骤：

tomcat / conf / context.xml 配置：

<Environment name="jndiName" value="jndiValue" type="java.lang.String" />



jsp中使用：

```jsp
<%
    Context ctx = new InitialContext();
    String textJndi = (String)ctx.lookup("java:comp/env/jndiName");

    out.print(textJndi);
%>
```

注意：java:comp/env/...     （固定格式）



## 3. 连接池

常见的连接池：Tomcat-dbcp、dbcp、c3p0、druid

可以用 **数据源** （javax.sql.DataSource）管理连接池	（数据源包含数据库连接池）



连接池：怎么用？

不用连接池

Class.forName();

Connection connection = DriverManager.getConnection();

使用连接池的核心：将连接的指向改了，现在指向的是数据源  而不是数据库

... ->  DataSource ds = .....

Connection connection = ds.getConnection;  // 指向的是数据源的连接







### Tomcat-dbcp连接池：

a. 类似jndi，在context.xml中配置数据库，加入以下代码



`<Resource name="student" auth="Container" type="javax.sql.DataSource" 
	maxActive="400" maxIdle="20" maxWait="5000" username="root" password="159357" 
	driverClassName="com.mysql.cj.jdbc.Driver" url="jdbc:mysql://localhost:3306/login" />`



![image-20220424173428003](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220424173428003.png)



b. 在项目web.xml中配置：

```jsp
<resource-ref>
    <res-ref-name>student</res-ref-name>
    <res-type>javax.sql.DataSource</res-type>
    <res-auth>Container</res-auth>
</resource-ref>
```



c. 使用数据源

​	更改 连接对象Connection的获取方式：

​	传统 JDBC 方式：

​	`connection = DriverManager.getConnection(URL, USER, PWD);`

​	数据源方式：

```java
Context ctx = new InitialContext(); // context.xml
ds = (DataSource)ctx.lookup("java:comp/env/student");
connection = ds.getConnection();
```



Tomcat - dbcp数据源总结：

1. 配置数据源（context.xml）
2. 指定数据源（web.xml）
3. 用数据源：通过数据源获取Connection



### dbcp连接池：

需要引入jar包：commons-dbcp-1.4.jar、commons-pool-1.6.jar

​							BasicDataSource、BasicDataSourceFactory

BasicDataSource方式：**硬编码方式**	BasicDataSource对象设置各种属性

![image-20220424215447606](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220424215447606.png)

```
// 获取dbcp方式的ds对象
public static DataSource getDataSourceWithDBCP() {
    BasicDataSource dbcp = new BasicDataSource();
    dbcp.setDriverClassName("com.mysql.cj.jdbc.Driver");
    dbcp.setUrl("jdbc:mysql://localhost:3306/login");
    dbcp.setUsername("root");
    dbcp.setPassword("159357");
    dbcp.setInitialSize(20);
    dbcp.setMaxActive(10);

    return dbcp;
}
```



BasicDataSourceFactory方式：**配置方式**（.properties文件，编写方式 key = value ）	常用这种
	dbcpconfig.properties

```
public static DataSource getDataSourceWithDBCPByProperties() throws Exception {
        DataSource dbcp = null;
        Properties props = new Properties();
        
        // 将文件变为数据流
        InputStream input = new DBCPDemo().getClass().getClassLoader().getResourceAsStream("dbcpconfig.properties");

        props.load(input); // 需要传入数据流

        // 只需要记住该行代码
        dbcp = BasicDataSourceFactory.createDataSource(props);
        return dbcp;
    }
```



DataSource	是所有sql是所有数据源的上级类。  BasicDataSource是dbcp类型的数据源

ComboPooledDataSource是c3p0的数据源；



### c3p0连接池：

核心类：ComboPooledDataSource

![image-20220426002313113](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220426002313113.png)

两种方式：硬编码与配置文件

-》合二为一，通过ComboPooledDataSource的构造方法参数区分：如果无参——硬编码；有参——配置文件

**jar包**：c3p0.jar

如果无参——硬编码

```
public static DataSource getDataSourceWithC3P0() throws PropertyVetoException {
    ComboPooledDataSource c3p0 = new ComboPooledDataSource();
    c3p0.setDriverClass("com.mysql.cj.jdbc.Driver");
    c3p0.setJdbcUrl("jdbc:mysql://localhost:3306/login");
    c3p0.setUser("root");
    c3p0.setPassword("159357");

    return c3p0;
}
```



有参——配置文件（c3p0-config.xml）文件

先写配置文件：

```
<?xml version="1.0" encoding="UTF-8"?>
<c3p0-config>
    <!-- 默认 -->
    <default-comfig>
        <property name="user">root</property>
        <property name="password">159357</property>
        <property name="driverClass">com.mysql.cj.jdbc.Driver</property>
        <property name="jdbcUrl">jdbc:mysql://localhost:3306/login</property>
        <property name="checkoutTimeout">30000</property>
    </default-comfig>
    
    <named-config name="jun">
        <property name="user">root</property>
        <property name="password">159357</property>
        <property name="driverClass">com.mysql.cj.jdbc.Driver</property>
        <property name="jdbcUrl">jdbc:mysql://localhost:3306/login</property>
    </named-config>
    
</c3p0-config>
```

在代码中传入参数，参数为<named-config name="jun">中的name:

```
public static DataSource getDataSourceWithC3P0ByXml() {
    ComboPooledDataSource c3p0 = new ComboPooledDataSource("jun");

    return c3p0;
}
```



### 总结：

所有连接池的思路：

1. 硬编码，某个连接池数据源的 对象 ds = new XxxDataSource();

   ds.setXxx();

   return ds;

2. 配置文件， ds = new XxxDataSource(); 加载配置文件，return ds;



数据源工具类：

​		可将dbcp、c3p0整合到一个类中，方便使用。

```
// 连接池帮助类
public class DataSourceUtil {

    // dbcp连接池
    public static DataSource getDataSourceWithDBCP() {
        BasicDataSource dbcp = new BasicDataSource();
        dbcp.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dbcp.setUrl("jdbc:mysql://localhost:3306/login");
        dbcp.setUsername("root");
        dbcp.setPassword("159357");
        dbcp.setInitialSize(20);
        dbcp.setMaxActive(10);

        return dbcp;
    }
    public static DataSource getDataSourceWithDBCPByProperties() throws Exception {
        DataSource dbcp = null;
        Properties props = new Properties();

        // 将文件变为数据流
        InputStream input = new DBCPDemo().getClass().getClassLoader().getResourceAsStream("dbcpconfig.properties");

        props.load(input); // 需要传入数据流

        // 只需要记住该行代码
        dbcp = BasicDataSourceFactory.createDataSource(props);
        return dbcp;
    }

    // c3p0连接池
    public static DataSource getDataSourceWithC3P0() throws PropertyVetoException {
        ComboPooledDataSource c3p0 = new ComboPooledDataSource();
        c3p0.setDriverClass("com.mysql.cj.jdbc.Driver");
        c3p0.setJdbcUrl("jdbc:mysql://localhost:3306/login");
        c3p0.setUser("root");
        c3p0.setPassword("159357");

        return c3p0;
    }

    public static DataSource getDataSourceWithC3P0ByXml() {
        ComboPooledDataSource c3p0 = new ComboPooledDataSource("jun");

        return c3p0;
    }
}
```

使用方法：

![image-20220426005106728](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220426005106728.png)



# 十五、ApacheDBUtil

下载commons-dbutils-1.7.jar，其中包含以下几个重点类：

DbUtils、QueryRunner、ResultSetHandler

## 1. DbUtils：辅助

包含了关闭连接、关闭结果、关闭提交、提交事务、传入驱动名，加载并注册JDBC驱动程序

## 2. QueryRunner：增删改、查

​	update()

​	query()



## 3. ResultSetHandler的实现类

如果是查询，则需要ResultSetHandler 接口，有很多实现类，一个实现类对应于一种 不同的查询结果类型

​		——Object[]	{1，zs}

​		实现类**ArrayHandler**：返回结果集中的**第一行**数据，并用Object[] 接收

​		实现类**ArrayListHandler**：返回结果集中的**多行**数据，List<Object[]>



​		——Student	(1，zs)

​		**BeanHandler**：返回结果集中的**第一行**数据，用对象（Student）接收

```
// 查询单行数据（放入对象中）
public static void testBeanHandler() throws SQLException {
    QueryRunner runner = new QueryRunner(DataSourceUtil.getDataSourceWithC3P0ByXml());
    Student Student = runner.query("select * from student where sno > ?", new BeanHandler<Student>(Student.class), 1);
    System.out.println(Student.getSno() + "," + Student.getSname());
}
```

反射会通过无参构造来创建对象

​	**BeanListHandler**：返回结果集中的**多行**数据，List<Student> students 接收

```
// 查询多行数据（放入对象中）
public static void testBeanListHandler() throws SQLException {
    QueryRunner runner = new QueryRunner(DataSourceUtil.getDataSourceWithC3P0ByXml());
    List<Student> students = runner.query("select * from student where sno > ?", new BeanListHandler<Student>(Student.class), 1);
    for (Student student:students) {
        System.out.println(student.getSno() + "," + student.getSname());
    }
}
```

​		**BeanMapHandler**：可以给每个数据加个key		1：stu ,  2：stu12 ， 3：stu3

```
// 查询单行数据（放入map对象中）
public static void testBeanMapHandler() throws SQLException {
    QueryRunner runner = new QueryRunner(DataSourceUtil.getDataSourceWithC3P0ByXml());
    Map<Integer, Student> studentMap = runner.query("select * from student where sno > ?", new BeanMapHandler<Integer, Student>(Student.class, "sno"), 1);
    Student student = studentMap.get(5);
    System.out.println(student.getSno() + "," + student.getSname());
}
```



​	——Map	查询的结果为 {sno=1，sname=zs }

1. **MapHandler**：返回结果集中的**第一行**数据

   {sno=1，sname=zs }

2. **MapListHandler**：返回结果集中的**多行**数据

   {{sno=1，sname=zs }	{sno=2，sname=ls }}

3. **KeyedHandler**： {zs={sno=1，sname=zs }，ls={sno=2，sname=ls }}



​	——

​	**ColumListHandler**：把结果集中的某一列  保存到List中

例如：	2，ls		3，ww

结果	{ls，ww}

​	**ScalarHandler**：单值结果集



问题：查询实现类的参数问题

query（...，Object...params ）

其中Object...params 代表可变参数：既可以写单值，也可以写数组



## 4. apache dbutil  增删改

增：

```
public static void add() throws PropertyVetoException, SQLException {
    QueryRunner runner = new QueryRunner(DataSourceUtil.getDataSourceWithC3P0());
    int count = runner.update("insert into student values(?,?,?,?)", new Object[]{11, "天天", 18, "北京"});
    System.out.println(count);
}
```

删：

```
public static void delete() throws PropertyVetoException, SQLException {
    QueryRunner runner = new QueryRunner(DataSourceUtil.getDataSourceWithC3P0());
    int count = runner.update("delete from student where sno = ? ", 10);
    System.out.println(count);
}
```

改：

```
public static void update() throws PropertyVetoException, SQLException {
    QueryRunner runner = new QueryRunner(DataSourceUtil.getDataSourceWithC3P0());
    int count = runner.update("update student set sno = ? where sname = ? ", new Object[]{10,"天天"});
    System.out.println(count);
}
```

自动提交事务	update（sql，参数）；update（sql）;

手动提交事务	update（connection，sql，参数）；



## 5. 手动提交事务

例如银行转账：zs-1000；li+1000；	需要一起提交

基础知识：

​				**①**如果既要保存数据安全，又要保证性能，可以考虑**ThreadLocal**

**ThreadLocal**：可以为每个线程  复制一个副本。每个线程可以访问自己内部的副本。 别名	线程本地变量

set（）：给tl中存放一个 变量

get（）：给tl中获取变量（副本）

remove（）：删除副本

​				**②**对于数据库来说，一个连接对应于一个事务，一个事务可以包含多个DML操作（增删改）

​				Service（多个原子操作）--》Dao（原子操作）

​				update：如果给每个 dao操作 都创建一个connection，则 多个dao操作对应于多个事务；

​								但是 一般来说，一个业务（Service）中的多个dao操作 应该包含在一个事务中。

​				——》**解决**：ThreadLocal，在第一次dao操作时，真正的创建一个connnection对象，然后

​							在其他几次dao操作时，借助于ThreadLocal本身特性，自动将该connection复制多个（connection只创建了一个，因此该connection中的所有操作，必然对应于同一个事务；并且ThreadLocal将

connection在使用层面复制了多个，因此可以同时完成多个dao操作）



事务流程：开始事务（将自动事务-->手动提交）-->进行各种DML-->正常，将刚才所有DML全部提交（全部成功）

​																							 -->失败（异常），将刚才所有DML全部回滚（全部失败）

事务的操作 全部和连接Connection密切相关





# 十六、元数据

元数据：描述数据的数据

<img src="C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220428234230282.png" alt="image-20220428234230282" style="zoom:67%;" />

三类：数据库元数据、参数元数据、结果集元数据

## 1. 数据库元数据：DataBaseMetaData

Connection——》DataBaseMetaData

```java
public static void databaseMetaData() {
    try {
        Class.forName(DRIVER);
        Connection connection = DriverManager.getConnection(URL, USER, PWD);
        // 数据库元信息
        DatabaseMetaData dbMetadata = connection.getMetaData();

        String dbName = dbMetadata.getDatabaseProductName();
        System.out.println("数据库名：" + dbName);

        String dbVersion = dbMetadata.getDatabaseProductVersion();
        System.out.println("数据库版本：" + dbVersion);

        String driverName = dbMetadata.getDriverName();
        System.out.println(driverName);

        String url = dbMetadata.getURL();
        System.out.println(url);

        String userName = dbMetadata.getUserName();
        System.out.println(userName);

        System.out.println("------------------");

        ResultSet rs = dbMetadata.getPrimaryKeys(null, "ROOT", "STUDENT");
        while (rs.next()) {
            Object tableName = rs.getObject(3);
            Object columnName = rs.getObject(4);
            System.out.println(tableName + "--" + columnName);
        }

    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



## 2. 参数元数据：ParameterMetaData

pstmt——》 ParameterMetaData

getParameterCount()：获取SQL语句中，参数占位符的个数

因为带参数的SQL是通过pstmt执行的，因此需要从pstmt中获取参数元数据相关信息

```
public static void parameterMetaData() {
    try {
        Class.forName(DRIVER);
        Connection connection = DriverManager.getConnection(URL, USER, PWD);
        String sql = "select * from student where sno = ? and sage = ?";
        PreparedStatement pstmt = connection.prepareStatement(sql);
        // 通过pstmt获取参数元数据
        ParameterMetaData metaData = pstmt.getParameterMetaData();

        int count = metaData.getParameterCount();
        System.out.println("参数个数：" + count);
        for (int i = 1; i <= count; i++) {
            String typeName = metaData.getParameterTypeName(i);
            System.out.println(typeName);
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



## 3. 结果集元数据：ResultSetMetaData

ResultSet——》 ResultSetMetaData

**Mysql**必须在 **url** 中附加参数配置：

**jdbc:mysql://localhost:3306/数据库名?generateSimpleParameterMetadata=true**

```java
private static final String URL = "jdbc:mysql://localhost:3306/login?generateSimpleParameterMetadata=true";
public static void resultSetMetaData() {
    try {
        Class.forName(DRIVER);
        Connection connection = DriverManager.getConnection(URL, USER, PWD);
        String sql = "select * from student";
        PreparedStatement pstmt = connection.prepareStatement(sql);
        ResultSet rs = pstmt.executeQuery();
        ResultSetMetaData metaData = rs.getMetaData();

        int count = metaData.getColumnCount();
        System.out.println("列的个数：" + count);
        System.out.println("-------------");

        for (int i=1;i<=count;i++) {
            String columnName = metaData.getColumnName(i);

            String columnTypeName = metaData.getColumnTypeName(i);
            System.out.println(columnName+"\t"+columnTypeName);
        }
        while (rs.next()) {
            for (int i = 1; i<=count; i++) {
                System.out.print(rs.getObject(i) + "\t");
            }
            System.out.println();
        }

    } catch (Exception e) {
        e.printStackTrace();
    }
}
```





# 十七、自定义标签

























# 验证码

防止恶意攻击

强制刷新：除了禁止缓存以外，还需要给服务端传递一个唯一的参数（没有实际用处）。随机数、时间

先通过jsp制作一个可以随机生成验证码图片

```
<%@ page import="java.awt.*" %>
<%@ page import="java.util.Random" %>
<%@ page import="java.awt.image.BufferedImage" %>
<%@ page import="javax.imageio.ImageIO" %>
<%@ page language="java" contentType="image/jpeg; charset=UTF-8" pageEncoding="UTF-8" %>

<%!
    // 随机产生颜色值
    public Color getColor() {
        Random ran = new Random();
        int r = ran.nextInt(256);// 产生0-255随机数
        int g = ran.nextInt(256);
        int b = ran.nextInt(256);
        return new Color(r,g,b); // red green blue 0-255
    }

    // 产生验证码值
    public String getNum() {
        // 0-8999   1000-9999
        int ran = (int)(Math.random() * 9000) + 1000;
        return String.valueOf(ran);
    }
%>

<%
    // 禁止缓存，防止验证码过期
    response.setHeader("Pragma", "no-cache");
    response.setHeader("Cache-control", "no-cache");
    response.setHeader("Expires", "0");

    // 绘制验证码
    BufferedImage image = new BufferedImage(80, 30, BufferedImage.TYPE_INT_RGB);
    // 画笔
    Graphics graphics = image.getGraphics();
    graphics.fillRect(0, 0, 80, 30);



    graphics.setFont(new Font("seif", Font.BOLD, 20));
    // 绘制验证码
    graphics.setColor(Color.BLACK);
    String checkCode = getNum();
    StringBuffer sb = new StringBuffer();
    for (int i=0;i<checkCode.length();i++) {
        sb.append(checkCode.charAt(i)+" "); // 验证码的每一位数字
    }

    graphics.drawString(sb.toString(), 15, 20); // 绘制验证码

    // 绘制干扰线条
    for (int i =0;i<30;i++) {
        Random ran = new Random();
        int xBegin = ran.nextInt(80);
        int yBegin = ran.nextInt(30);

        int xEnd = ran.nextInt(xBegin + 10);
        int yEnd = ran.nextInt(yBegin + 10);

        graphics.setColor(getColor());
        // 绘制线条
        graphics.drawLine(xBegin, yBegin, xEnd, yEnd);
    }

    // 将验证码真实值  保存在session中，供使用时比较真实性
    session.setAttribute("checkCode", checkCode);

    // 真实的产生图片
    ImageIO.write(image, "jpeg", response.getOutputStream());

    // 关闭
    out.clear();
    out = pageContext.pushBody();

%>
```

前台jsp:

```
<html>
    <head>
        <title>验证码</title>
        <script type="text/javascript" src="js/jquery.js"></script>
        <script type="text/javascript">

            function reloadCheckImg() {
                $("img").attr("src","img.jsp?t="+(new Date().getTime()));
            }

            $(document).ready(function () {
                $("#checkCodeId").blur(function () {
                    var $checkCode = $("#checkCodeId").val();
                    // 校验：文本框中输入的值 发送到服务端
                    // 服务端：获取文本框输入的值，和真实验证码图片中的值对比，并返回验证结果
                    $.post(
                        "CheckCodeServlet",
                        "checkCode="+$checkCode,
                        function (result) { // 图片地址（imgs/right.jpg imgs/wrong.jpg）
                            var resultHtml = $("<img src='"+result+"' height='15' width='15'>")

                            $("#tip").html(resultHtml);
                        }
                    );
                });

            });
        </script>

    </head>
    
    <body>
        验证码：
        <input type="text" name="checkCode" id="checkCodeId" size="4" />
        <%-- 验证码 --%>
        <a href="javascript:reloadCheckImg();"><img src="img.jsp" /></a>
        <span id="tip"></span>
    </body>
</html>
```

以及Servlet

```
public class CheckCodeServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String resultTip = "imgs/wrong.jpg";
        // 获取用户输入的验证码
        String checkCodeClient = request.getParameter("checkCode");

        // 真实的验证码值
        String checkCodeServer = (String) request.getSession().getAttribute("checkCode");
        if (checkCodeServer.equals(checkCodeClient)) {
            resultTip = "imgs/right.jpg";
        }
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter writer = response.getWriter();
        writer.write(resultTip);
        writer.flush();
        writer.close();
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }
}
```









































​	
