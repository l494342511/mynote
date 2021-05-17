# AOP整合Spring Boot

## 1. 使用方法

### 1.1 配置Aspect

```java
@Aspect
@Component
public class FastTextAop {

    public static Integer TrainDataNum = 0;

    @Pointcut("execution(* com.D9Lab.nuclear.service.impl.FastTextServiceImpl.getTrainData(..))")
    private void Count(){}

    @Before("Count()")
    private void start(){
        System.out.println("aop生效了");
        TrainDataNum++;
    }
    
}
```

应该要注意的地方：

```java
@Pointcut("execution(* com.service.FastTextService.getTrainData(..))")
```

在指向切入点的地方，除了方法名和包名需要正确以外，还必须保证方法的参数类型数量也和切入点一致，这里是使用了 .. 表达式来表示所有参数，这里表示含义等于

```java
@Pointcut("execution(* com.serviceFastTextService.getTrainData(String,int,Integer))")
```

## 1.2 配置启动类

```java
@SpringBootApplication
@MapperScan(basePackages = "com.D9Lab.nuclear.mapper")
@ComponentScan(basePackages = {"com.D9Lab.nuclear.*"})
@EnableAspectJAutoProxy(proxyTargetClass=true)
public class AnalysApplication {
    public static void main(String[] args) {
        SpringApplication.run(AnalysApplication.class, args);
    }
}
```

**重点是：**

```java
@ComponentScan(basePackages = {"com.D9Lab.nuclear.*"})
@EnableAspectJAutoProxy(proxyTargetClass=true)
```

由于aop是自建包，@SpringBootApplication不会扫描到，因此要指定扫描的路径，然后再开启aop