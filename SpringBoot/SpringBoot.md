# 配置

引导类扫包范围是：引导类所在包及其子包

配置中引用配置：${引用的配置名字(.引用的配置下面的子配置名字)}



把一组配置封装到一个对象中给类读取：

比方说我想把DataSource的配置封装到一个对象

```yaml
spring:
  datasource:
    # 注意下面这里
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql:///volunteer_management_activity_system
    username: root
    password: 123456
    type: com.alibaba.druid.pool.DruidDataSource
```

那就创建一个类，啥名无所谓,**但是属性名得一样！**

```java
@Data
@Component//定义为Spring管控的bean
@ConfigurationProperties(prefix = "spring.datasource")//指定加载的数据
/*
	点进ConfigurationProperties后发现属性value和prefix互为别名，所以这是前缀
*/
public class MyDataSource {
    private String driverClassName;//配置中划线分隔单词的就用驼峰命名法
	private String url;
	private String username;
	private String password;
}
```

然后在你要的类里面自动注入`MyDataSource`



# 整合JUnit

1. 测试类如果存在于引导类所在包或子包中无需指定引导类
2. 测试类如果不存在于引导类所在的包或子包中需要通过c1asses属性指定引导类
3. 

# 整合Mybatis

1. 创建工程时把MySQL和Mybatis勾上

2. 在`application.yml`中

   ```yaml
   spring:
     datasource:
       driverClassName: com.mysql.cj.jdbc.Driver
       url: jdbc:mysql:///volunteer_management_activity_system
       username: root
       password: 123456
   ```

3. 用注解写Dao层接口时，在接口上面加个`@Mapper`让容器识别到，产生自动代理的对象

4. 没了

MySQL8在低版本springboot会爆错，得把时区加上

```yaml
spring:
  datasource:
    driverClassName: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql:///volunteer_management_activity_system?serverTimezone=UTC
    username: root
    password: 123456
```



# 整合Mybatis Plus

1. 导坐标

   1. 法一：服务器URL改成`https://start.aliyun.com`（但是SpringBoot的版本可能会有点低），然后去“关系型数据库”中找

   2. 法二：手动导坐标

      ```java
      <dependency>
          <groupId>com.baomidou</groupId>
          <artifactId>mybatis-plus-boot-starter</artifactId>
          <version>3.5.3.1</version>
      </dependency>
      ```

      

2. ```java
   @Mapper
   //MP和Mybatis差别就在这：不用你手写
   public interface BookDao extends BaseMapper<Book> {
   }
   ```

3. 表名要和类名一致，不一致的（比如表前有tbl_前缀）

   ```yaml
   mybatis-plus: 
     global-config: 
       db-config: 
   	  table-prefix: tbl_
   ```


4. 默认id是雪花算法，但我们想要让他按照数据库自增来

   ```yaml
   mybatis-plus:
     global-config:
       db-config:
         id-type: auto
   ```

5. 用分页查询要配置mybatis-plus的分页拦截器

   ```java
   @Configuration
   public class MPConfig{
   	@Bean
   	public MybatisPlusInterceptor mybatisPlusInterceptor(){
   		MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
   		interceptor.addInnerInterceptor(new PaginationInnerInterceptor());
   		return interceptor;
   	}
   }
   ```

   > 在启动类所在包及其子包下面的`@Configuration`和标注的bean都会被加载

6. 开始查询

   ```java
   @Test
   void testGetBy2(){
   	String name = "1";
   	LambdaQueryWrapper<Book>lqw new LambdaQueryWrapper<Book>();
   	lqw.like(name != null, Book::getName, name);
   	bookDao.selectList(lqw);
   }
   ```

   

# 整合Druid

1. 是`Druid Spring Boot Starter`不是`Druid`!

   ```xml
   <dependency>
   	<groupId>com.alibaba</groupId>
   	<artifactId>druid-spring-boot-starter</artifactId>
   	<version>1.2.15</version>
   </dependency>
   ```

   

2. * 法一：**不推荐**，但你可以只导Druid不导那个Spring Boot专用的那个

     ```yaml
     spring:
       datasource:
         driver-class-name: com.mysql.cj.jdbc.Driver
         url: jdbc:mysql:///volunteer_management_activity_system
         username: root
         password: 123456
         type: com.alibaba.druid.pool.DruidDataSource
     ```
     
   * 法二：

     ```yaml
     # 先打个druid
     spring:
       datasource:
         druid:
           driver-class-name: com.mysql.cj.jdbc.Driver
           url: jdbc:mysql:///volunteer_management_activity_system
           username: root
           password: 123456
     ```

3. 报错：`o.s.b.d.LoggingFailureAnalysisReporter   : `

   启动类注解改成：`@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})`

   [ERROR：o.s.b.d.LoggingFailureAnalysisReporter解决办法](https://blog.csdn.net/qq_37887131/article/details/89705595)

![image-20230211011321999](images/image-20230211011321999.png)

Integer，如果判断条件加上`approvalStatus!=''`，传0进来approvalStatus语句不生效

# 配置数据读取

## 回顾：自定义bean属性绑定

application.yml

```yaml
servers:
	ipAddress: 192.168.0.1
	port: 2345
	timeout: -1
```

某个pojo

```java
@Component
@Data
@ConfigurationProperties(prefix="servers")
public class ServerConfig {
	private String ipAddress;
	private int port;
	private long timeout;
}
```



## 第三方bean属性绑定

```yaml
datasource:
	driverclassName: com.mysq1.jdbc.Driver456
```

```java
@Bean
@ConfigurationProperties(prefix="datasource")
public DruidDataSource datasource(){
	DruidDataSource ds new DruidDataSource();
	return ds;
}
```



## @EnableConfigurationProperties

**`@EnableConfigurationProperties`注解可以将使用`@ConfigurationProperties`注解对应的类加入Spring容器，但由于配置时强制自动把目标注册成bean管理，所以`@EnableConfigurationProperties`与`@Component`不能同时使用**

```java
@SpringBootApplication
@EnableConfigurationProperties(ServerConfig.class)
public class DemoApplication{
    
}
```

```java
//@Component
@Data
@ConfigurationProperties(prefix="servers")
public class ServerConfig{
    
}
```



## 报错：未配置Spring Boot配置注释处理器

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-configuration-processor</artifactId>
</dependency>
```



## (有且只有)`@ConfigurationProperties`绑定属性支持属性名宽松绑定

**同时也是springboot推荐的做属性绑定的方式**

不管yml那些配置文件是驼峰命名，还是下划线、中划线、常量命名，spring都可以根据模型类的属性名灵活适应

对于下面的注解

```java
@ConfigurationProperties(prefix="dataSource")
```

spring会报错

```markdown
Invalid characters:'S'
Bean:datasource
Reason:Canonical names should be kebab-case ('-separated),lowercase alpha-numeric characters and must start with a letter
```

翻译过来就是典型的名字应该是烤肉串命名法(中划线分隔)，小写字母、数字和下划线，并且必须以字母开头。



## 计量单位(JDK8开始)

### 时间计量单位

```java
@DurationUnit(ChronoUnit.MINUTES)
private Duration serverTimeOut;
```

### 存储空间计量单位

```java
@DatasizeUnit(DataUnit.MEGABYTES)
private DataSize datasize;
```

> 你不加这个注释直接在yml写10MB也是能读的



## 格式校验

### 先导坐标

就像jdbc也是一种规范,在我们学习中以MySQL驱动实现，写格式校验先导个JSR303校验规范

```xml
<dependency>
	<groupId>javax.validation</groupId>
	<artifactId>validation-api</artifactId>
</dependency>
```

使用hibernate框架提供的校验器做实现类

```xml
<dependency>
	<groupId>org.hibernate.validator</groupId>
	<artifactId>hibernate-validator</artifactId>
</dependency>
```

前置工作完成，下面是一个示例

### 开`@Validated`，设置具体的规则

```java
@Data
@ConfigurationProperties(prefix "servers")
//2.开启对当前bean的属性注入校险功能
@Validated
public class ServerConfig{
	//3.设置具体的规则
	@Max(value=8888, message="最大值不能超过8888")
	@Min(value=202, message="最小值不能低于202")
	private int port;
}
```



其他hibernate的使用注解

`@NotEmpty`, ...



## 进制数据转换规则: yml中纯数字建议使用双引号包裹

### 回顾：ymal语法规则

字面值表达方式

```yaml
# TRUE,true,True,FALSE,false,False均可
boolean: TRUE

# 6.8523015e+5#支持科学计数法
f1oat: 3.14

# 0b1010011101001010111
# 支持二进制、八进制、十六进制
# 8进制以0开头
# 16进制0x开头
int: 123

# 使用~表示muLL
nu11: ~

# 字符串可以直接书写
string: HelloWorld

# 可以使用双引号包裹特殊字符
string2: "Hello World"

# 日期必须使用yyyy-MM-dd格式
date: 2018-02-17

# 时间和日期之间使用T连接，最后使用+代表时区
datetime: 2018-02-17T15:02:31+08:00
```

所以当yml中有`password = 0127`，实际读取到的会变成87，正确的输入方式是`password = "0127"`



# 测试

## 加载测试临时属性值(常用)：properties

```yaml
test:
	prop: 666
```

```java
@SpringBootTest(properties = {"test.prop=777"})
public class PropertiesAndArgsTest {
	@Value("${test.prop}")
	private String msg;
    
	@Test
	void testProperties(){
		System.out.println(msg);
	}
}
```

## 模拟命令行参数：args

```java
@SpringBootTest(args={"--test.prop=testValue2"})
```

## 加载测试专用配置

是在测试类上面加！

请注意：这里的MsgConfig是在test包下的，不是在main包下的！  

```java
@SpringBootTest
@Import({MsgConfig.class})
```

## 加载环境，模拟请求

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
//开启虚拟MVC调用
@AutoConfigureMockMvc
public class WebTest{
	@Test
	//注入虚拟MVC调用对象
	public void testWeb(@Autowired MockMvc mvc)throws Exception {
		//创建虚拟请求，当前访问/books
		MockHttpServletRequestBuilder builder = MockMvcRequestBuilders.get("/books");
		//执行请求
		ResultActions action = mvc.perform(builder);
    }
}
```

web环境四种启动类型：

* Mock
* DEGINED_PORT：用你定义的端口
* RANDON_PORT：随机端口
* NONE：默认的启动方式：啥都不干

## 测试类中发送请求

```java
@Test
void testGetById(@Autowired MockMvc mvc) throws Exception {
    MockHttpServletRequestBuilder builder = MockMvcRequestBuilders.get("/books");
    ResultActions action = mvc.perform(builder);
    
    //响应状态匹配
    //设定预期值 与真实值进行比较，成功测试通过，失败测试失败
    //定义本次调用的预期值
    StatusResultMatchers status = MockMvcResultMatchers.status();
    ResultMatcher ok = status.isOk();
    //添加预计值到本次调用过程中进行匹配
    action.andExpect(ok);

    //响应头信息匹配
    //设定预期值 与真实值进行比较，成功测试通过，失败测试失败
    //定义本次调用的预期值
    HeaderResultMatchers header = MockMvcResultMatchers.header();
    ResultMatcher contentType = header.string("Content-Type", "application/json");
    //添加预计值到本次调用过程中进行匹配
    action.andExpect(contentType);

    //响应体匹配
    //设定预期值 与真实值进行比较，成功测试通过，失败测试失败
    //定义本次调用的预期值
    ContentResultMatchers content = MockMvcResultMatchers.content();
    ResultMatcher result = content.json("{\"id\":1,\"name\":\"springboot\",\"type\":\"springboot\"}");
    //添加预计值到本次调用过程中进行匹配
    action.andExpect(result);
}
```

## 业务层做测试

**只有InnoDB才支持事务！**

可以在测试类上面加`@Transactional`进行事务管理，自动回滚所有操作，防止在数据库留痕，此时`@Rollback()`括号中默认为true

## 随机值测试

对于测试用例的数据固定书写肯定是不合理的，springboot提供了在配置中使用随机值的机制，确保每次运行程序加载的数据都是随机的。具体如下：

```yaml
testcase:
  book:
    id: ${random.int}
    id2: ${random.int(10)}
    type: ${random.int!5,10!}
    name: ${random.value}
    uuid: ${random.uuid}
    publishTime: ${random.long}
```



当前配置就可以在每次运行程序时创建一组随机数据，避免每次运行时数据都是固定值的尴尬现象发生，有助于测试功能的进行。数据的加载按照之前加载数据的形式，使用@ConfigurationProperties注解即可

```JAVA
@Component
@Data
@ConfigurationProperties(prefix = "testcase.book")
public class BookCase {
    private int id;
    private int id2;
    private int type;
    private String name;
    private String uuid;
    private long publishTime;
}
```



对于随机值的产生，还有一些小的限定规则，比如产生的数值性数据可以设置范围等，具体如下：

- ${random.int}表示随机整数
- ${random.int(10)}表示10以内的随机数
- ${random.int(10,20)}表示10到20的随机数
- 其中()可以是任意字符，例如[]，!!均可

**总结**

# Spring(Boot)内置

## 数据源：默认HikariCP实现

轻量级，特别快

## 持久化技术：JDBCTemPlate

古法编程，跳过了

## 数据库：H2

极为轻巧，直接在内存中运行，特别适合在测试中用



# NoSQL

