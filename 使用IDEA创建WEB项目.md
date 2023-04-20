# 一、idea中Tomcat乱码：

a.	在idea	file - settings - 搜 File Encoding，改为utf-8

![image-20220424154219051](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220424154219051.png)



b.	打开IDEA工作目录，在idea64.exe.vmoptions和idea.exe.vmoptions最后追加

-Dfile.encoding=UTF-8



c.	配置Tomcat的页面中：VM options：添加	-Dfile.encoding=UTF-8

![image-20220424154623217](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220424154623217.png)

xxxxxxxxxx // 通用的数据操作方法public class DBUtil {    private static final String URL = "jdbc:mysql://localhost:3306/login";    private static final String USER = "root";    private static final String PWD = "159357";    public static Connection connection = null;    public static PreparedStatement pstmt = null;    static ResultSet rs = null;​    public static Connection getConnection() throws ClassNotFoundException, SQLException {        Class.forName("com.mysql.cj.jdbc.Driver");        return connection = DriverManager.getConnection(URL, USER, PWD);    }​    public static PreparedStatement createPreparedStatement(String sql, Object[] params) throws SQLException, ClassNotFoundException {        pstmt = getConnection().prepareStatement(sql);        if (params!=null) {            for (int i = 0; i < params.length; i++) {                pstmt.setObject(i + 1, params[i]);            }        }        return pstmt;    }        public static void closeAll(ResultSet rs,Statement stmt,Connection connection) {        try {            if (rs != null) rs.close();            if (pstmt != null) pstmt.close();            if (connection != null) connection.close();        } catch (SQLException e) {            e.printStackTrace();        }    }​    // 通用的增删改    public static boolean executeUpdate(String sql, Object[] params) {        try {            pstmt = createPreparedStatement(sql, params);            int count = DBUtil.pstmt.executeUpdate();            if (count > 0) {                return true;            } else {                return false;            }        } catch (ClassNotFoundException e) {            e.printStackTrace();            return false;        } catch (SQLException e) {            e.printStackTrace();            return false;        } catch (Exception e) {            e.printStackTrace();            return false;        } finally {            closeAll(null, pstmt, connection);        }    }​    // 通用的查:通用 表示 适合与 任何查询    public static ResultSet executeQuery(String sql,Object[] params) {        try {            pstmt = createPreparedStatement(sql, params);            rs = DBUtil.pstmt.executeQuery();            return rs;        } catch (ClassNotFoundException e) {            e.printStackTrace();            return null;        } catch (SQLException e) {            e.printStackTrace();            return null;        } catch (Exception e) {            e.printStackTrace();            return null;        }    }}java

## 终极解决办法

1. 打开tomcat的/conf/server.[xml](https://so.csdn.net/so/search?q=xml&spm=1001.2101.3001.7020)，给它显示的增加编码方式

2. 将日志的编码格式也修改一下，打开tomcat的\conf\logging.properties

   更改：java.util.logging.ConsoleHandler.encoding = GBK







# 二、IDEA热部署问题

更改代码，立即生效

![image-20220424154927026](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220424154927026.png)



idea：热部署，如果是run启动，仅JSP有效

​			如果是debug启动，java和jsp等静态资源均有效



总结：热部署

a.	更改idae为 update classes and resources

b.	以debug模式启动



注意：编写servlet前	需要先加入tomcat环境	

![image-20220424162122306](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20220424162122306.png)



# 三、IDEA中jar包问题

java项目：直接将jar复制到工程中，右键-Add as Library...



Web项目：IDEA会将web/lib/的jar，只在运行时生效，在其他阶段不生效。



解决方案：gradle/maven

​					手工解决：原理-结论

1. jar包本身只在 运行时有效

例如：jdbc.jar

处理办法：只需要将jar复制到web/lib/

（第一次需要新建lib文件夹）





​	2.jar包在各个阶段都有效

例如：commons-dbcp.jar（开发时、运行均有效）

手工解决：将commons-dbcp.jar在开发时也有效；直接将jar复制到工程的lib中，右键-Add as library...



总结论：

			1. java项目：直接复制到工程中
			1. web项目：同时添加到web/lib/中，以及复制在src中, 右键-Add as library...











