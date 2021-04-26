# 动态sql

## 1. if

```xml
<if test="表达式">sql语句</if>
```

当test的表达式不成立时，\<if>标签包裹的sql语句不生效，但同时可能会造成语法错误，如

```xml
<if test="name != null">name = #{name},</if>
<if test="pwd != null">pwd = #{pwd}</if>
```

若第一行成立第二行不成立，则会多出一个，

## 2. where

```xml
<where>
    sql语句
</where>
```

当被where标签包裹的sql语句生效时，where会为其添加一个where字段，并且当sql语句中有多余的and或or字段时，会被其自动剔除

## 3. set

```xml
<set>
	sql语句
</set>
```

与where语句一样，只有当被\<set>标签包裹的sql语句生效时，\<set>才会为其自动添加一个set字段

## 4. choose(when,otherwise) 

```xml
<choose>
    <when test="表达式1">
        sql语句
    </when>
    <when test="表达式2">
        sql语句
    </when>
    <otherwise>
        sql语句
    </otherwise>
</choose>
```

choose，when，otherwise三个标签的作用类似swith case default 

## 5. trim

```xml
<trim prefix="set" suffixOverrides=",">
    sql语句
</trim> 
<trim prefix="where" prefixOverrides="and|or">
    sql语句
</trim> 
```

trim标签可以替代set和where，并且更加灵活，当sql语句生效时，会自动添加上set或where，并且去除多余的字符。prefix是前缀，suffix是后缀

## 6. sql引用

```xml
<sql id="">
    语句
</sql>
```

sql标签可以定义一段sql片段，可以直接通过其id与\<include refid="">进行调用

```xml
<sql id="Base_Column_List">
    id, name, pwd
</sql>

```

把数据库字段写成一个sql片段是常用的方法

## 7. foreach

```xml
<foreach collection="ids" item="id" open="" close="" separator="or">
                id = #{id}
</foreach>
```

foreach标签可以对一个集合做循环操作，collection是循环的集合，item是取出的数据，open是循环体语句的开始，close是循环体的结束，separator是循环体内的间隔符