# MyBatis

## 1. 使用方法

### 1.1 直接使用mybatis

mybatis配置文件：配置连接方式，映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="aggressiveLazyLoading" value="false"/>
        <setting name="cacheEnabled" value="true"/>
    </settings>

    <typeAliases>
        <package name="org.example.pojo"/>
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://47.107.124.164:3306/db_smq?useSSL=false&amp;useUnicode=true&amp;characterEncoding=utf-8&amp;serverTimezone=GMT%2B8"/>
                <property name="username" value="root"/>
                <property name="password" value="666"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="sqlmap/UserMapper.xml"/>
    </mappers>
</configuration>
```

写好UserMapper.xml和对应的接口UserMapper，在xml文件中指定UserMapper

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.example.mapper.UserMapper">

</mapper>
```

使用方法：session要注意commit close

```java
        InputStream is = null;
        try {
            is = Resources.getResourceAsStream("mybatis/SqlMapConfig.xml");
        } catch (IOException e) {
            e.printStackTrace();
        }
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
        SqlSession session = factory.openSession();
        UserMapper userMapper = session.getMapper(UserMapper.class);
        userMapper.queryAllBook();
        session.commit();
        session.close();
```

### 1.2 整合spring

mybatis配置文件，都交给spring管理，所以不需要配置什么；

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

</configuration>
```

spring配置文件：applicationContext.xml

配置数据库源，连接方式；配置事务管理，创建factory的bean，将事务交给spring管理；

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example.dao"/>

    <context:property-placeholder location="classpath:db.properties" ignore-unresolvable="true"/>

    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="configLocation" value="mybatis/mybatis-conf.xml"/>
        <property name="dataSource" ref="dataSource"/>
        <property name="mapperLocations" value="sqlmap/UserMapper.xml"/>
    </bean>

    <bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory" />
    </bean>

    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="org.example.mapper"/>
        <property name="sqlSessionTemplateBeanName" value="sqlSessionTemplate"/>
    </bean>

</beans>
```

数据库的连接参数可以写在外部的properties文件里

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://47.107.124.164:3306/db_smq?useSSL=false&useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT%2B8
jdbc.username=root
jdbc.password=666
```

使用方法

```java
@Service
public class booksDAOImpl implements booksDAO {

    @Autowired
    private UserMapper userMapper;

    @Override
    public List<Books> getBooks() {
        return userMapper.queryAllBook();
    }

}
```

### 1.3 spring-boot整合mybatis

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

## 2. 标签

| 标签                                           | 作用                                                         |
| :--------------------------------------------- | ------------------------------------------------------------ |
| \<trim prefix="" suffix="" suffixOverrides=""> | 在有条件判断的sql语句中可能会因为条件判断的原因导致sql语句出现where后没有东西或者只有一个and，这时候可以利用\<trim>标签去除这些多余的语句 |
| \<if test="判断条件">                          | 相当于if语句                                                 |

