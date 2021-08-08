# Java 并发编程

## 0. 企业开发场景的并发

1. 创建普通资源类
2. 创建资源类的普通方法，加锁
3. 调用时新建一个线程
4. 使用lambda表达式将资源类放入线程
5. 在线程中调用方法

## 1. 使用Synchronized锁

在普通方法上加Synchronized锁，方法调用时加锁，返回时释放锁

```java
class Ticket{
    int number;

    public Ticket(int number) {
        this.number = number;
    }

    public synchronized void sale(){
        System.out.println(Thread.currentThread().getName()+"剩余票数："+(number--));
    }
}
```

```java
Ticket ticket = new Ticket(100);
        new Thread(()->{
            for(int i=0;i<30;i++){
                ticket.sale();
            }
        },"A").start();
```



## 2. Lock锁

```java
class Ticket{
    int number;

    public Ticket(int number) {
        this.number = number;
    }
    
    Lock lock = new ReentrantLock();

    public void sale(){
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName()+"剩余票数："+(number--));
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
}
```

### 3. 线程协同

#### 3.1 通知与等待

```java
public synchronized void incream() throws InterruptedException {
        while(number!=0){
            this.wait();
        }
        System.out.println(Thread.currentThread().getName()+"："+number++);
        this.notifyAll();
    }
```

进入等待状态的条件应该用while判断，如果用if进行判断会导致等待过程中若条件发生变化，则线程会被虚假唤醒

#### 3.2 Lock的通知等待

```java
public void odd(IntConsumer printNumber) throws InterruptedException {
        lock.lock();
        try{
            while (flag==0||process%2==0){
                condition1.await();
            }
            printNumber.accept(process++);
            flag = 0;
            condition0.signal();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
```

重点：

1. while中的条件应该是想要成功进行该线程的条件的反条件，如上是当process为奇数时进行该线程
2. 使用condition.await()后线程释放锁并进入等待
3. condition.signal()通知特定的condition

   