# Lambda表达式

## 1. Lambda的基本结构

```java
(输入参数)->{statement};
```

Lambda表示就是一段匿名的函数，由输入参数、箭头、函数体组成，下面举一个实例：

**打印函数：传统方法**

```java
interface myLamda{
    void print();
}

class myLamdaImpl implements myLamda {
    @Override
    public void printStr(String s) {
        System.out.println(s);
    }
}

public static void main(String[] args){
    myLamda myLamda = new myLamdaImpl();
    myLamda.printStr("lambda表达式");
}
```

**打印函数：lambda表达式**

```java
@FunctionInterface
interface myLambda{
	void print();
}

public static void main(String[] args){
    myLambda myLambda = (s)->{
    	System.out.println(s);
    }
    myLambda.print("lambda表达式");
}
```

分析两个实现方法，可以发现lambda表达式与普通方法的**区别**：

1. 不需要申明参数的类型，由程序自行推断
2. 不需要申明方法的返回类型，由程序自行推断
3. lambda表达式**可以被赋值给变量**，变量的类型为一个**函数式接口**，因此lambda表达式也**能作为返回结果**；

## 2. lambda的用法

### 2.1 

