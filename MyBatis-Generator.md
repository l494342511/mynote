# MyBatis-Generator

## 1. 使用

### 1.1 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <properties resource="generator.properties"/>
    <context id="MySqlContext" targetRuntime="MyBatis3" defaultModelType="flat">
        <!--配置分隔符 如在表名前后添加"`"-->
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>
        <property name="javaFileEncoding" value="UTF-8"/>
        <!-- 为模型生成序列化方法-->
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin"/>
        <!-- 为生成的Java模型创建一个toString方法 -->
        <plugin type="org.mybatis.generator.plugins.ToStringPlugin"/>
        <!--生成mapper.xml时覆盖原文件-->
        <plugin type="org.mybatis.generator.plugins.UnmergeableXmlMappersPlugin" />
        
        <!--配置注释-->
        <commentGenerator type="edu.whut.mall.MyCommentGenerator">
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true"/>
            <property name="suppressDate" value="true"/>
            <property name="addRemarkComments" value="true"/>
            <property name="author" value="wayne"/>
            <property name="dateFormat" value="yyyy/MM/dd"/>
        </commentGenerator>

        <!--配置相连数据库-->
        <jdbcConnection driverClass="${jdbc.driverClass}"
                        connectionURL="${jdbc.connectionURL}"
                        userId="${jdbc.userId}"
                        password="${jdbc.password}">
            <!--解决mysql驱动升级到8.0后不生成指定数据库代码的问题-->
            <property name="nullCatalogMeansCurrent" value="true" />
        </jdbcConnection>

        <!--指定model、mapper、xml生成路径-->
        <javaModelGenerator targetPackage="edu.whut.mall.model" targetProject="pojo\src\main\java"/>

        <sqlMapGenerator targetPackage="mapper" targetProject="pojo\src\main\resources"/>

        <javaClientGenerator type="XMLMAPPER" targetPackage="edu.whut.mall.mapper"
                             targetProject="pojo\src\main\java"/>
        <!--指定需要自动生成的表-->
        <!--若生成全部表tableName设为%-->
        <table tableName="refund_request" domainObjectName="RefundRequest">
            <generatedKey column="id" sqlStatement="MySql" identity="true"/>
        </table>
        
    </context>
</generatorConfiguration>
```

### 1.2 配置生成器

```java
private static void generatorPOJO() throws IOException, XMLParserException, InvalidConfigurationException, SQLException, InterruptedException {
        //MBG 执行过程中的警告信息
        List<String> warnings = new ArrayList<String>();
        //当生成的代码重复时，覆盖原代码
        boolean overwrite = true;
        //读取我们的 MBG 配置文件
        InputStream is = Generator.class.getResourceAsStream("/generatorConfig.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(is);
        is.close();

        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        //创建 MBG
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        //执行生成代码
        myBatisGenerator.generate(null);
        //输出警告信息
        for (String warning : warnings) {
            System.out.println(warning);
        }
    }
```

### 1.3 配置注释生成器

```java
public class myCommentGenerator extends DefaultCommentGenerator {

    private boolean addRemarkComments = true;
    private Properties properties;

    @Override
    public void addConfigurationProperties(Properties properties) {
        super.addConfigurationProperties(properties);
        this.properties = properties;
    }

    //model类上的注解
    @Override
    public void addModelClassComment(TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {
        String author = properties.getProperty("author");
        String dateFormat = properties.getProperty("dateFormat", "yyyy-MM-dd");
        SimpleDateFormat dateFormatter = new SimpleDateFormat(dateFormat);

        String remarks = introspectedTable.getRemarks();
        topLevelClass.addJavaDocLine("/**");
        topLevelClass.addJavaDocLine(" * "+remarks);
        topLevelClass.addJavaDocLine(" * @author"+author);
        topLevelClass.addJavaDocLine(" * @date"+dateFormatter.format(new Date()));
        topLevelClass.addJavaDocLine("**/");
    }

    //model类属性上的注解，根据数据库中属性的备注注解，remark代表备注
    @Override
    public void addFieldComment(Field field, IntrospectedTable introspectedTable, IntrospectedColumn introspectedColumn) {
        String remarks = introspectedColumn.getRemarks();
        if(addRemarkComments&& StringUtility.stringHasValue(remarks)){
            if(remarks.contains("\"")){
                remarks = remarks.replace("\"","'");
            }
            field.addJavaDocLine("/**");
            field.addJavaDocLine(" * " + remarks);
            field.addJavaDocLine(" */");
        }
    }

    @Override
    public void addGetterComment(Method method, IntrospectedTable introspectedTable, IntrospectedColumn introspectedColumn) {
        method.addJavaDocLine("这是Getter方法的注释");
    }

    @Override
    public void addSetterComment(Method method, IntrospectedTable introspectedTable, IntrospectedColumn introspectedColumn) {
        method.addJavaDocLine("这是setter方法的注释");
    }
}
```

## 2. 自动生成的代码

Mybatis-Generator生成的文件包括（以user表为例）：

| 文件名           | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| user.java        | user表对应的实体类                                           |
| UserMapper.java  | user表对应的mapper接口                                       |
| userMapper.xml   | UserMapper.java对应的sql方法                                 |
| userExample.java | user表对应的动态sql方法，用于更灵活的添加约束条件以及排序方法 |

### 2.1 Example

该类拥有的属性如下：

```java
public class userExample {
    
    protected String orderByClause;//用于排序

    protected boolean distinct;

    protected List<Criteria> oredCriteria;//用于添加约束条件
    
}
```

**这些属性在sql中的用法：**

**orderByClause：**

```xml
<if test="orderByClause != null">
      order by ${orderByClause}
</if>
```

**distinct：**

```xml
<if test="distinct">
      distinct
</if>
```

**oredCriteria：**

```xml
<if test="_parameter != null">
      <include refid="Example_Where_Clause" />
</if>
```

```xml
<sql id="Example_Where_Clause">
    <where>
      <foreach collection="oredCriteria" item="criteria" separator="or">
        <if test="criteria.valid">
          <trim prefix="(" prefixOverrides="and" suffix=")">
            <foreach collection="criteria.criteria" item="criterion">
              <choose>
                <when test="criterion.noValue">
                  and ${criterion.condition}
                </when>
                <when test="criterion.singleValue">
                  and ${criterion.condition} #{criterion.value}
                </when>
                <when test="criterion.betweenValue">
                  and ${criterion.condition} #{criterion.value} and #{criterion.secondValue}
                </when>
                <when test="criterion.listValue">
                  and ${criterion.condition}
                  <foreach close=")" collection="criterion.value" item="listItem" open="(" separator=",">
                    #{listItem}
                  </foreach>
                </when>
              </choose>
            </foreach>
          </trim>
        </if>
      </foreach>
    </where>
  </sql>
```

可以看到，criterion类有三个重要的属性，动态sql中主要使用这三个属性进行拼接：

| 属性        | 描述         |
| ----------- | ------------ |
| condition   | 约束的规则   |
| value       | 约束的值     |
| secondValue | 约束的第二值 |

**使用例子：**

```java
List<user> users;
userExample userExample = new userExample();
userExample.setDistinct(true);
userExample.setOrderByClause("id");
userExample.createCriteria().andIdEqualTo(1);
users = userMapper.selectByExample(userExample);
```

