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

