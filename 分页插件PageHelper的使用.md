# 分页插件PageHelper的使用

## 1. maven依赖引入

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.3.0</version>
</dependency>
```

## 2. 设置分页

```java
PageHelper.startPage(pageNum,pageSize);//指定页数和每页的数据条数
```

该语句只会影响其后的第一句sql语句，并且是线程安全的

## 3. 执行sql语句

```java
List<user> users = myService.getAllUser();
```

```xml
<select id="getAllUser" resultType="com.ljq.mymbg.model.user">
        select <include refid="base_comln_list"></include> from `user`
</select>
```

执行后发现，sql语句被插件拦截，仅返回了所设定页数下的数据，并且仅第一条sql受影响

![image-20210426105748642](%E5%88%86%E9%A1%B5%E6%8F%92%E4%BB%B6PageHelper%E7%9A%84%E4%BD%BF%E7%94%A8.assets/image-20210426105748642.png)

## 4. 封装分页信息

```java
PageInfo<user> pageInfo = new PageInfo<user>(users);
```

该插件封装了一个分页类来管理分页信息，该类的属性有

```java
public static final int DEFAULT_NAVIGATE_PAGES = 8;
//当前页
private int pageNum;
//每页的数量
private int pageSize;
//当前页的数量
private int size;

//由于startRow和endRow不常用，这里说个具体的用法
//可以在页面中"显示startRow到endRow 共size条数据"

//当前页面第一个元素在数据库中的行号
private long startRow;
//当前页面最后一个元素在数据库中的行号
private long endRow;
//总页数
private int pages;

//前一页
private int prePage;
//下一页
private int nextPage;

//是否为第一页
private boolean isFirstPage = false;
//是否为最后一页
private boolean isLastPage = false;
//是否有前一页
private boolean hasPreviousPage = false;
//是否有下一页
private boolean hasNextPage = false;
//导航页码数
private int navigatePages;
//所有导航页号
private int[] navigatepageNums;
//导航条上的第一页
private int navigateFirstPage;
//导航条上的最后一页
private int navigateLastPage;
```

