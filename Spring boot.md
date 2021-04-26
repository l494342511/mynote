# Spring boot

## 1. Spring boot项目创建

![image-20210119123828384](Spring%20boot.assets/image-20210119123828384-1615170974861.png)

## 2. spring boot的运行机制

![image-20210119133544011](Spring%20boot.assets/image-20210119133544011-1615170977262.png)

### 2.1 自动装配实现机制

**依赖管理**

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.4.2</version>
  </parent>
```

有了这个父依赖，以后**导入依赖默认是不需要写版本**；但是如果导入的包没有在依赖中管理就需要重新配置版本了。

**配置yml的地方**

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.2</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

### 2.2 启动器

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

**spring-boot-starter-xxx**：是spring-boot的场景启动器

**spring-boot-starter-web**：帮我们导入了web模块正常运行所依赖的组件；

SpringBoot将所有的功能场景都抽取出来，做成一个个的starter （启动器），只需要在项目中引入这些starter即可，所有相关的依赖都会导入进来 ， 我们要用什么功能就导入什么样的场景启动器即可 ；我们未来也可以自己自定义 starter；

### 2.3 主启动器

#### 2.3.1 spring boot的主启动类

```java
@SpringBootApplication
public class SpringBoot01Application {
    public static void main(String[] args) {
        SpringApplication.run(SpringBoot01Application.class, args);
    }
}
```

该类必须和controller在一个包下才能扫描到controller

**注解**

```java
@SpringBootApplication
	@SpringBootConfiguration：spring boot的配置
		@Configuration：说明是一个spring配置类
		@Component：说明是一个spring的组件
	@EnableAutoConfiguration：自动配置
		@AutoConfigurationPackage：自动配置包
			Import(AutoConfigurationPackages.Registrar.class)：自动配置`包注册`
		@Import(AutoConfigurationImportSelector.class)
			getAutoConfigurationEntry(annotationMetadata)：获得在自动配置的实体
			getCandidateConfigurations(annotationMetadata, attributes)：获取候选的配置
				//从标注了@EnableAutoConfiguration的类中获取
				protected Class<?> getSpringFactoriesLoaderFactoryClass() {return EnableAutoConfiguration.class;}
					Enumeration<URL> urls = classLoader.getResources(FACTORIES_RESOURCE_LOCATION);//项目资源
						public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";//从这获取配置
							@ConditionalOnXXX//核心注解，只有导入了相应的start配置才会起作用
					Properties properties = PropertiesLoaderUtils.loadProperties(resource);//封装成properties供使用
```

主启动类会从spring.factories获取配置，spring.factories中有很多xxxAutoConfiguration类，这些类一旦生效就会为容器添加很多组件

一个AutoConfiguration类

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ DispatcherHandler.class, HttpHandler.class })
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.REACTIVE)
@ConditionalOnMissingBean(HttpHandler.class)
@AutoConfigureAfter({ WebFluxAutoConfiguration.class })
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
public class HttpHandlerAutoConfiguration {

   @Configuration(proxyBeanMethods = false)
   public static class AnnotationConfig {

      private final ApplicationContext applicationContext;

      public AnnotationConfig(ApplicationContext applicationContext) {
         this.applicationContext = applicationContext;
      }

      @Bean
      public HttpHandler httpHandler(ObjectProvider<WebFluxProperties> propsProvider) {
         HttpHandler httpHandler = WebHttpHandlerBuilder.applicationContext(this.applicationContext).build();
         WebFluxProperties properties = propsProvider.getIfAvailable();
         if (properties != null && StringUtils.hasText(properties.getBasePath())) {
            Map<String, HttpHandler> handlersMap = Collections.singletonMap(properties.getBasePath(), httpHandler);
            return new ContextPathCompositeHandler(handlersMap);
         }
         return httpHandler;
      }

   }

}
```

AutoConfiguration类通过properties获取配置属性。该类对应的properties如下

```java
@ConfigurationProperties(prefix = "spring.webflux")
public class WebFluxProperties {
   private String basePath;
   private final Format format = new Format();
   //
}
```

该properties与主配置文件中的spring.webflux前缀绑定，可以在主配置文件中对这些属性进行配置，不配置的时候使用默认值

#### 2.3.2 @SpringBootApplication

申明某类是spring-boot的主配置类，会运行该类的main方法来启动spring-boot项目

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    // ......
}
```

#### 2.3.3 @ComponentScan

自动扫描并加载符合条件的组件或者bean，然后装载到IOC容器中

#### 2.3.4 @SpringBootConfiguration

```java
// 点进去得到下面的 @Component
@Configuration
public @interface SpringBootConfiguration {}

@Component
public @interface Configuration {}
```

@Configuration，说明这是一个配置类 ，配置类就是对应Spring的xml 配置文件；

@Component 这就说明，启动类本身也是Spring中的一个组件而已，负责启动应用！

#### 2.3.5 @EnableAutoConfiguration

以前我们需要自己配置的东西，而现在SpringBoot可以自动帮我们配置 ；@EnableAutoConfiguration告诉SpringBoot开启自动配置功能，这样自动配置才能生效；

#### 2.3.6 AutoConfigurationPackage

```java
@Import({Registrar.class})
public @interface AutoConfigurationPackage {
}
```

**@import** ：Spring底层注解@import ， 给容器中导入一个组件

**Registrar.class** ：将主启动类的所在包及包下面所有子包里面的所有组件扫描到Spring容器 ；

## 3. Spring boot注入方式

spring boot注入有三种方法：

1. 通过注解注入
2. 通过配置yml文件注入属性
3. 通过占位符和properties文件注入属性

### 3.1 通过注解注入

```java
@Component
public class Dog {
    
    @Value("小狗比")
    private String name;
    
    @Value("1")
    private Integer age;
}
```

通过@Value指定属性值，不需要构造器和setter方法

@Component指定该类为spring的一个组件，才能被spring boot管理

### 3.2 通过yml文件注入属性

YAML是 "YAML Ain't a Markup Language" （YAML不是一种标记语言）的递归缩写。在开发的这种语言时，YAML 的意思其实是："Yet Another Markup Language"（仍是一种标记语言）

```java
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private Integer age;
    private Boolean happy;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
    //setter方法
}
```

**@ConfigurationProperties**：指定yml中对应的配置名

```yml
person:
  name: ljq
  age: 18
  happy: false
  birth: 1998/03/02
  maps: {k1: v1,k2: v2}
  lists:
    code
    boy
    music
  dog:
    name: 狗
    age: 1
```

### 3.3 通过占位符和properties注入属性

```java
@Component
@PropertySource(value = "classpath:person.properties")
public class Person {
    @Value("${name}")
    private String name;
    private Integer age;
    private Boolean happy;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
    //setter方法
}
```

该方法将属性值写在properties文件里，通过占位符注入。

**@PropertySource**: 指定了对应的properties文件

```properties
name=卢嘉祺
```

**properties**语法：key=data

## 4. 数据校验与多环境切换

### 4.1 数据校验

spring boot中可以使用@validated来校验数据，如果数据异常会统一抛出异常，在spring boot 2.3以后，validated需要手动导入

```xml
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <scope>compile</scope>
</dependency>
```

常用参数

```java
@NotNull(message="名字不能为空")
private String userName;
@Max(value=120,message="年龄最大不能查过120")
private int age;
@Email(message="邮箱格式错误")
private String email;

空检查
@Null       验证对象是否为null
@NotNull    验证对象是否不为null, 无法查检长度为0的字符串
@NotBlank   检查约束字符串是不是Null还有被Trim的长度是否大于0,只对字符串,且会去掉前后空格.
@NotEmpty   检查约束元素是否为NULL或者是EMPTY.
    
Booelan检查
@AssertTrue     验证 Boolean 对象是否为 true  
@AssertFalse    验证 Boolean 对象是否为 false  
    
长度检查
@Size(min=, max=) 验证对象（Array,Collection,Map,String）长度是否在给定的范围之内  
@Length(min=, max=) string is between min and max included.

日期检查
@Past       验证 Date 和 Calendar 对象是否在当前时间之前  
@Future     验证 Date 和 Calendar 对象是否在当前时间之后  
@Pattern    验证 String 对象是否符合正则表达式的规则

.......等等
除此以外，我们还可以自定义一些数据校验规则
```



### 4.2 多环境切换

profile是Spring对不同环境提供不同配置功能的支持，可以通过激活不同的环境版本，实现快速切换环境；

#### 4.2.1 多配置文件

如果使用properties文件来配置多环境的话，需要使用不同的properties文件来配置不同的环境，然后交给主配置文件管理

**例如：**

application-test.properties 代表测试环境配置

application-dev.properties 代表开发环境配置

**application.properties**配置使用的环境

```xml
#比如在配置文件中指定使用dev环境，我们可以通过设置不同的端口号进行测试；
#我们启动SpringBoot，就可以看到已经切换到dev下的配置了；
spring.profiles.active=dev
```

#### 4.2.2 yaml的多文档块

使用yml实现多环境不需要多个配置文件

```yml
server:
  port: 8081
#选择要激活那个环境块
spring:
  profiles:
    active: prod

---
server:
  port: 8083
spring:
  profiles: dev #配置环境的名称


---

server:
  port: 8084
spring:
  profiles: prod  #配置环境的名称
```

### 4.3 配置文件的加载路径

![image-20210119154004475](C:\Users\ljq\AppData\Roaming\Typora\typora-user-images\image-20210119154004475.png)

```
优先级1：项目路径下的config文件夹配置文件
优先级2：项目路径下配置文件
优先级3：资源路径下的config文件夹配置文件
优先级4：资源路径下配置文件
```

## 5. 读源码

@Configuration：表示是一个配置类

@EnableConfigurationProperties：自动配置属性，指定 要配哪一个类

@ConfigurationProperties：绑定属性

```java
//spring的底层注解：根据不同的条件，来判断当前配置或者类是否生效
@ConditionalOnWebApplication
@ConditionalOnClass
@ConditionalOnProperties
```

## 6. spring-boot web应用

### 6.1 项目结构

![image-20210122164124185](Spring%20boot.assets/image-20210122164124185-1615170984554.png)

启动器应该和controller在一个包下，才能扫描到controller

### 6.2 访问web资源的方式

1. 通过webjars目录

   ![image-20210122165345129](Spring%20boot.assets/image-20210122165345129-1615170990318.png)

2. 静态路径

   ![image-20210122171453033](Spring%20boot.assets/image-20210122171453033-1615170993599.png)

   使用localhost:8080/**.js 会自动去这几个目录下找相应的js文件访问

   静态路径的优先级：resources>static>public

### 6.3 首页

1. 首页的html放在三个静态资源包下，命名为index，通过localhost:8080即可访问

2. 通过模板引擎thymeleaf访问

   引入依赖：

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-thymeleaf</artifactId>
   </dependency>
   ```

   该依赖等于

   ```xml
   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter</artifactId>
     <version>2.4.2</version>
     <scope>compile</scope>
   </dependency>
   <dependency>
     <groupId>org.thymeleaf</groupId>
     <artifactId>thymeleaf-spring5</artifactId>
     <version>3.0.12.RELEASE</version>
     <scope>compile</scope>
   </dependency>
   <dependency>
     <groupId>org.thymeleaf.extras</groupId>
     <artifactId>thymeleaf-extras-java8time</artifactId>
     <version>3.0.4.RELEASE</version>
     <scope>compile</scope>
   </dependency>
   ```

   将首页放于templates下，通过接口访问

   ![image-20210123154426985](Spring%20boot.assets/image-20210123154426985-1615170999266.png)

   ![image-20210123154032514](Spring%20boot.assets/image-20210123154032514-1615170997677.png)

## 7. spring boot 数据库的使用

### 7.1 使用Template使用JDBC

**引入依赖**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

**配置DataSource**

```yml
spring:
  datasource:
    username: root
    password: 666
    url: jdbc:mysql://localhost:3306/nuclear?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
    driver-class-name: com.mysql.cj.jdbc.Driver
```

mysql8需要设定时区

mysql8使用新的驱动：com.mysql.cj.jdbc.Driver

**使用**

```java
@Autowired
JdbcTemplate jdbcTemplate;

@GetMapping("/hello")
public List<Map<String,Object>> userList(){
    String sql = "select * from testname";
    List<Map<String,Object>> userList = jdbcTemplate.queryForList(sql);
    return userList;
}
```

### 7.2 使用Druid连接池

**引入依赖**

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.4</version>
</dependency>
```

**配置连接池属性**

```yml
spring:
  datasource:
    username: root
    password: 666
    url: jdbc:mysql://localhost:3306/nuclear?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource

    #spring boot 默认不注入这些属性值，需要自己绑定
    #druid 数据源专有配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMills: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true

    #配置监控统计拦截filters, stat：监控统计、log4j
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```

**管理Druid连接池**

Druid的私有属性不被spring boot管理，需要手动配置bean

```java
@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druidDataSource(){
        return new DruidDataSource();
    }
    
}
```

**配置后台监控和过滤器**

```java
//    后台监控
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean<StatViewServlet> statViewServletServletRegistrationBean = new ServletRegistrationBean<>(new StatViewServlet(),"/druid/*");
        HashMap<String,String> initPara = new HashMap<>();//后台账号密码设置
        initPara.put("loginUsername","root");
        initPara.put("loginPassword","123456");
        initPara.put("allow","");//允许用户列表
        statViewServletServletRegistrationBean.setInitParameters(initPara);//设置初始化参数
        return statViewServletServletRegistrationBean;
    }

    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
        filterRegistrationBean.setFilter(new WebStatFilter());
        Map<String,String> initPara = new HashMap<>();
        //这些东西不进行统计
        initPara.put("exclusion","*.js,*.css,/druid/*");
        filterRegistrationBean.setInitParameters(initPara);
        return filterRegistrationBean;
    }
```

**通过/druid访问后台**

![image-20210124163456349](Spring%20boot.assets/image-20210124163456349-1615171003458.png)

### 7.3 spring-boot整合mybatis

添加依赖

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.4</version>
</dependency>
```

等同于

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
  </dependency>
  <dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-autoconfigure</artifactId>
  </dependency>
  <dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
  </dependency>
  <dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
  </dependency>
</dependencies>
```

**创建mapper类**

```java
@Mapper
@Repository
public interface UserMapper {

}
```

可以通过@Mapper指定该类为mapper类

@Repository指定该类为spring的一个组件

也可以通过@MapperScan()让主启动类扫描mapper

```java
@SpringBootTest
@MapperScan("com.ljq.mapper")
class SpringBootMybatis01ApplicationTests {
}
```

**创建对应的mapper.xml**

命名空间要指定

```xml
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ljq.mapper.UserMapper">

</mapper>
```

**配置mapper与mapper.xml的匹配**

type-aliases-package：指定mapper.xml的实体包，如com.ljq.pojo.user=user

```yml
spring:
  datasource:
    username: root
    password: 666
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mytestdb?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
mybatis:
  type-aliases-package: com.ljq.pojo
  mapper-locations: classpath:mybatis/mapper/*.xml
```

**使用**

```java
@RestController
public class UserController {

    @Autowired
    private UserMapper userMapper;

    @GetMapping("/query")
    public List<Users> queryUserList(){
        return userMapper.queryUserList();
    }
}
```

## 