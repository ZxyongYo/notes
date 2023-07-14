## 基础语法
#### 基本数据类型
- 整数类型：byte，short，int，long
- 浮点数类型：float，double
- 字符类型：char
- 布尔类型：boolean  
**数据类型转换**
- `Boolean`类型不能转换为其他数据类型。
- 整形，字符型，浮点型在混合运算中互相转换，容量小的类型自动转为容量大的数据类型  
> byte, short, char->int->long->float->double  
> byte, short, char之间不会互相转换，**他们三者在计算时会先转换为`int`类型**  
> 容量大的转为容量小的数据类型时，要加上强制转换符，但可能造成精度降低或溢出  

 #### 循环，条件控制
- swich里边只能放int类型，可以自动转换

## 面向对象

#### 内存解析
- 定义成员变量时，可以初始化，如果不初始化，java使用默认值对其初始化
- **构造方法**是方法名与类名相同没有返回值的方法， 用new关键字构建一个新的对象
- 当没有指定构造函数时，编译器会自动添加
****

#### 方法重载（overload）
- 方法重载是指一个类中可以定义有相同名字 但参数不同的多个方法。
调用时会根据不同的参数列表调用对应的方法。

#### this关键字
- 在类的方法定义中使用this关键字代表使用该方法的对象引用，
this可以看作一个对象它的值是当前对象的引用。

#### static关键字
- 类中用static声明的变量成为静态变量，它为该类的公共变量，
在第一次使用时被初始化，用于该类的所有对象来说，static成员变量只有一份。
- 用static声明的方法为静态方法，在调用该方法时，不会将对象的引用传递给它，
所以在static方法中不能访问非static成员。

#### package 和 import
- 如果想将一个类放入包中，在这个类的第一行指定package，
必须保证该类的class文件位于正确的目录下。
- 同个包直接访问，其它访问需要写全名或用import引入

#### 继承和权限控制
- java中使用`extends`关键字实现类的继承，通过继承，子类自动拥有基类的所有成员变量。
- java只支持单继承，不允许多继承，一个子类只能有一个基类，一个基类可以派生出多个子类。

> private 类内部  
> default 类内部  同一个包  
> protected  类内部  同一个包  子类  
> public  类内部  同一个包  子类  任何地方
- 对于class只能用public 或 default修饰
- 子类构造的过程中必须调用父类的构造方法，如果子类没有显示的调用父类的构造方法，系统默认调用
父类的无参构造方法，如果没有则编译出错。

#### 方法重写（override）
- 子类可以重写父类的方法, 重写方法必须和被重写方法具有相同方法名称、参数列表和返回类型。
- 重写方法不能使用比被重写方法更严格的访问权限。

#### super关键字
- 在java中使用`super`关键字引用父类的属性和方法

#### 对象转型
- 一个父类的引用可以指向其子类的对象， 父类的引用不能访问其子类新增的成员（属性和方法）
- 使用 引用变量 instanceof 类名 来判断该引用类型是否属于该类或该类的子类
- 子类的对象可以当作父类的对象来使用称作向上转型，反之称为向下转型。

#### 动态绑定和多态
- 动态绑定是指在执行期间（而非编译期）判断所引用对象的实际类型，根据实际类型调用其相应的方法。
- 三个必要条件：1.要有继承  2.要有重写  3.父类引用指向子类对象

#### 抽象类
- 用abstract关键字修饰的类叫抽象类，用abstract修饰的方法叫抽象方法。
- 含有抽象方法的类必须声明为抽象类，抽象类必须被继承，抽象方法必须被重写。
- 抽象类不能被实例化，抽象方法只需声明不需实现。

#### final关键字
- 用final修饰的变量一旦初始化，就不能改变，final的方法不能被重写，final类不能被继承。

#### 接口（interface）
- 接口是抽象方法和常量的定义的集合。
- 接口是一种特殊的抽象类，这种抽象类中只包含常量和方法的定义，而没有变量和方法的实现。

#### 数组
- 数组拷贝：`System.arraycopy(src, srcPos, dest, destPos, length);`
- src:原数组、srcPos：原数组拷贝起始位置、dest：目标数组、destPos：目标数组起始位置、length：拷贝元素个数

#### 工具类
- String 类常用方法：  

  ```java
  public char charAt(int index)   //返回字符串中第index个字符   
  public int length()     //返回字符串的长度   
  public int indexOf(String str)  //返回字符串中出现str的第一个位置  
  public int indexOf(String str, int fromIndex)   //返回str中从fromIndex开始出现str的第一个位置   
  public boolean equalsIgnoreCase(String another)  //比较str与another是否一样（忽略大小写）    
  public String replace(char oldChar, char newChar)   //在str中用newChar字符替换oldChar字符 ...
  ```

  

- **StringBuffer**将一个字符串转为一个可变的字符序列

#### 基础数据类型包装类
```java
 int i = 100;   
 Integer n = new Integer(i);   //基本类型转为包装类型   
 n.intValue();   //包装类型转为基本类型  
```



## 容器 

**如果重写了equals()方法，也要重写hashCode()方法**

- Java API 所提供的一系列类的实例，用于在程序中存放对象。
- set 无顺序 不能重复； list 有顺序 可以重复
- 所有实现了Collection接口的容器类都有一个iterator方法，iterator对象称作迭代器，用以方便对
容器内元素遍历，提供了：boolean hasNext(); Object next(); void remove(); 方法。
- 选择： Array 读快改慢； Linked改快读慢； Hash两者之间；

- 自动装箱/拆箱 : 自动将基础类型转换为对象 / 自动将对象转换为基础类型

## 流 stream
- 按数据流的方向不同可分为输入流和输出流  

- 按处理数据单位不同可分为字节流和字符流  
  按功能不同可分为节点流和处理流
  
- J2 SDK提供的所有流类型位于包java.io内分别继承自以下四种抽象流类型  
  
|        | 字符流 | 字节流       |
| ------ | ------ | ------------ |
| 输入流 | Reader | InputStream  |
| 输出流 | Writer | OutputStream |


​    

```java
FileInputStream in = new FileReader("E:\\test.java");	// 创建输入流实例
FileOutputStream out = new FileWriter("C:\\test.txt");	// 创建输出流实例
// 如果读到东西就写到另一个文件
while ((b = in.read()) != -1){
    out.write(b);
}
// 关闭流
in.close();
out.close();
```
#### 缓冲流
- `BufferedInputStream、BufferedOutputStream、BufferedWriter、BufferedReader`

#### 转换流

#### 数据流



## JDBC

#### JDBC 编程六部

- 1. 注册驱动（告诉Java程序，要连接的是哪个数据库）
  
     ```java
     // mysql  先导入mysql驱动包
     Class.forName("com.mysql.cj.jdbc.Driver");
     ```
  
  2. 获取连接（表示JVM的进程和数据库进程之间的通道打开了，进程之间的通信，使用完一定要关闭）
  
     ```java
     Connection conn = DriverManager.getConnection(url, user, password);
     ```
  
  3. 获取数据库操作对象（执行sql语句的对象）
  
     ```java
     // PreparedStatement ps = conn.prepareStatement(sql);	//预处理操作对象
     Statement stmt = conn.createStatement();
     ```
  
  4. 执行SQL语句
  
     ```java
     // select: excuteQuery		update、delect、insert: excuteUpdate
     ResultSet rs = stmt.excuteQuery(sql);
     ```
  
  5. 处理查询结果集（只有当第四步执行的是select语句的时候才有）
  
     ```java
     while(rs.next){
         String name = rs.getString("name");
         System.out.print(name);
     }
     ```
  
  6. 释放资源（使用完资源之后一定要关闭）
  
     ```java
     // 从小到大依次关闭  捕获异常也应该一个一个捕获
     rs.close();
     ps.close();
     conn.close();
     ```
  
     

## Servlet

- Servlet来自于Servlet规范下的一个接口，存在于Http服务器提供的jar包。
- Servlet规范中，Http服务器能调用的【动态资源文件】必须是一个Servlet接口的实现类。

#### Servlet开发步骤

- 第一步：创建一个Java类继承于HttpServlet父类，使之成为一个Servlet接口实现类

- 第二步：重写HttpServlet父类两个方法，doGet  、 doPost

- 第三步：将Servlet接口实现类信息【**注册**】到Tomcat服务器

  > 【网站】---> 【web】 ---> 【WEB-INF】 ---> web.xml

  ```xml
  <!--将Servlet接口实现类 类路径地址交给Tomcat-->
  <servlet>
  	<servlet-name>xxxx</servlet-name>	<!--声明一个变量存储servlet接口实现类 类路径-->
      <servlet-class>com.zxyong.controller.OneServlet</servlet-class>		<!--声明servlet接口-->
      <load-on-startup>1</load-on-startup>	<!--在启动时创建-->
  </servlet>
  
  <!--为了降低用户访问Servlet接口实现类难度，需要设置简短请求别名-->
  <servlet-mapping>
  	<servlet-name>xxxx</servlet-name>
      <url-pattern>/do</url-pattern>	<!--设置简短的请求别名，别名在书写时必须以"/"开头-->
  </servlet-mapping>
  ```


- 将处理结果返回给浏览器

  ```java
  public class ThreeServlet extends HttpServlet {
      protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
          /**
           * 要使用html标签
           * 需要在得到输出流之前 设置响应头 content-type  默认是 "text"
           */
          String result = "java<br/>HTML<br/>Vue<br/>MySQL<p>哈哈哈<p/>";
          // 设置响应头
          response.setContentType("text/html;charset=utf-8");
          PrintWriter out = response.getWriter();
          out.print(result);
      }
  }
  ```



# 框架

## spring mvc

#### RequestMapping注解

- RequestMapping注解的作用是建立请求URL和处理方法之间的对应关系

  ```java
  @Controller		// 把当前类交给IOC容器进行管理
  @RequestMapping(value = "/role")    //一级请求路径
  public class RoleController {
      /*RequestMapping的属性:
          1. path 指定请求路径的url
          2. value value属性和path属性是一样的
          3. mthod 指定该方法的请求方式
          4. params 指定请求参数*/
      @RequestMapping(
              value = "/save.do",
              method = {RequestMethod.GET, RequestMethod.POST},
              params = "username")     //二级请求路径
      public String save(String username){
          System.out.println("save role ..."+username);
          return "suc";
      }
  }
  
  ```

  

#### RequestParam注解

- 作用：把请求中的指定名称的参数传递给控制器中的形参赋值
- 属性：value：请求参数中的名称； required：请求参数中是否必须提供此参数；defaultValue：参数默认值

```java
@RequestMapping("/save1.do")
public String save(@RequestParam(value = "username",required = false,defaultValue ="abc") String name){
    System.out.println("姓名："+name);
    return "suc";
}
```

#### RequestBody注解

- 作用：用于获取请求体的内容（get方法不可以）

```java
@RequestMapping("/save2.do")
public String save2(@RequestBody String body){
    System.out.println("请求体内容："+body);
    return "suc";
}
```

#### PathVaribale注解

- 作用：拥有绑定url中的占位符。例如：url中有 /delete/{id}，{id}就是占位符
- restful风格的URL
  - 请求路径一样，可以根据不同的请求方式去执行后台的不同方法。
  - restful风格的URL优点：1.结构清晰；2.易于理解；3.易于理解；4.扩展方便

```jav
@Controller
public class EmpController {
    // 保存
    @RequestMapping(path = "/emp", method = RequestMethod.POST)
    public String save() {
        System.out.println("保存");
        return "suc";
    }
    // 查询全部
    @RequestMapping(path = "/emp", method = RequestMethod.GET)
    public String findAll() {
        System.out.println("查询全部");
        return "suc";
    }
    // 查询一个
    @RequestMapping(path = "/emp/{id}", method = RequestMethod.GET)
    public String findById(@PathVariable(value = "id") Integer id) {
        System.out.println("通过id查询"+id);
        return "suc";
    }
}
```

#### RequestHeader注解

- 作用：获取指定请求头的值

#### CookieValue注解

- 作用：用于获取指定cookie的名称的值

#### ModelAndView返回值

- ModelAndView对象是spring提供的一个对象，可以用来调整jsp视图

```java
@RequestMapping("/save3.do")
public ModelAndView save3(){
    // 创建mv对象
    ModelAndView mv = new ModelAndView();
    // 把一些数据，存储到mv对象中
    mv.addObject("msg","用户名或者密码已经存在");
    // 设置逻辑视图的名称
    mv.setViewName("suc");
    return mv;
}
// jsp页面 用 ${msg} 取值
```

#### forward & redirect 转发和重定向

```java
@RequestMapping("/save4.do")
public String save4(){
    System.out.println("执行了...");
    //return "forward:/pages/suc.jsp";	//转发
    return "redirect:/pages/suc.jsp";	//重定向
}
```

#### RequestBody & ResponseBody 注解

- 处理json数据，需要使用对应的jar包

```java
@RequestMapping("/save6.do")
public @ResponseBody User save6(@RequestBody User user){
    System.out.println(user);
    // 模拟，调用业务层代码
    user.setUsername("hello");
    user.setAge(100);
    // 把user对象转换成json，字符串，再响应。使用@ResposeBody注解 
    return user;
}
```


