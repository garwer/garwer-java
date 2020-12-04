# 线程封闭

当访问共享的可变数据时，常需要用同步实现，还有一种避免使用同步的方式就是不共享数据，仅仅在单线程内访问数据，就不需要同步。这种技术叫线程封闭【Thread Confinement】，是实现线程安全的最简单方式之一，即使对象不是线程安全的，但是被封闭在一个线程中，也能够实现线程安全。

```
java没有强制要求变量要由锁保护，也没强制要求将对象封闭到线程中，这些需要我们平时自己在程序中实现，java提供了一些机制来维持线程封闭性，但即使如此，也应当控制封闭在线程内的对象不从线程中逸出。
```

线程封闭的一种常用应用是JDBC的Connection对象，jdbc规范不要求Connection是线程安全的



线程封闭的实现方式

### ThreadLocal

```
ThreadLocal: java提供线程封闭的核心类。ThreadLocal是线程的本地实现版本，不是一个Thread，而是线程局部变量。
因为不同线程之间是无法看到彼此线程中的变量。ThreadLocal采用类似线程间数据隔离的效果，为变量在每个线程中创建了一个副本，即使这个变量是线程不安全的，但是在彼此线程中，每个线程可以访问自己内部的副本变量不会互相干扰。
```



####  ThreadLocal和对比同步机制的好处

> 同步机制中，通过对象的锁机制保证同一时间只有一个线程访问变量,这个时候对象是多个线程共享，需要用同步的方式，并考虑什么时候锁住、释放对象，设计和编写比较麻烦。
>
> ThreadLocal使用特别的方式处理多线程并发访问，为每个线程提供一个变量副本，从而隔离不同线程对数据访问的冲突，就没必要对变量同步了，可以考虑把不安全的变量封装到ThreadLocal。
>
> `前者只有一个对象，以时间换空间，后者则为每个线程创建一个副本。以空间换时间`



#### ThreadLocal使用场景

当有些对象thread unsafe，但不想同步访问该对象，而是使用创建副本的方式给每个线程自己的对象实例，用线程封闭的技术实现线程安全。

> 常用于高并发情况下数据库连接、session管理等

```java
private static ThreadLocal<Connection> connectionHolder  = new ThreadLocal<Connection>() {
    public Connection initialValue() {
        return DriverManager.getConnection(DB_URL);
    }
};

public static Connection getConnection() {
    return connectionHolder.get();
}
```



#### Thread使用注意事项

`ThreadLocal用完需和同步机制一样做清理，不然可能导致内存泄漏`。



```
/**可能出现的情况
1.用static修饰导致ThreadLocal生命周期变长
2.使用ThreadLocal却没及时回收，回收通过ThreadLocalMap的(get/set/remove)方法
其中remove通过判断是否有null键，没有则删除
set是通过replaceStaleEntry，当出现null，可通过在当前位置重新创建新的Entry以清理**/
//例如
public void remove() {
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
   	 	m.remove(this);
}
```



### ThreadLocal内存泄漏问题

　　每个thread中都存在一个map, map的类型是ThreadLocal.ThreadLocalMap. Map中的key为一个threadlocal实例. 这个Map的确使用了弱引用,不过弱引用只是针对key，当把threadLocal实例设置为null，没有任务强引用执行threadLocal对象，该对象将被gc回收，但是value不会被回收，留着也没用，产生了内存泄漏问题。

使用弱引用相对保险 如果使用强引用，那么被回收了没有手动删除key，所以只要当前线程不消亡，ThreadLocalMap引用的那些对象就不会被回收，可以认为这导致Entry内存泄漏

那就是在每次`get()`/`set()`/`remove()`ThreadLocalMap中的值的时候，会自动清理key为null的value

ThreadLocalMap和Thread生命周期一样长如果不合理使用做好清理工作都可能导致内存泄露，使用弱引用会多一层保障【但最好也要做清理】

```
说到底是因为ThreadLocalMap的生命周期跟Thread一样长，如果没有手动删除对应key就会导致内存泄漏，而不是因为弱引用。【至少用弱引用更保险点】
```

![7A077E15D8A05EA40BEF0F037DEF3A64](images/7A077E15D8A05EA40BEF0F037DEF3A64.png)



#### 栈封闭

方法区是线程私有的，其中局部变量存在虚拟机栈中，也是线程私有的，当多个线程访问同一个方法的局部变量时，只要这个局部变量没有逸出，那么就不会出现线程安全问题。

好比平时写代码，在一个方法中创建对象如果对象没发生逃逸，则不用做多余的同步。

```
public void method () {
  int a = 1; //只要变量a没溢出，就不可能有线程安全问题
}
```

