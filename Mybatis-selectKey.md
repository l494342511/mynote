# Mybatis-\<selectKey>

```xml
<insert id="insertItem" parameterType="com.ljq.mymbg.model.item">
    <selectKey resultType="Integer" keyProperty="id" order="AFTER">
        select LAST_INSERT_ID()
    </selectKey>
    insert into item(name,userID)
    values (#{name},#{userID})
</insert>
```

这里通过selectKey实现的功能是，将插入新数据获得的自增id放入传入的实体类中；

**例子：**

```java
item item = new item("phone",1);
System.out.println(myService.insertItem(item));
System.out.println("id: "+item.getId());
```

```java
id: 5
```

