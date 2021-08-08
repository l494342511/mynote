# Spring三种注入方式

## 1. 最常见且最不推荐的Autowired注入

```java
public class MyBean {
   @Autowired
   private AnotherBean anotherBean;
   //Business logic...
}
```

#### 优点

- 最简洁
- 很多java开发者都喜欢这种方式

#### 缺点

- 便利会弱化代码结构设计
- 很难测试
- 依赖不能是可变的(无法final)
- 容易出现循环依赖
- 需要使用到多个spring或者java注解

## 2. Setter注入

```java
public class MyBeanTest {
    private BookMapper bookMapper;

    @Autowired
    public void setBookMapper(BookMapper bookMapper) {
        this.bookMapper = bookMapper;
    }
}
```

#### 优点

- 对循环依赖免疫
- 随着setter的添加，高度耦合的类很容易被识别出来。

#### 缺点

- 违反开放封闭原则
- 会把循环依赖隐藏掉
- 三种方法里最模板化的方式
- 依赖不能是可变的(无法final)

## 3. 最推荐的构造器注入

```java
public class MyBean{
	private final BookMapper bookMapper;
    //@Autowired 在只有一个构造函数情况下可以不写注解
    public MyBean(BookMapper bookMapper) {
        this.bookMapper = bookMapper;
    }
}
```

#### 优点

- 依赖可以是final的
- [spring官方推荐的方式](https://spring.io/blog/2007/07/11/setter-injection-versus-constructor-injection-and-the-use-of-required)
- 三种方式里最容易测试的方式
- 高耦合类随着构造参数的增长很容易被识别出来
- 其他开发平台的开发者也很熟悉
- 不需要依赖@Autowired注解

#### 缺点

- 构造函数需要下沉到子类
- 容易产生循环依赖