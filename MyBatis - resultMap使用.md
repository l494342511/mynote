# MyBatis - \<resultMap>使用

## 1. 为什么要使用resultMap

```xml
<select id="getUsersAndItems" resultType="com.ljq.entity.user">
        select * from `user`
</select>
```

当我们查询的只是一个表时，或者装配的实体类中，都是基本数据类型，并且每个属性都对应着数据表中的字段且字段名相同，这时返回类型使用实体类的话，也可以实现自动装配

但是当字段名不同，或者实体类中有类型为其他实体类的属性时，又或者涉及多表查询，有集合类型的属性时，直接使用实体类作为返回对象就很难实现自动装配。

**因此我们需要利用resultMap指定返回的字段需要装配到实体类的哪个属性中**

## 2. resultMap使用

首先实体类中有几种类型的属性：

### 2.1 基本数据类型属性

```xml
<id column="id" jdbcType="INTEGER" property="id"/>
<result column="name" jdbcType="VARCHAR" property="name"/>
```

这两句作用是相同的，不同的是\<id>指定的是实体类的主键属性，\<result>指定一般属性

column：指定sql返回的字段

jdbcType：指定sql返回的字段类型

property：指定字段装配到实体类的哪个属性

### 2.2 实体类属性

```xml
<association property="item" javaType="com.ljq.mymbg.model.item">
    <id column="itemID" jdbcType="INTEGER" property="id"/>
    <result column="itemName" jdbcType="VARCHAR" property="name"/>
    <result column="userID" jdbcType="INTEGER" property="userID"/>
</association>
```

通过association标签来关联其他实体类

### 2.3 集合属性

```xml
<collection property="items" ofType="com.ljq.mymbg.model.item">
    <id column="itemID" jdbcType="INTEGER" property="id"/>
    <result column="itemName" jdbcType="VARCHAR" property="name"/>
    <result column="userID" jdbcType="INTEGER" property="userID"/>
</collection>
```

## 2.4 整个resultMap

```xml
<resultMap id="resultMap" type="com.ljq.mymbg.model.user">
        <id column="id" jdbcType="INTEGER" property="id"/>
        <result column="name" jdbcType="VARCHAR" property="name"/>
        <result column="pwd" jdbcType="VARCHAR" property="pwd"/>
        <association property="item" javaType="com.ljq.mymbg.model.item">
            <id column="itemID" jdbcType="INTEGER" property="id"/>
            <result column="itemName" jdbcType="VARCHAR" property="name"/>
            <result column="userID" jdbcType="INTEGER" property="userID"/>
        </association>
        <collection property="items" ofType="com.ljq.mymbg.model.item">
            <id column="itemID" jdbcType="INTEGER" property="id"/>
            <result column="itemName" jdbcType="VARCHAR" property="name"/>
            <result column="userID" jdbcType="INTEGER" property="userID"/>
        </collection>
    </resultMap>
```

