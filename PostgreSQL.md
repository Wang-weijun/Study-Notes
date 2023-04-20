# PostgreSQL

## 设置id自增

> 建表时，使用关键字 SERIAL

![图片](https://img2020.cnblogs.com/blog/2203909/202101/2203909-20210123153122643-2028585758.png)



如果使用Springboot中Mybatis-plus主键id自增异常

> **在实体类自增的id字段添加一个注解**

![图片](https://img-blog.csdnimg.cn/2019101816594674.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MzAxNzkz,size_16,color_FFFFFF,t_70)

AUTO(0, “数据库ID自增”),
INPUT(1, “用户输入ID”),
ID_WORKER(2, “全局唯一ID”),
UUID(3, “全局唯一ID”),
NONE(4, “该类型为未设置主键类型”),
ID_WORKER_STR(5, “字符串全局唯一ID”);





## 设置创建时间

![image-20230116192651217](C:\Users\Admin\Desktop\笔记\img\image-20230116192651217.png)

> 默认值： ('now'::text)::timestamp(0) with time zone



















