# 核心思想：充分解耦

* IoC : 由主动new产生对象转换为由外部提供对象，此过程中对象创建控制权由程序转移到外部，此思想称为控制反转

* bean：Ioc容器负责对象的创建、初始化等一系列工作，被创建或被管理的对象在IoC容器中统称为Bean

* IOC容器：管理bean，在IoC容器内将有依赖关系的bean进行关系绑定
* 效果：使用对象时不仅可以直接从Ioc容器中获取，并目获取到的bea已经绑定了所有的依赖关系 

# 注解开发

## SpringConfig

`@Configuration`

`@ComponentScan("com.orion")`,配置多个扫描包时用数组：`@ComponentScan({"com.orion.service","com.orion.dao"})`

## 自动装配：(括号内为示例)

### 引用类型：@Autowired和@Qualifier("bookDao2")

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
@Import({JdbcConfig.class})//多个请用数组，单个不用加大括号也行
public class SpringConfig{
}
```

```java
public class JdbcConfig(){
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
public class JdbcConfig(){
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
