> 注意：由于本笔记大量使用OCR截图文字识别，所以会出现部分代码缺少等号或者字母出错，缺少的注解可以去黑马提供的笔记中的“知识点”部分寻找

# 核心思想：充分解耦

* IoC : 由主动new产生对象转换为由外部提供对象，此过程中对象创建控制权由程序转移到外部，此思想称为控制反转

* bean：Ioc容器负责对象的创建、初始化等一系列工作，被创建或被管理的对象在IoC容器中统称为Bean

* IOC容器：管理bean，在IoC容器内将有依赖关系的bean进行关系绑定
* 效果：使用对象时不仅可以直接从Ioc容器中获取，并目获取到的bea已经绑定了所有的依赖关系 

# 注解开发

## SpringConfig

`@Configuration`代表这是一个配置类

`@ComponentScan("com.orion")`,配置多个扫描包时用数组：`@ComponentScan({"com.orion.service","com.orion.dao"})`

### 使用包扫描引入(不能快速知道引入了哪些配置类)

详见Spring_day02 4.3.1

### 使用`@Import`引入

这种方案**要引用的类**可以不用加`@Configuration`注解，但是必须在Spring配置类上使用`@Import`注解手动引入
需要加载的配置类
步骤1:去除JdbcConfig类上的注解
步骤2:在Spring配置类中引入

#### 注意!:

* 扫描注解可以移除(也就是说可以不ComponentScan这个包)
* `@Import`参数需要的是一个数组，可以引入多个配置类。
* `@Import`注解在配置类中只能写一次!

## 自动装配：(括号内为示例)

### 引用类型：@Autowired按类型注入和附加@Qualifier("bookDao2")改用按名称注入

暴力反射，不用提供setter

建议使用无参构造方法创建对象（默认），不然造不出对象

### 简单类型（包括String）：@Value("itheima")

### properties（以配置jdbc.properties的name为例）

在SpringConfig类前加上`@PropertySource("jdbc.properties")`

配置多个properties请用数组：`@PropertySource({"jdbc.properties","jdbc.properties"})`

**不支持使用通配符！如**`*.properties`

然后你要用到的属性前`@Value("${name}")`

### 第三方bean

`@Bean`后没加括号就会按类型匹配，因为加载SpringConfig时会加载这些配置，所以我们直接：

* 按类型匹配(请脑补代码补全)：

  ```java
  new ACAC;//叫ctx
  ctx.getBean(DataSource.class);
  ```

#### DataSource

```java
@Configuration
@ComponentScan("com.orion")
@PropertySource("classpath:jdbc.properties")
@Import({JdbcConfig.class})//多个请用数组，单个不用加大括号也行
public class SpringConfig{
}
```

```java
public class JdbcConfig {
    //1. 先提供一个方法用来获得对应的bean
	//2. 添加@Bean，表明该方法返回值是个Bean
	@Bean
	public DataSource dataSource5(){
    	DruidDataSource ds = new DruidDataSource();
    	ds.setDriverClassName("com.mysql.jdbc.Driver");
    	ds.setUrl("jdbc:mysql://localhost:3306/spring_db");
    	ds.setUserName("root");
    	ds.setPassword("123456");
    	return ds;
	}
}
```

#### 为第三方bean注入资源

引用类型放方法形参里，会自动装配

简单类型就当成员变量

```java
public class JdbcConfig {
    @Value("com.mysql.jdbc.Driver")
	private String driver;
	@Value("jdbc:mysq1://localhost:3306/spring_db")
	private String url;
	@Value("root")
	private String userName;
	@Value("123456")
	private String password;
	@Bean
	public DataSource dataSource5(BookDao bookDao){
    	DruidDataSource ds new DruidDataSource();
		ds.setDriverclassName(driver);
		ds.seturl(url);
		ds.setUsername(userName);
		ds.setPassword(password);
		return ds;
	}
}
```

## 总结

![image-20230106124433297](images/image-20230106124433297.png)

# Spring整合Mybatis

多import一个就行

```java
@Configuration
@ComponentScan("com.orion")
@PropertySource("classpath:jdbc.properties")
@Import({JdbcConfig.class,MybatisConfig.class})//多个请用数组，单个不用加大括号也行
public class SpringConfig{
}
```



要导的包

```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-jdbc</artifactId>
	<version>5.2.10.RELEASE</version>
</dependency>
<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis-spring</artifactId>
	<version>1.3.0</version>
</dependency>
```

只有加注释的地方是动态的，其他地方都是固定格式

```java
public class MybatisConfig {
	@Bean
	public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource){
		SqlSessionFactoryBean ssfb new SqlSessionFactoryBean();
		ssfb.setTypeAliasesPackage("com.itheima.domain");//扫描类型别名的
		ssfb.setDataSource(dataSource);
		return ssfb;
    }
	@Bean
	public MapperScannerConfigurer mapperScannerConfigurer(){
		MapperScannerConfigurer msc new MapperScannerConfigurer();
		msc.setBasePackage("com.itheima.dao");//扫描映射的包
		return msc;
    }
}
```

## 补充（摘自黑马的SSM笔记）：FactoryBean都是造对象的

查看源码会发现，FactoryBean接口其实会有三个方法，分别是:

```java
T getObject() throws Exception;

Class<?> getObjectType();

default boolean isSingleton() {
		return true;
}
```

方法一:getObject()，被重写后，在方法中进行对象的创建并返回

方法二:getObjectType(),被重写后，主要返回的是被创建类的Class对象

方法三:没有被重写，因为它已经给了默认值，从方法名中可以看出其作用是设置对象是否为单例，默认true

# Spring整合JUnit

下面是一个测试类示例，前两行是固定格式

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfig.class)
public class AccountServiceTest {
	@Autowired
	private AccountService accountService;
	@Test
	public void testFindById(){
		System.out.println(accountService.findById(2));
    }
}
```

# Spring事务

## 案例：银行账户转账

### 在业务层接口上添加Spring事务管理

Spring注解式事务通常添加在业务层接口中而不会添加到业务层实现类中，降低耦合
注解式事务可以添加到业务方法上表示当前方法开启事务，也可以添加到接口上表示当前接口所有方法开启事务

```java
public interface Accountservice {
	@Transactional
	public void transfer(String out,String in Double money);
}
```

### 设置事务管理器

事务管理器要根据实现技术进行选择
MyBatis框架使用的是]DBc事务

```java
@Bean
public PlatformTransactionManager transactionManager(DataSource dataSource){
	DataSourceTransactionManager ptm new DataSourceTransactionManager();//JDBC的事务
	ptm.setDataSource(dataSource);
	return ptm;
}
```

### 开启注解式事务驱动

```java
@Configuration
@ComponentScan("com.itheima")
@PropertySource("classpath:jdbc.properties")
@Import({JdbcConfig.class,MybatisConfig.class})
@EnableTransactionManagement//这个
public class SpringConfig{
}
```

## 事务相关配置

![image-20230107110345664](images/image-20230107110345664.png)

### 事务只有出现了error或运行时异常才会回滚

`IOException`就不属于以上两种

有些异常就是这种默认不加入回滚的，所以我们要手动给他加上

```java
public interface Accountservice {
	@Transactional(rollbackFor ={IOException.class})//在这里
	public void transfer(String out,String in ,Double money)throws IOException;
}
```

## 事务传播行为

![image-20230107111458005](images/image-20230107111458005.png)

比方说我们银行转账时，无论转账是否成功都要记录操作情况

```java
public interface LogService{
	@Transactional(propagation = Propagation.REQUIRES_NEW)//注意这行
	void log(String out,String in,Double money);
}
```

```java
@Service
public class LogServiceImpl implements LogService{
	@Autowired
	private LogDao logDao;
	//@Transactional(propagation Propagation.REQUIRES_NEW)//接口写了,实现类就不用写了
	public void log(String out,String in,Double money){
		logDao.log("转账操作由"+out+"到"+in+",金额："+money);
    }
}
```

```java
@Service
public class AccountServiceImpl implements AccountService {
	@Autowired
	private AccountDao accountDao;
	@Autowired
	private LogService logService;
    
	public void transfer(String out,String in Double money){
		try{
			accountDao.outMoney(out,money);
			int i 1/0;
			accountDao.inMoney(in,money);
		}finally{
			logService.log(out,in,money);
    	}
	}
}
```

# SpringMVC

## 入门案例

1. 导坐标

   ```xml
   <dependency>
   	<groupId>javax.servlet</groupId>
   	<artifactId>javax.servlet-api</artifactId>
   	<version>3.1.0</version>
   	<scope>provided</scope>
   </dependency>
   <dependency>
   	<groupId>org.springframework</groupId>
   	<artifactId>spring-webmvc</artifactId>
   	<version>5.2.10.RELEASE</version>
   </dependency>
   ```

2. 创建SpringMVC控制器类（等同于Servlet功能）

   ```java
   @Controller
   public class UserController {
   	@RequestMapping("/save")
   	@ResponseBody
   	public String save(){
   		System.out.println("user save ...")
   		return "{'info':'springmvc'}";
       }
   }
   ```

3. 初始化SpringMVC环境（同Spring环境），设定SpringMVC加载对应的bean

   ```java
   @Configuration
   @ComponentScan("com.itheima.controller")
   public class SpringMvcConfig {
   }
   ```

4. 初始化Servlet容器，加载SpringMVC环境，并设置SpringMVC技术处理的请求(见下)

## bean的加载格式

```java
public class ServletContainersInitConfig extends AbstractDispatcherServletInitializer{
    //加载SpringMVC配置
	protected WebApplicationcontext createservletApplicationcontext(){
		AnnotationConfigWebApplicationContext ctx new AnnotationConfigWebApplicationContext();
		ctx.register(SpringMvcConfig.class);
		return ctx;
    }
    //加载Spring配置
	protected WebApplicationcontext createRootApplicationcontext(){
		AnnotationConfigWebApplicationContext ctx new AnnotationConfigWebApplicationContext();
		ctx.register(SpringConfig.class);
		return ctx;
    }
    //把所有请求交给SpringMVC处理
	protected String[]getservletMappings(){
		return new string[]{"/"};
    }
}
```

继承ADSI的子类更加简化开发

```java
public class ServletContainersInitConfig extends AbstractAnnotationConfigDispatcherServletInitializer {
    
	protected class<?>[]getRootConfigclasses() {
		return new class[]{SpringConfig.class};
    }
    
	protected Class<?>[]getServletConfigclasses() {
		return new class[]{SpringMvcConfig.class};
    }
    
	protected String[]getservletMappings(){
		return new String[]{"/"};
    }
}
```

对于Spring配置类加载，直接精准扫service和dao

下面是Main

```java
public class App {
	public static void main(String[]args){
		AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
		System.out.println(ctx.getBean(UserController.class));
    }
}
```

## 请求与响应-请求

### 参数名不一致的简单类型

```java
@RequestMapping("/commonParamDifferentName")
@ResponseBody
//请求参数名和形参不一样，就用@RequestParam，把请求里面的name给到username,或者说是绑定参数关系
public String commonParamDifferentName(@RequestParam("name")String userName int age){
	System.out.print1n("普通参数传递userName=>"+userName);
	System.out.printIn("普通参数传递age=>"+age);
	return "{'module':common param different name'}";
}
```

### 实体类

**首先给你的pojo重写toString！！！**

如果形参是个实体类，并且传的属性名和实体类的属性名一样，它就会自动被打包成一个对象

如果你要给实体类里面的对象传值，就用“加点”的方式传

![image-20230107205522019](images/image-20230107205522019.png)



### 数组

如果传的是数组，那就是看谁的key和形参名的一样，自动打包

![image-20230107205901324](images/image-20230107205901324.png)

### List

SpringMVC对引用类型的态度是：当成一个pojo(实体类对象)，所以造一个对象，然后把对应属性值塞进去

但是对List不管用所以只要在前面加个`@RequestParam`注解，告诉它把参数都扔进去就行

```java
//集合参数
@RequestMapping("/listParam")
@ResponseBody
public String listParam(@RequestParam List<String>likes){
	System.out.printin("集合参数传递likes=>"+likes);
	return "{'module':'list param'}";
}
```

### JSON

```xml
<dependency>
	<groupId>com.fasterxml.jackson.core</groupId>
	<artifactId>jackson-databind</artifactId>
	<version>2.9.0</version>
</dependency>
```



![image-20230107210927583](images/image-20230107210927583.png)

#### 开启由JSON数据自动类型转化为对象的功能

`@EnableWebMvc`功能之一：根据类型匹配对应的类型转换器

```java
@Configuration
@ComponentScan("com.itheima.controller")
@EnableWebMvc//就是这一句
public class SpringMvcConfig {
}
```

`@RequestParam`用不了了，得用`@RequestBody`把JSON数据塞进去

```java
//集合参数：json格式
@RequestMapping("/listParamForJson")
@ResponseBody
public String listParamForJson(@RequestBody List<String>likes){
	System.out.printin("1 ist common(json)参数传递list=>"+likes);
	return "{'module':'list common for json param'}";
}
```

当然，如果形参是实体类，也是加`@RequestBody`就会自动打包

#### 日期类型参数传递

接收形参时，根据不同的日期格式设置不同的接收方式

`@DateTimeFormat(pattern = "??????")`

```java
@RequestMapping("/dataParam")
@ResponseBody
public String dataParam(Date date,
                        @DateTimeFormat(pattern = "yyyy-MM-dd")Date date1,
                        @DateTimeFormat(pattern = "yyyy/MM/dd HH:mm:ss")Date date2){
	System.out.println("参数传递date=>"+date);
	System.out.println("参数传递date(yyyy-MM-dd)==>"+date1);
	System.out.println("参数传递date(yyyy/MM/ddHH:mm:ss)=>"+date2);
	return "{'module':'data param'}";
}
```

## 请求与响应-响应

### 响应页面/跳转页面

```java
@RequestMapping("/toJumpPage")
public String toJumpPage(){
	System.out.printin("跳转页面");
	return "page.jsp";
}
```

### 响应文本数据

`@ResponseBody`设置当前控制器返回值作为响应体，不加的话它会以为这是个网页，参考上例

如果返回值是String，就当做响应体，如果是对象，就通过jackson转成可识别的数据再响应回去

```java
@RequestMapping("/toText")
@ResponseBody
public String toText(){
	System.out.println("返回纯文本数据")；
	return "response text";
}
```

### JSON-响应POJO对象

加上`@ResponseBody`，然后直接返回对象，就会自动转JSON

```java
@RequestMapping("/toJsonPOJO")
@ResponseBody
public User toJsonPOJO(){
	System.out.println("返回json对象数据")；
	Useruser new User();
	user.setName("itcast");
	user.setAge(15);
	return user;
}
```

但转JSON这事可不是SpringMVC帮我们做的，而是jackson

```xml
<dependency>
	<groupId>com.fasterxml.jackson.core</groupId>
	<artifactId>jackson-databind</artifactId>
	<version>2.9.0</version>
</dependency>
```

### JSON-响应POJO集合对象

和上面一样

```java
@RequestMapping("/to]sonList")
@ResponseBody
public List<User>toJsonList(){
	System.out.println("返回json集合数据")；
	User user1 new User();I
	user1.setName("传智播客")；
	user1.setAge(15);
	User user2 = new User();
	user2.setName("黑马程序员")；
	user2.setAge(12);
	List<User>userList new ArrayList<~>();
	userList.add(user1);
	userList.add(user2);
	return userList;
}
```

 ## REST风格

按照REST风格访问资源时使用行为动作区分对资源进行了何种操作

| 访问路径                 | 行为             | 请求方式        |
| ------------------------ | ---------------- | --------------- |
| http://localhost/users   | 查询全部用户信息 | GET(查询)       |
| http://localhost/users/1 | 查询指定用户信息 | GET(查询)       |
| http://localhost/users   | 添加用户信息     | POST(新增/保存) |
| http://localhost/users   | 修改用户信息     | PUT(修改/更新)  |
| http://localhost/users/1 | 删除用户信息     | DELETE(删除)    |

根据REST风格对资源进行访问称为RESTful

**描述摸块的名称通常使用复数**，也就是加s的格式描述，表示此类资源，而非单个资源，例如：users、books、accounts.…

加个`@PathVariable`代表变量来自访问路径（占位），然后还要用`{}`标明路径变量，告诉哪个值给谁

```java
//{id}是路径变量，下面括号先写("/users/{id}")，然后加上,method，idea会自动把前面的value=补上
@RequestMapping(value = "/users/{id}",method = RequestMethod.DELETE)
@ResponseBody
public String delete(@PathVariable Integer id){
System.out.println("user delete..."+id);
return "{'module':'user delete'}";
```

### `@RequestBody` `@RequestParam` `@PathVariable`
区别

* `@RequestParam`用于接收url地址传参或表单传参
* `@RequestBody`用于接收json数据
* `@PathVariable`用于接收路径参数，使用{参数名称)描述路径参数

应用

* 后期开发中，发送请求参数超过1个时，以封装成pojo然后采用json格式为主，`@RequestBody`应用较广
* 如果发送非json格式数据，选用`@RequestParam`接收请求参数
* 采用RESTful进行开发，当参数数量较少时，例如1个，可以采用`@PathVariable`接收请求路径变量，通常用于传递id值

### 优化

我们学到现在，一个Controoler是这样的

```java
@Controller
public class BookController {
	@RequestMapping(value = "/books", method = RequestMethod.POST)
	@ResponseBody
	public String save(@RequestBody Book book){
		System.out.println("book save..."+book);
		return "{'module':'book save'}";
    }
	@RequestMapping(value ="/books/{id)",method = RequestMethod.DELETE)
	@ResponseBody
	public String delete(@PathVariable Integer id){
		System.out.println("book delete..."+id);
		return "{'module':'book delete'}";
    }
	@RequestMapping(value = "/books", method = RequestMethod.PUT)
	@ResponseBody
	public String update(@RequestBody Bookbook){
		System.out.println("book update..."+book);
		return "{'module':'book update'}";
    }
}
```

显然我们在每个方法中都有重复的地方：`@ResponseBody`,`value = "/books", `

我们可以像之前JavaWeb优化servlet那样，同时用上更方便的注解

```java
//@Controller
//@ResponseBody
@RestController//其实就是上面2个的合二为一
@RequestMapping("/books")
public class BookController {
    
	//@RequestMapping(method = RequestMethod.POST)
    @PostMapping
	public String save(@RequestBody Book book){
		System.out.println("book save..."+book);
		return "{'module':'book save'}";
    }
    
	//@RequestMapping(value = "/{id}", method = RequestMethod.DELETE)
    @DelectMapping("/{id}")
	public String delete(@PathVariable Integer id){
		System.out.println("book delete..."+id);
		return "{'module':'book delete'}";
    }
    
	//@RequestMapping(method = RequestMethod.PUT)
	@PutMapping
    public String update(@RequestBody Bookbook){
		System.out.println("book update..."+book);
		return "{'module':'book update'}";
    }
}
```

### 放行非SpringMVC的请求

我们前面设置了`SpringConfig`下的`getServletMappings`方法把所有资源拉给SpringMVC处理，但这会造成静态资源如html，css，图片等资源也被拉去SpringMVC而非交给Tomcat处理

所以我们要添加一个资源的过滤

```java
//记得在SpringConfig那边@Import一下
public class SpringMvcSupport extends WebMvcConfigurationSupport {
@Override
protected void addResourceHandlers(ResourceHandlerRegistry registry){
    //给page目录添加一个资源的处理器，当访问/pages/????的时候，走/pages目录下的内容
	registry.addResourceHandler("/pages/**").addResourceLocations("/pages/");
	registry.addResourceHandler("/js/**").addResourceLocations("/pages/");
	registry.addResourceHandler("/css/**").addResourceLocations("/pages/");
	registry.addResourceHandler("/plugins/**").addResourceLocations("/pages/");
}
```

