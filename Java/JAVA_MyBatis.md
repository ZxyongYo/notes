# MyBatis

## 1 MyBatis概述



## 2 MyBatis 入门

### 2.1 MyBatis 实现步骤

- 步骤：

  1. 新建数据库，建一张表；

  2. 创建maven项目，加入Mybatis依赖和Mysql驱动的依赖；

  3. 创建实体类Student，保存表中的一行数据，

     [类中的实例变量与表中字段一 一对应，创建这些变量的setter，getter，重写toString]

  4. 创建持久层的DAO接口，定义操作数据库的方法，

     查询方法返回值类型一般是带有泛型的List，泛型类型为实体类 类型，

     insert方法返回值为int，表示修改数据库中记录的行数，参数为实体类对象；

  5. 创建Mybatis使用的xml配置文件（称为sql映射文件），一般一张表一个sql映射文件，

     此文件跟接口所在的文件同级，文件名与接口名保持一致；

     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE mapper
       PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
     <!--namespace: 命名空间  使用接口的全限定名称-->
     <mapper namespace="com.zxyong.dao.StudentDao">
       <!--
         id:要执行的sql语句唯一标识，mybatis会通过这个id找到要执行的sql语句
             应该使用对应的接口中的方法名
         resultType：结果集类型；是sql执行完之后得到的ResultSet，遍历这个ResultSet得到的java的类型
             值为类型的全限定名称
       -->
       <select id="selectStudents" resultType="com.zxyong.domain.Student">
         select * from t_student order by id
       </select>
     </mapper>
     ```

  6. resources目录下创建mybatis主配置文件，

     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE configuration
       PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-config.dtd">
     <configuration>
       <environments default="development">
         <environment id="development">
           <transactionManager type="JDBC"/>
           <dataSource type="POOLED">
             <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
             <property name="url" value="jdbc:mysql://localhost:3306/user?useSSL=false&amp;serverTimezone=UTC"/>
             <property name="username" value="root"/>
             <property name="password" value="123456"/>
           </dataSource>
         </environment>
       </environments>
       <mappers>
         <!-- 一个mapper指定一个sql mapper文件信息
            指的是从类路径开始的路径信息。即，target/classes/目录下开始的路径
        	-->
         <mapper resource="com\zxyong\dao\studentDao.xml"/>
       </mappers>
     </configuration>
     ```

  7. 在pom.xml中添加插件，正确编译配置文件

     ```xml
     <build>
         <resources>
           <resource>
             <directory>src/main/java</directory>
             <includes>
               <include>**/*.properties</include>
               <include>**/*.xml</include>
             </includes>
             <filtering>false</filtering>
           </resource>
         </resources>
       </build>
     ```
     
  8. 通过mybatis访问数据库
  
     ```java
     public class MyApp {
         public static void main(String[] args) throws IOException {
             // 1.定义mybatis主配置文件的名称,从类路径根目录开始,即target/classes之后
             String config = "mybatis.xml";
           // 2.读取主配置文件	org.apache.ibatis.io.Resources
             InputStream in = Resources.getResourceAsStream(config);
           // 3.创建sqlSessionFactoryBuilder对象
             SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
             // 4.创建sqlSessionFactory对象
             SqlSessionFactory factory = builder.build(in);
             // !5.获取sqlSession对象，从sqlSessionFactory中取出sqlSession
             SqlSession sqlSession = factory.openSession();
             // !6.指定要执行的sql语句的标识，sql映射文件中的namespace+”.“+标签的id值
             String sqlId = "com.zxyong.dao.StudentDao"+"."+"selectStudents";
             // 7.执行通过sqlId找的sql语句
             List<Student> studentList = sqlSession.selectList(sqlId);
             // 8.输出结果
             studentList.forEach(stu -> System.out.println(stu));
             // 9.关闭sqlSession对象
             sqlSession.close();
         }
     }
     ```
  
- 配置Mybatis输出日志，在Mybatis主配置文件中加入

  ```xml
  <!--settings:控制mybatis全局行为-->
    <settings>
      <!--输出日志-->
      <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
  ```


### 2.2 Mybatis中主要对象

- [org.apache.ibatis.io.] Resources：用来读取配置文件

- SqlSessionFactoryBuilder：用来创建SqlSessionFactory对象

- **SqlSessionFactory**：重量级对象（创建对象耗时长，占用资源多，整个项目中有一个就够用了）；

  是一个接口，实现类：DefaultSqlSessionFactory；用来获取sqlSession对象。

  方法：1. openSession() ：获取sqlSession对象，默认非自动提交事务，可以传一个Boolean用来表是 是否自动提交事务。

- **SqlSession**：接口，定义了操作数据库的方法。SqlSession不是线程安全的，需要在方法内部使用，

  使用之前通过 SqlSessionFactory的openSession获取，使用后通过它的close()方法关闭。

### 2.3 封装Mybatis工具类

```java
package com.zxyong.utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

public class MybatisUtils {
    private static SqlSessionFactory factory = null;
    // SqlSessionFactory 放在静态代码块中  只在类加载时创建一次
    static {
        String config = "mybatis.xml";
        try {
            InputStream in = Resources.getResourceAsStream(config);
            // 创建SqlSessionFactory对象
            factory = new SqlSessionFactoryBuilder().build(in);
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
    // 获取SqlSession的方法
    public static SqlSession getSqlSession(){
        SqlSession sqlSession = null;
        if(factory != null){
            sqlSession = factory.openSession();
            return sqlSession;
        }
        return sqlSession;
    }
}
```

## 3 使用Mybatis  Dao代理

### 3.1 Mybatis的dao动态代理实现

- 创建好Dao接口，并定义好操作数据库的抽象方法，

- Mybatis会根据Dao方法的调用，获取执行Sql语句的信息，Mybatis根据Dao接口自动创建实现类，并创建对象。完成sqlSession调用方法，访问数据库。

  ```java
  /**
  * 使用mybatis的动态代理机制，使用SqlSession。getMapper(dao接口)
  * getMapper能获取dao接口的实现类对象
  */
  @Test
  public void testSelect(){
      SqlSession sqlSession = MybatisUtils.getSqlSession();
      StudentDao dao = sqlSession.getMapper(StudentDao.class);
      List<Student> studentList = dao.selectStudents();
      studentList.forEach(student -> System.out.println(student));
      sqlSession.close();
  }
  ```


### 3.2 深入理解参数

#### 3.2.1 参数类型

- 在sqlMapper配置文件中通过 `parameterType`属性来定义接口方法的参数类型，

  因为Mybatis中的参数类型可以通过反射机制获得，parameterType属性可以不写。

#### 3.2.2 简单类型参数传递

- 在mapper文件中获取一个简单类型参数的值，使用#{xxx}

#### 3.2.3 多个参数传递

 - 第一种方式 **使用@Param命名参数**

   在Dao接口抽象方法中，参数前加注解@Param("自定义参数名")，

   ```java
   // 在mapper文件中使用自定义的参数名来获取参数
   List<Student> selectStudentBy(@Param("myId") Integer id, @Param("myName") String name);
   ```

- 第二种方式 **使用java对象传参**

  ```xml
  <!--
          #{属性名, [javaType=Java对象属性的数据类型, jdbcType=在数据库中的数据类型]}
          javaType和jdbcType的值mybatis通过反射可以获取  可以写
      -->
      <select id="selectByObject" resultType="com.zxyong.domain.Student">
          select * from t_student where
          id=#{paramId, javaType=java.lang.Integer, jdbcType=INTEGER} or
          name=#{paramName, javaType=java.lang.String, jdbcType=VARCHAR}
      </select>
  ```

- 第三种方式 **按位置传参**（了解）
- 第四种方式 **使用Map传值**（了解）

#### 3.2.4 '$' 和 '#' 占位符

- $：使用statement来执行sql；多用来做表明或列名的替换；
- #：使用PreparedStatement来执行sql，更安全 建议使用；

### 3.3 封装Mybatis输出结果

#### 3.3.1 resultType

- 处理方式：
  1. 执行完sql语句，调用resultType的值代表的类的无参构造器；
  2. 将每条数据的所有字段按字段名调用同名属性的setter方法，然后放入List；

- 返回数据转成的java对象是任意的，不一定是实体类；

- 自定义别名     （建议尽量使用全限定名称，不用别名）

  ```xml
  <!--Mybatis主配置文件添加-->
  <typeAliases>
      <!-- 第一种方式
              type:自定义类的全限定名称
              alias：别名-->
      <!--        <typeAlias type="com.zxyong.domain.Student" alias="stu" />-->
      <!--第二种方式
              name是包名，这个包种的所有类，类名就是别名（不区分大小写）-->
      <package name="com.zxyong.domain"/>
  </typeAliases>
  ```

- 返回类型是Map的时候，只能返回一行数据；

#### 3.3.2 resultMap 结果映射

- 列名和属性名不同时，

  方法一：使用resultMap指定列名与java属性的对应关系，在mapper文件中：

  ```xml
  <!--
          列名和java属性的对应关系
          主键列使用 id 标签  非主键列使用 result 标签
          column：列名    property：java属性名
      -->
  <resultMap id="studentMap" type="com.zxyong.domain.Student">
      <id column="id" property="id"/>
      <result column="name" property="name" />
      <result column="age" property="age"/>
      <result column="email" property="email"/>
  </resultMap>
  
  <select id="selectAllStudent" resultMap="studentMap">
      select * from t_student
  </select>
  ```

  方法二：sql语句种使用关键字as修改查询出的数据的列名

#### 3.3.3 模糊查询 like

- 方法1：java代码中指定like内容

  ```java
  String name = "%李%";
  List<Student> students = dao.selectByVagueName(name);
  ```

- 方法2：mapper文件中拼接like内容

  ```xml
  select * from t_student where name like "%"#{name}"%"
  ```

## 4 Mybatis 动态sql

### 4.1 动态sql之if

```xml
<select id="selectIf" resultType="com.t_yang.enity.Student">
	select * from t_student where 1=1
	<!--test属性作为判读那条件，如果为true，则添加此标签下的sql语句到逐语句。否则不添加。-->
	<!--在test属性值中，使用resultType中实体类的属性名作为属性值进行条件判断-->
	<if test="name != null and name != '' ">
		and name=#{name}
	</if>
	<if test="age > 0">
		and age > #{age}
	</if>
</select>
```

### 4.2 动态sql之where

- 用来代替sql语句中的where关键字，包含多个if标签，当有一个if标签的test成立时，就会自动给sql语句加上where关键字，

  ```xml
  <select id="selectStudentWhere" resultType="com.zxyong.domain.Student">
      select * from t_student
      <where>
          <if test="name!=null and name!=''">
              name=#{name}
          </if>
          <if test="age>0">
              and age>#{age}
          </if>
      </where>
  </select>
  ```

### 4.3 动态sql之foreach

- 用来循环java中的数组，list集合，主要用在sql的in语句中；

  ```xml
  <select id="selectStudentFor" resultType="com.zxyong.domain.Student">
      select * from t_student where id in
      <!--
  	collection：表示接口中的方法参数类型，如果是数组用array，list集合用list
  	item：自定义的，表示数组和集合成员的变量
  	open：循环开始时的字符
  	close：循环结束时的字符
  	separator：集合成员之间的字符
  	-->
      <foreach collection="list" item="myid" open="(" close=")" separator=",">
  		#{myid}
      </foreach>
  </select>
  ```

### 4.4 动态sql之代码片段

- 复用sql语句，
  1. 先定义`<sql id="自定义名称">sql语句</sql>`
  2. 再使用，`<included refid="id值" />`

## 5 mybatis 配置文件

### 5.1 使用属性配置文件

- 将数据库连接信息放在一个单独的文件中进行管理；和mybatis主配置文件分开，便于修改、保存或者处理多个数据库信息。

  在主配置文件下指定属性配置文件的位置：`<properties resource="从类路径开始相对路径" />`

### 5.2 mappers

- 当有多个mapper文件时：

  1. 写多个`<mapper/>`标签，一个对应一个mapper文件

  2. 使用包名，例如：`<package name="com.zxyong.dao">`；可以将com.zxyong.dao包下的所有xml文件一次性加载给mybatis

     要求 1）mapper文件名必须与接口名称一样；2）mapper文件与接口在相同目录下

## 6 扩展

- 分页查询工具：pageHelper

  1. 在pom.xml中加入PageHelper依赖

  2. 在Mybatis主配置文件中加入plugin插件配置：

     ```xml
     <!--environments标签前-->
     <plugins>
         <plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
     </plugins>
     ```

  3. 在获取dao接口对象后，执行抽象方法前，调用静态方法`PageHelper.startPage(int pageNum, int pageSize);`