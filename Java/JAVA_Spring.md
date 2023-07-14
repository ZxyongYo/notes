# Spring

## 一 概述

- **核心技术：**IOC、AOP，能使模块之间、类之间解耦合。
- **优点：**（1）轻量 （2）针对接口编程，解耦合 （3）AOP编程支持 （4）方便集成各种优秀框架 ；
- **使用步骤**
  1. 创建maven项目，加入spring依赖；
  2. 创建业务接口与实现类；
  3. 创建spring的配置文件，使用`<bean>`标签声明对象；
  4. 使用容器中的对象，通过`ApplicationContext`接口和它的实现类`ClassPathXmlApplicationContext`的`getBean()`方法

##  二 ioC控制反转

- **控制反转**（ioC，Inversion of Control），是一种概念，是一种思想。指将传统上由程序代码直接操控的对象调用权交给容器，通过容器来实现对象的装配和管理。控制反转就是对 对象控制权的转移，从程序代码本身反转到了外部容器。通过容器实现对象的创建，属性赋值，依赖管理。

- **控制**：创建对象，对象的属性赋值，对象之间的关系管理。

  **反转**：把原来的开发人员管理、创建对象的权限转移给代码之外的容器实现。由容器代替开发人员管理对象，创建对象，给属性赋值。

  **目的**：减少对代码的改动，实现解耦合。

  <span style="background-color:yellow">Spring框架使用依赖注入（DI）实现ioC</span>



### 2.1 Spring第一个程序

- 1. 创建maven项目，导入spring依赖

- 2. 编写接口和实现类

  ```java
  package com.zxyong.service;
  
  public interface SomeService {
      void doSome();
  }
  ```

  ```java
  package com.zxyong.service.impl;
  import com.zxyong.service.SomeService;
  
  public class SomeServiceImpl implements SomeService {
      @Override
      public void doSome() {
          System.out.println("SomeServiceImpl的doSome方法执行了");
      }
  }
  ```

- 3. 创建spring配置文件 *（resources–>new–>XML configuration file–>Spring Config）*

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans.xsd">
  
  <!--
      告诉spring创建对象
      声明bean就是告诉spring创建某个类的对象
      id：对象的自定义名称  唯一值  spring通过这个名称找到对象
      calss：类的全限定名称（不能是接口  因为spring是反射机制创建对象，必须是类）
  
      spring会自动完成  SomeService someSersvice = new SomeServiceImpl();
      spring把创建好的对象放入map中     spring有个map存放创建好的对象
      例如： springMap.put("someSersvice", new SomeServiceImpl())
      一个bean声明一个对象
  -->
      <bean id="someService" class="com.zxyong.service.impl.SomeServiceImpl"></bean>
  </beans>
  <!--
      spring的配置文件
      1.bean：是跟标签，spring把java对象成为bean
  -->
  ```

- 4. 创建测试

  ```java
  package com.zxyong;
  
  public class Mytest {
      @Test
      public void test02(){
          // 使用spring创建的对象
          // 1.指定spring配置文件名称
          String config = "beans.xml";
          // 2.创建表示spring容器的对象 ， ApplicationContext就表示spring的容器
          // ClassPathXmlApplicationContext表示从类路径中加载spring的配置文件
          ApplicationContext ac = new ClassPathXmlApplicationContext(config);
          // 从容器获取某个对象  要调用对象的方法
          // getBean("配置文件中bean的id")
          SomeService service = (SomeService) ac.getBean("someService");
          service.doSome();
      }
  }
  ```


- **Spring默认创建对象的时间： 在创建spring容器时，会创建配置文件中的所有对象**

### 2.2 容器中Bean的作用域

1. singleton：单例模式。即在整个 Spring 容器中，使用 singleton 定义的 Bean 将是单例 的，叫这个名称的对象只有一个实例。默认为单例的。
2. prototype：原型模式。即每次使用 getBean 方法获取的同一个的实例都是一个 新的实例。
3. request：对于每次 HTTP 请求，都将会产生一个不同的 Bean 实例。
4. session：对于每个不同的 HTTP session，都将产生一个不同的 Bean 实例。

- 对于scope的值为request、session只有在Web应用中使用Spring时，该作用域才有效；
- 对于scope为singleton的单例模式，该bean是在容器被创建时即被装配好了；
- 对于scope为prototype的原型模式，bean实例是在代码中使用该bean实例时才进行装配的；

### 2.3 spring提供的常用方法

- 获取spring容器中，Java对象的信息

```java
ApplicationContext ac = new ClassPathXmlApplicationContext(config);
// 获取容器中定义的对象的数量
int nums = ac.getBeanDefinitionCount();
// 获取容器中定义的每个对象的名称
String names[] = ac.getBeanDefinitionNames();
```

### 2.4 基于配置文件XML的DI

#### 2.4.1 set注入

- set注入（设值注入）：spring通过调用类中的setter方法给属性赋值；

  ```xml
  <bean id="myStudent" class="com.zxyong.Student">
  	<property name="属性名" value="简单类型的属性值"></property>
  </bean>
  ```

- **一个property标签，完成一个属性的赋值；**spring是执行`name`属性对应的set方法，并不关心set方法的具体实现和属性是否真的存在。

- 当指定bean的某属性值为另一bean的实例时，通过ref指定它们之间的引用关系；ref的值必须为某个bean的id值

  ```xml
  <bean id="myStudent" class="com.zxyong.Student">
  	<property name="属性名" value="简单类型的属性值"></property>
      <property name="school" ref="mySchool"></property>
  </bean>
  <bean id="mySchool" class="com.zxyong.School">
  	<property name="属性名" value="简单类型的属性值"></property>
  </bean>
  ```

#### 2.4.2 构造注入（理解）

- 构造注入：spring调用类的有参构造方法，在创建对象的同时，在构造方法中给属性赋值

```java
// 定义有参构造方法
public Student(String name, int age, School school){
    this.name = name;
    this.age = age;
    this.school = school;
}
```

```xml
<!--使用name属性构造注入-->
<bean id="myStudent" class="com.zxyong.ba03.Student">
    <constructor-arg name="name" value="zxy"></constructor-arg>
    <constructor-arg name="age" value="22"></constructor-arg>
    <constructor-arg name="school" ref="mySchool"></constructor-arg>
</bean>
<!--使用index属性-->
<bean id="myStudent1" class="com.zxyong.ba03.Student">
    <constructor-arg index="0" value="zxy"></constructor-arg>
    <constructor-arg index="1" value="22"></constructor-arg>
    <constructor-arg index="2" ref="mySchool"></constructor-arg>
</bean>
<!--也可以不指定index或name 单参数必须按照顺序-->
```

#### 2.4.3 引用类型自动注入

- 对于引用类型属性的注入，可以不在配置文件中显示的注入；可以通过设置`<bean>`标签的`autowire`属性，

  为引用类型属性进行自动注入；根据自动注入的判断标准不同，可以分为两种：

  1. byName 方式自动注入：java类中引用类型的属性名和spring容器中（配置文件）bean的id名称一样，可以使用byName方式，让容器自动将被调用者bean注入给调用者bean；

  2. byType 方式自动注入：java类中引用类型的数据类型和spring容器中（配置文件）bean的calss属性是同源关系；

     byType自动注入时  xml配置文件中声明的bean只能有一个符合条件的，如果多出会报错；

#### 2.4.4 为应用指定多个spring配置文件

- 在实际应用中，随着应用规模增加，系统中bean的数量也大量增加，导致配置文件变得非常庞大；为了避免这种情况提高配置文件的可读性 可维护性，可以将spring配置文件拆分为多个配置文件。

- 包含关系的配置文件：多个配置文件中有一个总文件，总配置文件通过`<import>`将其他子文件引入，在java代码中只需要使用总配置文件对容器进行初始化即可。

  ```xml
  total.xml
  <!--关键字"classpath:"表示类路径，用来告诉spring到类路径中查找文件-->
  	<import resource="classpath:ba06/spring-school.xml"/>
     	<import resource="classpath:ba06/spring-student.xml"/>
  <!--在包含关系的配置文件可以使用通配符（*：表示任意字符）-->
  	<import resource="calsspath:ba06/spring-*">
  ```

  

### 2.4 基于注解的DI（重点）

#### 2.4.1 使用步骤

  1. 创建maven项目，添加依赖

  2. 编写类，在类中添加注解

  3. 在spring配置文件中声明组件扫描器

     ```xml
     <context:component-scan base-package="com.zxyong.ba01" />
     <!-- 扫描指定多个包的三种方式-->
     <!--    1. 使用多个组件扫描器，指定不同的包-->
         <context:component-scan base-package="com.zxyong.ba01" />
         <context:component-scan base-package="com.zxyong.ba02" />
     <!--    2.使用分隔符（;或,）分割多个包名-->
         <context:component-scan base-package="com.zxyong.ba01;com.zxyong.ba02" />
     <!--    3.指定父包-->
         <context:component-scan base-package="com.zxyong" />
     ```

#### 2.4.2 @Component 注解

- **@Component：**创建类的对象，默认是单例，等同于<bean>

  属性：**value: **表示类的名称；(value可以省略不写，如果不指定对象名称，spring默认提供类名的首字母小写的对象名称)

  位置：在类定义的上面，表示创建此类的对象；

`@Component(value = "myStudent")`等价于`<bean id="myStudent" class="com.bjpowernode.ba01.Student"/>`

- 和@Component 功能一致，创建对象的注解还有：
  1. **@Repository（用于持久层类）**：创建dao对象，用来访问数据库
  2. **@Service（用于业务层类）**：创建service对象，处理业务逻辑，可以有事务功能
  3. **@Controller（用于控制器）**：创建控制器对象，接受请求，显示处理结果

#### 2.4.3 @Value注解

- **@Value**：简单类型属性注入

  属性：**value**：值必须String类型，表示简单类型的属性值；

  位置：在属性定义的上面，无需set方法

#### 2.4.4 @Autowired、@Qualifier注解

- **@Autowired**：引用类型自动注入，默认byType

  byName需要配合 **@Qualifier **使用

  属性：required 默认为true表示如果引用类型赋值失败，程序报错终止执行；false表示赋值失败，程序继续执行

  ```java
  //byName
  @Autowired(required = true)
  @Qualifier("school")	// 与School类的@component注解value值相同
  private School school;
  ```

#### 2.4.5 @Resource JDK注解

- **@Resource**：引用类型自动注入，来自JDK的注解，spring框架提供了对这个注解的支持，使用该注解必须 JDK6以上；

- 既可以`byType`也可以`byName`，**如果不指定任何参数，默认使用`byName`方式**，如果`byName`不能成功，则会再使用`byType`；

  ```java
  // 默认使用byName方式注入，找id为school的bean 找不到则再使用byType方式注入
  @Resource
  private School school;
  
  // 只是用byName方式注入
  @Resource("mySchool")
  private School school;
  ```

  

## 三 AOP 面向切面编程

- AOP：面向切面编程，基于动态代理的，可以使用jdk，cglib两种代理方式；
- AOP就是动态代理的规范化，把动态代理的实现步骤，方式都定义好了，让开发人员用一种统一的方式，使用动态代理；

### 3.1 动态代理

- 动态代理：可以在程序执行过程中，创建代理对象，通过代理对象执行方法，给目标类的方法增加额外的功能（功能增加）

- 动态代理的实现：
  - JDK动态代理：使用java反射包中的类和接口实现动态代理的功能；
  - cgLib代理：三方开源项目，可以在运行期间扩展java类，实现java接口，被广泛的AOP框架使用；
  
  使用JDK动态代理，要求目标类与代理类实现相同的接口。**若目标类五接口，则不能使用该方法实现。**
  
  cgLib代理的生成原理 使生成目标类的子类，子类对象即代理对象。因此**目标类和方法不能是final的**

- jdk动态代理实现步骤：
  1. 创建目标类，
  2. 创建InvocationHandler接口的实现类，在这个类实现给目标方法增加功能；
  3. 使用jdk中 类Proxy，创建代理对象，实现创建对象的能力；

### 3.2 AOP的相关术语

- **切面（Aspect）**：切面泛指交叉业务逻辑。例子中的事务处理、日志处理就可以理解为切面，实际就是主业务逻辑的一种增强；

- **连接点（JoinPoint）**：连接点指可以被切面织入的具体方法。通常业务接口中的方法均为连接点；

- **切入点（Pointcut）**：切入点指声明的一个或多个连接点的集合。通过切入点指定一组方法；被标记为**final**的方法不能作为连接点与切入点，因为最终是不能被修改的，不能被增强；

- **目标对象（target）**：目标对象指将要被增强的对象。即包含主业务逻辑的类的对象；

- **通知（Advice）**：通知是切面的一种实现，可以完成简单织入功能。Advice也叫增强，通知定义了增强代码切入到目标代码的时间点，是目标方法执行之前还是执行之后等，通知类型不容，切入时间不同；

  **切入点定义切入的位置，通知定义切入的时间。**

### 3.3 AOP的实现

1. Spring：Spring的AOP实现较为笨重，一般用在事务处理；

2. aspectJ：一个开源，专门做AOP的框架，

   spring框架集成了aspectJ的功能。

### 3.4 aspectJ对AOP的实现

- aspectJ目标类有接口时 默认使用jdk动态代理， 没有接口使用cglib动态代理

  ```xml
  <!--如果目标类有接口 想用cglib加入proxy-target-class="true"-->
  <aop:aspectj-autoproxy proxy-target-class="true" />
  ```

#### 3.4.1 AspectJ的通知类型 

- aspectJ中常用的通知有五种类型
  1. @Before：前置通知
  2. @AfterReturning：后置通知
  3. @Around：环绕通知
  4. @AfterThrowing：异常通知
  5. @After：最终通知

#### 3.4.2 AspectJ 的切入点表达式

```java
execution ( [modifiers-pattern]  访问权限类型
	 ret-type-pattern 返回值类型
	 [declaring-type-pattern]  全限定性类名 
	 name-pattern(param-pattern) 方法名(参数类型和参数个数) 
	 [throws-pattern]  抛出异常类型 )
```

- 切入点表达式要匹配的对象就是目标方法的方法名。所以，execution表达式中明显就是**方法的签名**。表达式中加 [ ] 的部分表示可以省略的部分，各部分之间用空格分开。

- 可以使用以下符号：

  `*` ：0至多个任意字符；

  `..` ：用在方法的参数中，表示任意多个参数；用在包名后，表示当前包及其子包；

  `+` ：用在类名后，表示当前类及其子类；用在接口后，表示当前接口及其实现类；

  ```java
  execution(public * *(..));	// 指定切入点为：任意公共方法
  execution(* set*(..));	// 指定切入点为：任何一个以“set”开始的方法
  execution(* com.zxyong.service.*.*(..));	// service包（不含子包）的任意类任意方法
  execution(* com.zxyong.service..*.*(..));	// service包或子包里的任意方法。“..”出现在类名中时，后面必须跟“*”，表示包、子包下的所有类
  execution()
  ```

#### 3.4.3 使用aspectJ实现AOP步骤

1. 新建maven项目

2. 加入spring依赖、aspectJ依赖、junit单元测试

3. 创建目标类：接口和它的实现类

4. 创建切面类：普通类

   1） 在类的上面加入@Aspect注解。

   2） 在类中定义方法，方法就是切面要执行功能代码；

   ​		在方法上面加入aspectJ的通知注解，指定切入点表达式execution。

5. 创建spring的配置文件：声明对象，把对象交给容器统一管理，

   1）声明目标对象

   2）声明切面类对象

   3）声明aspectJ框架中的自动代理生成器标签

   ​		自动代理生成器：用来完成代理对象的自动创建功能；

   ​	 `<aop:aspectj-autoproxy />`通过扫描找到`@Aspect`定义的切面类，再由切面类根据切入点找到目标类的目标方法，再由通知类型找到切入时间点。

6. 创建测试类，从spring容器中获取目标对象（实际就是代理对象）

   通过代理执行方法，实现AOP的功能增强 

#### 3.4.4 通知类型和参数

##### 3.4.4.1 JoinPoint参数

- 所有的通知方法均可包含 JoinPoint 参数，通过该参数可以获取切入点表达式、方法签名、目标对象等；必须放在参数列表的第一位。

```java
 @Before(value = "execution(* *..SomeServiceImpl.doSome(..))")
    public void myBefore(JoinPoint jp){
        //获取方法的定义
        System.out.println("连接点的方法定义："+jp.getSignature());
        System.out.println("连接点的方法名称："+jp.getSignature().getName());
        //获取方法执行时的参数
        Object args [] = jp.getArgs();
        for (Object arg : args) {
            System.out.println(arg);
        }
     }
```

##### 3.4.4.2 @AfterReturning 后置通知

- 在目标方法执行之后执行，可以获取到目标方法的返回值，该注解的`returning`属性用来指定接受方法返回值的变量名；
- 该变量一般为Object类型，因为目标方法的返回值可能是任何类型；

```java
@AfterReturning(value = "execution(* *..SomeServiceImpl.doOther(..))",returning = "result")
    public void afterReturing(JoinPoint jp, Object result){
        System.out.println("后置通知，在目标方法之后执行的。能够获取到目标方法的执行结果："+result);
    }
```

##### 3.4.4.3 @Around 环绕通知

- 特点：
  1. 功能最强的通知
  2. 在目标前后都能增强功能
  3. 能控制目标方法是否被调用
  4. 能修改目标方法的执行结果，影响最后的调用结果
  5. 有固定参数ProceedingJoinPoint， 返回值就是目标方法的执行结果

```java
@Around("execution(* *..SomeServiceImpl.doFirst(..))")
    public Object myAround(ProceedingJoinPoint pjp) throws Throwable {
        //获取目标方法的第一个参数
        String name = null;
        Object args[] = pjp.getArgs();
        if(args !=null&&args.length>1){
            name = (String) args[0];
        }
        
        Object res = null; //目标方法执行结果
        System.out.println("环绕通知 目标方法执行前"+new Date());
        // 目标方法调用
        if("zhang".equals(name)){
            res = pjp.proceed();
        }
        System.out.println("环绕通知 目标方法执行后");

        // 修改目标发放执行结果 影响目标方法调用结果
        if(res != null){
            res = "hello";
        }
        return res;
    }
```

##### 3.4.4.4 @AfterThrowing 异常通知

- 目标方法放生异常时执行， 有一个Exception参数 表示异常信息
- 属性： throwing 目标方法抛出异常的对象  变量名必须和方法的参数一样

```java
@AfterThrowing(value = "execution(* *..SomeServiceImpl.doSome(..))", throwing = "ex")
    public void myAfterThrowing(Exception ex){
        System.out.println("异常通知 方法发生异常 执行"+ex.getMessage());
    }
```

##### 3.4.4.5 @After 最终通知

- 总是会执行 ，在目标方法之后执行

##### 3.4.4.6 @Pointcut注解 

- @Pointcut注解用来定义和管理切入点，

- 特点：当使用@Pointcut定义在一个方法上面，此时这个方法的名称就是切入点表达式的别名，

  其他的通知中，value属性就可以使用这个方法的别名代替切入点表达式了

```java
@Aspect
public class MyAspect {
    @Before(value = "myPt()")
    public void myBefore(){
        System.out.println("myBefore");
    }
    @Before(value = "myPt()")
    public void myAfter(){
        System.out.println("myAfter");
    }
    @Pointcut("execution(* *..SomeServiceImpl.doSome(..))")
    private void myPt(){}
}
```

## 四 集成mybatis和spring

### 4.1 概述

- Spring集成mybaits，本质工作就是 将mybatis框架使用到的一些需要自己创建的对象，交给spring统一管理。

  主要有以下对象：

  1. 数据源dataSource。保存数据库连接信息的对象，实际开发中，不用Mybatis的数据库连接池，使用阿里的Druid
  2. 生成SqlSession对象的SqlSessionFactory对象；
  3. Dao接口的实现类对象；

### 4.2 Spring集成mybatis流程

- 1. 新建mysql数据库，建表；

  2. 创建maven项目，加入spring依赖、mybatis依赖、mybatis-spring依赖、mysql驱动、druid数据库连接池依赖；

  3. 创建实体类Student，与数据库表字段 一 一 对应;

  4. 创建Dao接口和sql映射文件；

     ```java
     public interface StudentDao {
         List<Student> selectAll();
         int  insertStudent(Student student);
     }
     ```

     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE mapper
             PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
             "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
     <mapper namespace="com.zxyong.dao.StudentDao">
         <select id="selectAll" resultType="student">
             select * from t_student order by id
         </select>
         <insert id="insertStudent">
             insert into t_student values(#{id}, #{name}, #{email}, #{age})
         </insert>
     </mapper>
     ```

  5. 创建mybatis主配置文件，使用阿里数据库连接池，所以这里不用`<environments/>`;

     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE configuration
             PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
             "http://mybatis.org/dtd/mybatis-3-config.dtd">
     <configuration>
         <!--输出日志-->
         <settings>
             <setting name="logImpl" value="STDOUT_LOGGING"/>
         </settings>
         <!--设置别名-->
         <typeAliases>
             <!--实体类所在的包名-->
             <package name="com.zxyong.domain"/>
         </typeAliases>
         <!--mapper文件的位置-->
         <mappers>
             <package name="com.zxyong.dao" />
         </mappers>
     </configuration>
     ```

  6. 创建service接口和实现类

     ```java
     public interface StudentService {
         int addStudent(Student student);
         List<Student> queryStudents();
     }
     
     public class StudentServiceImpl implements StudentService {
         private StudentDao studentDao;
         // 使用ioc，设值注入，在配置文件中给dao赋值
         public void setStudentDao(StudentDao studentDao) {
             this.studentDao = studentDao;
         }
         @Override
         public int addStudent(Student student) {
             int n = studentDao.insertStudent(student);
             return n;
         }
         @Override
         public List<Student> queryStudents() {
             List<Student> studentList = studentDao.selectAll();
             return studentList;
         }
     }
     ```

  7. 创建spring配置文件（重要）

     1）声明数据源DataSource对象；

     2）声明SqlSessionFactoryBean，创建SqlSessionFactory对象；

     3）声明mybatis的扫描器，创建Dao接口的实现类对象；

     4）声明自定义的Service，把Dao对象注入Service

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <beans>
         <!--属性配置文件-->
         <context:property-placeholder location="classpath:jdbc.properties"/>
         <!--声明数据源Datasource, 用来连接数据库-->
         <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
             <property name="url" value="${jdbc.url}" />
             <property name="username" value="${jdbc.username}" />
             <property name="password" value="${jdbc.password}" />
             <property name="maxActive" value="20" />
         </bean>
         
         <!--声明mybatis中提供的SqlSessionFactoryBean, 创建SqlSessionFactory-->
         <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
             <!--set注入,把数据库连接池赋给它-->
             <property name="dataSource" ref="dataSource" />
             <!--mybatis主配置文件的位置-->
             <property name="configLocation" value="classpath:mybatis.xml" />
         </bean>
         
         <!--创建dao对象,使用SqlSession的getMapper(StudentDao.class)-->
         <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
             <!--指定SqlSessionFactory对象-->
             <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
             <!--指定dao接口所在的包名,每个接口都执行一次mapper方法,得到的每个接口的dao对象,创建好放入spring容器中-->
             <property name="basePackage" value="com.zxyong.dao" />
         </bean>
         
         <!--创建service对象-->
         <bean id="studentService" class="com.zxyong.service.impl.StudentServiceImpl">
             <property name="studentDao" ref="studentDao" />
         </bean>
     </beans>
     ```

  8. 创建测试类，从spring容器中获取service，调用service方法

     ```java
     @Test
     public void testQueryStudents(){
         String config = "applicationContext.xml";
         ApplicationContext ac = new ClassPathXmlApplicationContext(config);
         StudentService service = (StudentService) ac.getBean("studentService");
         List<Student> studentList = service.queryStudents();
         studentList.forEach(System.out::println);
     }
     ```


## 五 spring 事务

### 5.1 理论

#### 5.1.1 事务管理器接口

- 事务管理器是PlatformTransactionManager 接口对象，主要用于完成事务的提交、回滚，及获取事务的状态信息；

  有两个常用的实现类

  - DataSourceTransactionManager：使用 JDBC或mybatis进行数据库操作时使用；
  - HibernateTransactionManager：使用 Hibernate 进行持久化数据时使用；

#### 5.1.2 spring 的回滚方式

- spring 事务的默认回滚方式是：发生运行时异常和error时回滚，发生受查（编译）异常时提交。

  不过，对于受查异常，程序员也可以手动设置其回滚方式。

- **运行时异常**，是RuntimeException类或其子类，即只有在运行时才出现的异常。

  这些异常由JVM抛出，在编译时不要求必须处理（捕获或抛出）。只要代码写的够仔细，程序足够健壮，运行时异常是可以避免的。

- **受查异常**，也叫编译时异常，即在代码编写时要求必须捕获或抛出的异常，若不处理，则无法通过编译。

#### 5.1.3 事务定义接口

​	事务定义接口TransactionDefinition中定义了事务描述相关的三类常量：事务隔离级别、事务传播行为、事务默认超时时限，及对他们的操作。

- **定义了五个事务隔离级别常量**

  这些常量均是以ISOLATION_开头。

  - DEFAULT：采用DB默认的事务隔离级别。MySQL默认REPEATABLE_READ;Oracle默认为READ_COMMITTED。
  - READ_UNCOMMITTED：读未提交。未解决任何并发问题。
  - READ_COMMITTED：读已提交。解决脏读，存在不可重复读与幻读。
  - REPEATABLE_READ：可重复读。解决脏读、不可重复读，存在幻读。
  - SERIALIZABLE：串行化。不存在并发问题。

- **定义了七个事务传播行为常量**

  所谓事务传播行为是指，处于不同事务中的方法在相互调用时，执行期间事务的维护情况。如：A事务中的方法doSome()调用 B事务中的方法doOther()，在调用执行期间事务的维护情况，就成为事务传播行为。事务传播行为是加在方法上的。事务传播行为常量都是以PROPAGATION_开头。

  **PROPAGATION_REQUIRED**：指定的方法必须在事务内执行。若当前存在事务，就加入到当前事务中；若当前没有事务，则创建一个新事务。这种传播行为是最常见的，也是spring默认的事务传播行为。
  **PROPAGATION_REQUIRES_NEW**：指定的方法支持当前事务，但若当前没有事务，也可以以非事务方式执行。
  **PROPAGATION_SUPPORTS**：总是新建一个事务，若当前存在事务，就将当前事务挂起，直到新事务执行完毕。
  PROPAGATION_MANDATORY
  PROPAGATION_NESTED
  PROPAGATION_NEVER
  PROPAGATION_NOT_SUPPORTED

### 5.2 使用Spring的事务注解管理事务

1. 开启注解驱动

2. 声明事务管理器

   ```xml
   <!--applicationContext.xml-->
   <!--使用spring的事务处理   声明事务管理器-->
   <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
       <!--指定数据源-->
       <property name="dataSource" ref="dataSource" />
   </bean>
   <!--开启事务注解驱动，告诉spring使用注解管理事务，创建代理对象-->
   <tx:annotation-driven transaction-manager="transactionManager" />
   ```

3. 业务层 public 方法上添加注解@Transactional，只能用在public方法上

### 5.3 使用AspectJ的AOP配置管理事务

- 一般大型项目使用，不用修改源代码的条件下管理事务。

1. 添加Maven依赖，spring-aop依赖、spring-aspects依赖

2. 添加事务管理器

3. 配置事务通知，为事务通知设置相关属性。

   ```xml
   <!--声明业务方法他的事务属性（隔离级别，传播行为，超时时间）-->
   <tx:advice id="myAdvice" transaction-manager="transactionManager">
       <tx:attributes>
           <tx:method name="buy" propagation="REQUIRED" isolation="DEFAULT"
                      rollback-for="java.lang.NullPointerException, com.zxyong.excep.NotEnoughException"/>
           <!--使用通配符，指定很多方法-->
           <tx:method name="add*" propagation="REQUIRES_NEW" />
           <tx:method name="modify*" />
           <tx:method name="remove*" />
           <tx:method name="*" propagation="SUPPORTS" read-only="true" />
       </tx:attributes>
   </tx:advice>
   ```

4. 配置增强器，指定事务管理的方法所在的类

   ```xml
   <aop:config>
       <!--配置切入点表达式：指定哪些包中类，需要使用事务-->
       <aop:pointcut id="servicePt" expression="execution(* *..service..*.*(..))"/>
       <!--配置增强器：关联 事务通知对象advice 和 切入点表达式pointcut -->
       <aop:advisor advice-ref="myAdvice" pointcut-ref="servicePt" />
   </aop:config>
   ```


## 六 Spring与Web

- 解决servlet中重复创建spring容器对象，造成资源浪费的问题。

  1. 添加maven依赖，spring-web

  2. 在web.xml中注册监听器

     ```xml
     <!--监听器被创建后会读取 /WEB-INF/applicationContext.xml
         可以使用context-param重新指定文件的位置-->
     <context-param>
         <param-name>contextConfigLocation</param-name>
         <param-value>classpath:applicationContext.xml</param-value>
     </context-param>
     <listener>
         <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
     </listener>
     ```

     

  3. 在servlet中获取**ApplicationContext**对象

     ```java
     /*WebApplicationContext ac = null;
     // 获取Servlet Context中的容器对象
     String key = WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE;
     Object attr = getServletContext().getAttribute(key);
     if(attr != null){
     	ac = (WebApplicationContext) attr;
     }*/
     
     // 使用框架提供的工具类获取
     ServletContext sc = getServletContext();
     WebApplicationContext ac = WebApplicationContextUtils.getRequiredWebApplicationContext(sc);
     ```

     