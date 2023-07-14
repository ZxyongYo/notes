# Spring mvc

## 1 Spring MVC 概述

- Spring MVC是属于spring框架内的一个模块，专门用做web开发。

- Spring MVC可以理解为servlet的升级，它在servlet的基础上添加了一些功能是开发人员开发web更方便。

- Spring MVC就是一个Spring，它可以像Spring一样看作是一个容器，只不过这是一个用来存放控制器对象的容器，

  开发人员需要做的就是使用@Controller注解创建控制器对象，并放在Spring MVC容器中。

- 控制器对象就是一个像servlet一样能够接收用户请求，显示处理结果。但它是一个普通的类，并不是servlet，Spring MVC赋予了它特殊功能。
- Spring MVC中有一个Servlet：DispatherServlet（“中央调度器”），它负责接收用户所有请求，并转发给控制器对象，由控制器对象处理请求。

### 1.1 实现步骤

1. 新建web maven工程。

2. 加入依赖。spring-webmvc依赖   间接把spring的依赖都加入到项目，jsp，servlet依赖。

3. 重点 ： 在web.xml中注册spring mvc框架的核心对象DispatcherServlet。

   ```xml
   // webapp/WEB-INF/web.xml
   <!--
   	声明注册DispatcherServlet  需要在tomcat启动时创建DispatcherServlet对象实例
   	因为 在DispatcherServlet创建的过程中会同时创建springmvc容器对象，读取springmvc的配置文件
   	把这个配置文件中的对象都创建好 ， 当用户发起请求时就可以直接使用对象了
   -->
   <servlet>
       <servlet-name>springmvc</servlet-name>
       <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
       <!--自定义springmvc读取配置文件的位置  在resources目录下创建springmvc.xml配置文件-->
       <init-param>
           <param-name>contextConfigLocation</param-name>
           <param-value>classpath:springmvc.xml</param-value>
       </init-param>
       <!--load-on-startup 在tomcat启动后就创建这个对象
           它的值是个大于0的整数 表示创建的顺序
   	-->
       <load-on-startup>1</load-on-startup>
   </servlet>
   
   <servlet-mapping>
       <servlet-name>springmvc</servlet-name>
       <!-- 使用框架的时候url-pattern可以使用两种值
               1.使用扩展名方式 *.xxx  xxx是自定义的扩展名  表示只要扩展名是xxx的请求 都交给这个servlet处理
               2.使用斜杠 “ / ”-->
       <url-pattern>*.do</url-pattern>
   </servlet-mapping>
   ```

4. 创建一个发起请求的页面 index.jsp。

   ```jsp
   <a href="some.do">发起some.do请求</a>
   ```

5. 创建控制器类

   1）在类的上面加入@Controller注解，创建对象，并放在springmvc容器中。

   2）在类中的方法上面加入@RequestMapping注解，用来处理请求。

   ```java
   // @Controller注解 创建控制器对象  放在springmvc容器中
   @Controller
   public class myController {
       @RequestMapping("some.do")
       public ModelAndView doSome(){
           ModelAndView mv = new ModelAndView();
           // 添加数据 框架在请求最后 把数据放在request作用域
           mv.addObject("msg", "hello");
           mv.addObject("fn", "doSome()");
           // 指定视图 指定视图的完整路径  框架对试图执行forward操作
           mv.setViewName("/show.jsp");
           return mv;
       }
   }
   ```
   
6. 创建一个作为结果的jsp，显示请求的处理结果。

   ```jsp
   // webapp/show.jsp
   <p>msg: ${msg}</p>
   <p>fn: ${fn}</p>
   ```

7. 创建springmvc的配置文件（跟spring的配置文件一样）

   1）声明组件扫描器，指定@Contorller注解所在的包名。

   2）声明视图解析器。帮助处理视图。

   ```xml
   // resources/springmvc.xml
   <context:component-scan base-package="com.zxyong.controller" />
   ```
   
8. 创建视图解析器，

   避免用户可以直接通过地址栏访问show.jsp，可以将它放在WEB-INF目录下或子目录下，

   把show.jsp放在WEB-INF/view下

   ```xml
   // springmvc.xml
   <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
       <!--前缀-->
       <property name="prefix" value="/WEB-INF/view/" />
       <!--后缀-->
       <property name="suffix" value=".jsp" />
   </bean>
   ```

****

- springmvc请求处理流程
  1. 发起some.do
  2. tomcat (web.xml - - url-pattern 知道 *.do 的请求给 DispatcherServlet)
  3. DispatcherServlet 根据springmvc配置文件知道some.do ---> doSome()方法
  4. DispatcherServlet 把some.do转发给MyController.doSome()方法
  5. 框架执行doSome方法  把得到的ModelAndView处理 ，转发到show.jsp

## 2 Springmvc 注解式开发

### 2.1 @RequestMapping注解

- @RequestMapping注解 请求映射， 把一个请求和一个方法绑定在一起 一个请求指定一个方法处理
  	属性：value 接收的请求 可以接收一个数组

  ​				method 接收的请求方式


```java
@Controller
@RequestMapping（"/test"）	// 在控制器类上添加 一级请求路径 
public class MyController(){
    @RequestMapping(value = "/some.do", method = RequestMethod.GET)
    public ModelAndView doSome(){
        System.out.println("/test/some.do");
        return mv;
    }
}
```

### 2.2 接收参数

- **方法1**：在控制器方法中添加形式参数HttpServletRequest、HttpServletResponse、HttpSession。

- **方法2**：逐个接收。 控制器方法的形参名和请求中的参数名必须一致。

  如果不一致 ，使用@RequestParam注解

  ```java
  /*
  	@RequestParam 属性：1.value 请求中的参数名
  					2.require Boolean类型 表示请求中是否必须包含此参数 默认是true
  */
  @RequestMapping(value = "first.do", method = RequestMethod.POST)
  public ModelAndView testPost(@RequestParam(value = "username", required = false) String name, Integer age){
      ModelAndView mv = new ModelAndView();
      mv.addObject("username", name);
      mv.addObject("age", age);
      mv.setViewName("show");
      return mv;
  }
  ```


- **方法3**：使用java对象接收

  ```java
  @RequestMapping(value = "first.do", method = RequestMethod.POST)
  /**
       * 控制器方法形参是java对象，这个对象的属性名和请求中参数名一样
       * 框架会创建形参java对象，调用set方法 把请求参数赋值给java对象中对应的属性
       */
  public ModelAndView testPost(Student student){
      ModelAndView mv = new ModelAndView();
      mv.addObject("username", student.getUsername());
      mv.addObject("age", student.getAge());
      mv.setViewName("show");
      return mv;
  }
  ```

- 使用过滤器解决 post方式提交中文乱码

```xml
// webapp/WEB-INF/web.xml
<!--声明过滤器，解决post请求乱码问题-->
<filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <!--设置项目中使用的字符编码-->
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
    <!--强制请求对象使用encoding编码的值-->
    <init-param>
        <param-name>forceRequestEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
    <!--强制响应对象使用encoding编码的值-->
    <init-param>
        <param-name>forceResponseEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <!-- /* 表示 强制所有请求先通过 过滤器-->
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### 2.3 处理器方法的返回值

- 1 返回 ModelAndView。

- 2 返回 String。 如果处理方法返回时不涉及数据，可以返回一个String。
- 3 返回 void。（用来处理ajax请求，不常用）。
- 4 返回对象 Object。响应ajax，实现步骤：
  
  1. 加入处理json的工具库依赖，springmvc默认使用jackson
  2. 在 springmvc配置文件中加入注解驱动 `<mvc:annotation-driven>`
3. 在处理器方法上面加@ResponseBody注解
  
- 如果有@ResponseBody注解返回String 就是返回的数据，如果没有返回的就是视图。

  如果返回String数据包含中文，需要在@RequestMapping注解中添加属性` produces=“text/plain;charset=utf-8”`，否则会乱码

### 2.4 解读`<url-pattern>`标签

- tomcat是如何处理静态资源请求的？

  在 tomcat安装文件 conf/web.xml下，存在一个默认Servlet，用来处理所有没有映射到其他servlet的servlet。

- 在springmvc中，中央调度器的url-pattern如果只写 “ / ”，可以处理所有请求，但会造成静态资源无法访问。

- 解决方法 一：

  在springmvc.xml配置文件中添加：

  ```xml
  <!--注解驱动-->
  <mvc:annotation-driven />

  <!--处理静态资源
  原理是创建DefaultServletHttpRequestHandler对象，类似控制器，它能将静态资源请求转发给tomcat的defaultServlet-->
  <mvc:default-servlet-handler />
  ```
  
- 解决方法二：

  在springmvc.xml配置文件中添加：

  ```xml
  <!--注解驱动-->
  <mvc:annotation-driven />
  <!--把资源都放在webapp/static目录下
  	<mvc:resources /> 加入后 框架会创建 ResourceHttpRequestHandle这个处理对象
          让这个对象处理静态资源的访问，不依赖tomcat服务器
          mapping 访问静态资源的uri地址
          location： 静态资源在你项目中的目录位置
  -->
  <mvc:resources mapping="/static/**" location="/static/" />
  ```

## 3 ssm整合开发

## 4 Springmvc核心技术

### 4.1 请求转发与重定向

#### 4.1.1 请求转发

`ModelAndView.setViewName("forward:转发目标完整路径")`；

- 特点：不与视图解析器一同使用

#### 4.1.2 重定向

`ModelAndView.setViewName("redirect:重定向目标完整路径")`；

- 特点：不与视图解析器一同使用。本质是两次访问，不能访问WEB-INF目录下的资源

- 由于两次不同的请求，不能通过getAttribute获得参数。当向ModelAndView对象添加数据后，

  这些数据会再发送第二次请求时以get方式呈现在url后。

### 4.2 异常处理

- springmvc处理异常使用的是aop的思想，将异常处理代码与业务代码分开，达到解耦合的目的。

- 1. 创建自定义的异常类，  继承java.lang.Exception，重写无参构造器和带String参数的构造器

  2. 在Controller类上抛出异常

  3. 创建异常处理类

     类上加@ControllerAdvice注解

     在处理异常的方法上添加@ExceptionHandler注解，

  4. 在springmvc配置文件中添加扫描器与注解驱动

     ```xml
     <context:component-scan base-package="异常所在的包名" />
     <mvc:annotation-driven />
     ```

### 4.3 拦截器

- 拦截器和过滤器类似，功能侧重点不同，过滤器使用来过滤请求参数，设置编码字符集等工作的。

  拦截器是拦截用户请求，做请求的判断处理的。

- 拦截器是全局的，可以对多个Controller做拦截，常用在用户登录处理，权限检查，记录日志等。

#### 4.3.1 使用步骤

- 创建拦截器类实现HandleInterceptor接口，该接口有三个方法：

- **preHandle**：该方法在处理器方法执行之前执行。返回值为Boolean，若为true，会紧接着执行处理器方法，

  且会将afterCompletion()方法放入到一个专门的方法栈中等待执行。

- **postHandle**：该方法在处理器方法执行之后执行。方法参数中包含ModelAndView，

  可以修改处理器方法的处理结果，可以修改跳转页面。

- **afterCompletion**：当preHandle方法返回true时，会将该方法放到专门的方法栈中，等到对请求进行响应的所有工作完成之后才执行该方法。