> java中同步机制的关键字是`synchronized`，`volatile修饰类型的变量`，juc下的`原子变量`【如AtomicInteger，封装了一些原子操作方法，保证所有操作都是原子的】、Lock等



# synchronized

> 一句话描述: synchronized是java关键字，jvm层面的，并发的基本实现手段。
>
> jvm通过使用minitorenter和monitorexit来加锁和解锁，同时保证了同时只有一个线程可以执行指定代码，从而保证了线程安全，同时具有可重入和不可中断的性质。
>
> 每个对象自带一个看不见的锁 minitor  在对象头的markword里
>
> _waitSet和_EntryList 等待池和锁池 ObjectMonitor _owner指向持有锁的对象



> 使用同步的思想达到线程安全:
>
> 同步方法支持一种简单的策略防止线程干扰和内存一致性错误，如果一个对象对多个对象可见，则对该对象变量的所有读取和写入都是通过同步方法完成的,能够保证在同一时刻最多只有一个线程执行该段代码，达到并发安全效果



> synchronized是一种独占的加锁方式。典型的悲观锁，为什么说悲观，是因为它在处理数据的过程中，保持的是悲观的态度，总怕会被别人做操作，所以都会加上锁，线程进入的时候就会获取到锁，这样别的线程进入就可能会有阻塞，必须等进入的线程释放锁才能进入，否则就会处于挂起状态。
>
> 对synchronized的规定
>
> 线程解锁时，必须把共享变量最新值刷新到主内存中。
>
> 线程加锁时,将清空工作内存中共享变量的值，从而使用共享变量时需要从主内存中重新读取最新的值【加锁和解锁是同一把锁】



### synchronized的用法

在java中，synchronized可以修饰在`代码块`和`方法`上，即synchronized块和synchronized方法。

```
synchronized锁住的是括号里的对象，而不是代码。对于非static的synchronized方法，锁的就是对象本身也就是this。
```

[synchronized锁住的是代码还是对象](https://www.cnblogs.com/QQParadise/articles/5059824.html)

```java
4种用法
①修饰方法
public synchronized void method()
{
   // todo
}
②修饰代码块
public void method()
{
   synchronized(this) {
      // todo
   }
}
③修饰静态方法
public synchronized static void method() {
   // todo
}
④类锁
public synchronized static void method() {
   // todo
}

对于静态方法和类锁，取得的锁都是对类对象，即该类的索引对象只有一个能持有锁
类锁和对象锁总结
1.一个线程访问对象同步代码块，另外线程可以访问该对象的非同步代码块
2.锁对象的话，一个线程访问该对象同步代码块，另一线程访问该对象同步代码块时会被阻塞[若锁住对象，访问方法同]
3.若锁住同一对象，会被阻塞。
4.同一个类的不同对象的对象锁互不干扰
5.类锁也是特殊的对象锁，表现和其它一样，但同一个类只有一把对象锁，所以对于同一个类的不同对象使用类锁将会同步
6.类锁和对象锁互不干扰
```



### 核心思想

1.一把锁只能同时被一个线程获取，没拿到的只能等待

2.每个实例都对应自己的一把锁，不同实例之间互不影响，例外:锁对象是class以及synchronized修饰的是static方法时，所有对象共用同一把类锁。

3.无论方法是正常执行完毕还是抛出异常，jvm都会做释放锁的操作。



### 性质

##### 1.可重入

重入:当一个线程试图操作一个由其他线程持有的对象锁的临界资源时，将处于阻塞状态，但当一个线程再次请求自己持有对象锁的临界资源时，这种称为重入

可重入指的是同一线程的外层函数获得锁之后，内层函数可以直接再次获取到该锁

可重入锁：自己可以再次获取自己的内部的锁。比如有线程A获得了某对象的锁，此时这个时候锁还没有释放，当其再次想获取这个对象的锁的时候还是可以获取的，如果不可锁重入的话，就会造成死锁。【重入就是，你拿了锁，再调用该锁包含的代码可以不用再次等待拿锁】

```
java中获取锁的操作的粒度是“线程”，而不是“调用”，即不是每一次调用都是建立一个锁。
```



##### 2.不可中断

如果这个锁被别人获得了，如果还想获得，那么只能等待或者阻塞，直到别的线程释放这个锁。



## 底层原理

##### 可重入原理，加锁次数计算器。

jvm负责跟踪对象被加锁的次数

当计数为0的时候，锁被完全释放

##### 可见性原理

线程内存和主内存，释放前时候，会把修改的东西写入到主内存中



### 加锁和释放锁的原理

#### 反编译看monitor指令

minitorenter monitorexit

可能有多个minitorenter和monitorexit



### synchronized缺陷

1.效率低，锁释放情况少【异常、执行完毕】，尝试获取锁不能设定超时、不能中断一个已经在试图获取锁的线程，不撞南墙不回头

2.不够灵活:加锁和释放锁很单一，如果是读写操作，读的话不需要同步锁

3.无法知道是否成功获取到锁



### 使用synchronized需注意的地方

#### 1.缩小锁的粒度

使用synchronized关键字时，能缩小代码范围就尽量缩小，就是不需要用到同步的地方就尽量不用，能在代码块加同步的地方就尽量不要在整个方法上加，以减小锁的粒度。

```
同步锁一般需要通过减小同步的粒度以达到更好的性能。[在保证线程安全性的前提下]
```



#### 2.不能滥用synchronized

使用synchronized可以避免竞态条件问题，那么为什么不在每个方法声明时都用synchronized修饰。

事实上这将导致synchronized的滥用，导致程序中有过多的同步，也会有弊端, 即使编译器会对可预测的不必要同步做优化但还是尽量不做过多同步。

```
1.首先并非所有的数据都需要锁保护，只有被多个线程访问的可变数据才需要。
2.性能问题，通常情况下不加性能上是会更快的，毕竟jvm少做处理。
3.每个方法都是同步或者说是原子操作，并不能保证复合操作是原子的。
```

```java
//解释下3 这里举一个书上的例子 这是一个典典型的put-if-absent操作【如果不存在则添加】，
if(!vector.contains(element))  //contains是原子操作
	vector.add(element) //add是原子操作
//这里两个操作都是原子操作，这样的复合操作如果没有额外的加锁操作，在多线程竞争的时候还是可能会存在竞态条件，因此不是说复合操作里每个方法都是同步就能保证线程安全，有时候还是要在这个复合操作外做加锁保护 
```



# volatile

java提供的一种较弱的同步机制，用来确保将变量的更新操作通知到其它线程，当把变量声明为volatile后，编译器和运行时都会注意的这个变量是共享的，且不会对该变量的操作和其它内存操作重排序，`被volatile修饰的变量不会被缓存到寄存器或者其它处理器不可见的地方，因此读取volatile类型的变量总会返回最新写入的值`。

> volatile的特性【两个主要用处】
>
> 1.可见性:对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量`最后`的写入.可以防止指令重排序
>
> 2.对任意单个volatile变量的单纯的读/写不会受到干扰，但类似于volatile++这种复合操作不具有原子性。
>
> 比如 n++ 被拆分为几个指令 1.getfield拿到原始n 2.iadd进行加1操作 3.执行putfield把累加后的值写回n

做个小测验 

```java
public class VolatileTest2 { 
    private boolean flag = true; //加上后volatile 不管线程1睡多久，线程2都会取到最后写入值 不会进入while(true)死循环 

    public static void main(String[] args) {
        System.out.println("volatile test");

        VolatileTest2 v = new VolatileTest2();
        Thread t1 = new Thread(() -> {
            try {
                sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            v.flag = false;
        });

        Thread t2 = new Thread(() -> {
            while(v.flag) {
            }
        });
        t1.start();
        t2.start();
    }
}

//执行结果volatile test 然后线程2进入死循环无法exit，线程1的sleep调小一点可能还能exit
//如果线程1的sleep时间稍长，就会让出时间片给线程2，这个时候线程2感应不到flag的变化，就进入死循环。[个人猜想:对于线程2来说 它没及时感应到线程1的改变 貌似就一直读缓存值了 一直跳不出循环]
```



#### 3.2.2.1 什么时候用volatile

volatile作为一种轻量的同步实现，使用方便，但也存在局限性。

volatile常用于某个操作完成，发生中断、或者操作的标志，但是如果验证正确性需要对可见性做复杂的判断，则最好用其它的同步实现，例如volatile的语义不能保证递增操作，比如修饰count，但count++并不是原子操作【实际包含读-改-写三个】，在高并发下容易出现竞争，出现不可预料的结果。

`加锁操作能确保可见性和原子性，而volatile只能确保可见性`，如果有复杂操作或者说对可见性有复杂判断，操作的话用加锁，仅仅只是状态变化之类的轻量操作，如果仅仅通过可见性就能保证线程安全,那么建议使用volatile。

```
//例如 <<java并发实战>> 一书的数绵羊例子 该例子只需让其它线程可见asleep的状态变化，即可以保证线程安全
volatile boolean asleep
...
  while (asleep) {
	 doSomething();
  }
```

**总结:**

1.对变量的写操作不依赖于当前值，如状态控制直接赋值

2.该变量没有包含其它变量的复杂式子中

3.在访问变量时不需要加锁

使用场景:状态控制、单例中的dcl





# 原子变量







## Lock

jdk5的java.util.concurrent.locks包下提供了Lock类来实现同步访问。

synchronized是用于实现同步的关键字，但是存在几个不足的地方。

```
1.代码块synchronized修饰，当一个线程获取到相应的锁执行代码，其它线程只能一直等待该线程释放锁，这个时候如果线程阻塞了没有释放锁，其它线程只能等待，将严重影响程序执行效率。

假如多个线程执行读写文件，读文件，读文件和写文件不允许并发，但读文件允许并发执行的，这个时候如果用synchronized性能上会比较差。
```



### Lock和synchronized的区别

> 1 synchronized是内置的，Lock是一个接口。
>
> 2 synchronized不需要手动释放锁，当synchronized方法或代码块执行完后，系统就会自动让线程释放对锁的占用，而Lock必须要让用户手动释放。
>
> 3.lock支持设置等待时间，避免一直等待，而用synchronized修饰，当一个线程占有锁，其它线程必须等待。



Lock的使用

Lock一般使用try catch finally执行释放锁以保证锁一定被释放。



tryLock方法用于尝试获取锁，有返回值【布尔值】。获取到了立即返回true，如果别的线程占用立即返回false

【tryLock(long timeout,TimeUnit unit)可以等待参数给定时间。在等待的过程中如果获取到了则返回true】



lockInterruptibly方法比较特殊，如果线程等待获取锁，可以中断线程等待状态，假设两个线程同时用lockInterruptibly()想获取某个锁，假设a获取到锁，b只能等待。【threadB.interrupt()可以中断b的等待过程】

```
注意: 当一个线程获取到锁，是不会被interrupt()方法中断，因为interrupt()方法不能中断运行中的线程，只能中断阻塞中的线程。
```



> 如果用synchronized修饰，当一个线程处于等待某个锁的状态，是无法被中断的，只能一直等待下去。

### reentrantlock【可重入锁】

jdk1.5后开始引入。

```
重入的概念
```

ReentrantLock实现了Lock，与synchronized有相同的并发性和内存语义，但是添加了类似轮询，定时锁、可中断锁等候的一些特性、所有加锁释放锁的操作都是显式的。

```
提供了激烈争用时更加的性能:
简单来说，当多线程都想访问共享资源时，jvm可以花更少的时间调度线程，更多的时间用于执行线程
```

### 请不要放弃synchronized

[synchronized](https://www.infoq.cn/article/java-se-16-synchronized)

虽然ReentrantLock是个强大的实现，相对synchronized有很多重要的优势，但ReentrantLock并不是替代内置锁，而是当内置锁不适用时提供的高级实现，synchronized作用java中同步的关键字，仍然有些自己的优势。

```
synchronized的优点
1.synchronized不用手动释放锁，相对安全，而Lock的话有时候出现死锁很难定位到问题，所以不深入了解Lock的话需谨慎使用。Lock必须要手动释放，虽然在finally块释放是件很容易的操作，但也可能忘记，忘记的话就很危险了[就像jdbc不释放连接，都是很危险的事，俺曾经就干过]
2.当jvm用synchronized管理锁和释放时，jvm生成线程转储时能包含锁定信息，这些对调试信息很有用
```



### 什么时候使用ReentrantLock

在早期，ReentrantLock的性能是比内置锁好很多的，但是后面jvm有对Synchronized进行优化，引入了偏向锁、轻量级锁后，性能其实和ReentrantLock差不多了，甚至有些场景用Synchronized更好【参考上面说的synchronized的优点】

```
什么时候用ReentrantLock，就是要用到它的特性的时候。ReentrantLock的三大特性如下。
1.
```







### 总结

在同一个线程中，只要它持有锁，如果接着想用这把锁访问其它方法，只要它需要的锁是手中的这把锁，就不用显示释放锁直接去做事情。

**synchronized 修饰方法时锁定的是调用该方法的对象。它并不能使调用该方法的多个对象在执行顺序上互斥。**

不同场景使用不同的同步方式

```
synchronized:不可中断锁，适用于竞争不激烈，可读性好，且不用手动释放
Lock:可中断锁，多样化同步、竞争激烈时能维持常态
Atomic:缺点只能同步一个值，基于CAS性能也好。
volatile: 可用作并发下的状态标识，防止指令重排序等(如单例的双重校验锁)
```

