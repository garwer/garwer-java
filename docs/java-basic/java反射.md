>最近有空的时候会看下jdk和spring的源码，发现反射的使用是非常频繁的。之前也对反射或多或少有过了解，但也只是停留在了解的阶段，总结一下来加深自己的印象。

```
反射的基本概念:程序可以访问、检测和修改其本身状态或行为的一种能力。
```
>  反射机制是java的特性之一，指的是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为java语言的反射机制。(摘自百度)

> 反射是java语言的一个特性，它允程序在运行时（注意不是编译的时候）来进行自我检查并且对内部的成员进行操作。例如它允许一个java的类获取它所有的成员变量和方法并且显示出来(官方概念)

## 常见应用场景

1.各种框架中,比如spring的IOC(控制反转)

spring ioc的思想是将设计好的对象交给容器控制，帮我们实例化，其中就有用到反射来实现。

##### 大致步骤(伪代码)

```java
①spring配置好bean
<bean id="courseDao" class="com.qcjy.learning.Dao.impl.CourseDaoImpl"></bean>  
②解析bean中class属性
③通过反射获取Class对象
Class<?> cls = Class.forName(classStr);  
④实例化对象
Object obj = cls.newInstance();
⑤放到容器中
```

2.tomcat读取web.xml

tomcat服务器提供了处理请求和应答的方式，针对不同处理动作，对外提供接口让开发者做相应的具体实现。

参考:https://blog.csdn.net/tjiyu/article/details/54590259

#### 学习反射机制前，需要先简单了解一下JVM，java之所以能跨平台，是因为java虚拟机。类的加载和运行都是依托它。



![uml_class_struct](images/uml_class_struct.jpg)


##### 举个栗子:

```
Object object = new Object();
```
##### 运行该程序顺序
>1.JVM启动，先将代码编译成.class文件，根据类加载器(Object类由顶层父类Boostrap ClassLoader)加载这个class文件并加载到jvm内存中
>2.方法区存类的信息，类加载器通过方法区上类的信息在堆上创建类的Class对象(不是new出来的对象，而是类的类型对象，每个类只有一个Class对象，该Class对象由jvm保证唯一，之后类的创建根据这个Class对象操作)。
>3.jvm创建对象前，会先检查类是否被加载，如果加载好，则为对象分配内存，初始化就是代码:new Object()

这种new出对象的方式在实际应用中很常见，相当于程序相当于写死了给jvm去跑。假如一个服务器上突然遇到某个请求要用到某个类，但没加载进jvm，这种时候总不能停下来再new一个这个类的对象，之后再重启服务器。。

这个时候回到java反射的概念。简单来说能动态获取一个类的信息并且去操作它(属性，方法等等…)。它允许在程序在运行时加载，探知使用编译期间已知或未知的class，method，属性，参数等等。



## 动态加载和静态加载

> 动态加载:程序在运行期间调用方法，即使方法是错误的程序依旧执行，通过动态加载可以使程序更加灵活方便日后维护
>
> 静态加载:程序在编译时执行，在此过程中只要方法出错，编译器会报错，就会中断程序，这是我们常用的。

而反射恰好在编译时不执行，而是在运行期间生效。



### jdk中反射中常用的类

```class
java.lang.Class
java.lang.reflect.Constructor
java.lang.reflect.Field        
java.lang.reflect.Method
java.lang.reflect.Modifier
```



## 反射可以做的事

```java
package reflect;

//Person类用于后续测试
public class Person {
    private String sex;
    public String age;
    public String work;
    public String name;
    public String getSex() {
        return sex;
    }
    public void setSex(String sex) {
        this.sex = sex;
    }
    public String getAge() {
        return age;
    }
    public void setAge(String age) {
        this.age = age;
    }
    public String getWork() {
        return work;
    }
    public void setWork(String work) {
        this.work = work;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Person() {
        System.out.println("公有，调用无参的构造方法");
    }
    public Person(String name, String sex) {
        System.out.println("公有，调用有参构造方法" + "name=" + name + "sex" + sex);
    }
    private Person(String sex) {
        System.out.println("调用私有，sex=" + sex);
    }
    public void sayHello(String word) {
        System.out.println("公有方法说的话->" + word);
    }
    private void say(String word) {
        System.out.println("私有方法说的话->" + word);
    }
}
```



### 1.获取Class对象(使用Person类做演示)

```java
/**
 * 获取Class对象的三种方式
 */
public class ClassTest {
    public static void main(String[] args) throws ClassNotFoundException {
        /**
         * 1. 通过getClass
         *  Object类和Class类都有getClass()本地方法用于获取
         *  在Java中所有的类都继承于Object类，但不用在声明一个类时显示的extends Object
         *  这边采用的是Object类里的getClass()方法
         */
        Object o = "";
        System.out.println("o->" + o.getClass()); //o->class java.lang.String
        Person p = new Person();
        Class clazz1 = p.getClass();
        //两个Person对象不同 但类是相同的 就好比说两个人有各个方面不同 但都是人这个类
        Person p1 = new Person();
        System.out.println("p->" + clazz1);//class reflect.Persons
        System.out.println(p == p1); //false
        System.out.println(p.getClass() == Person.class); //true
        System.out.println(p.getClass() == p1.getClass()); //true
        /**
         * 2.类名.class
         */
        Class clazz2 = Person.class;//class reflect.Person

        /**
         * 3.Class.forName() jdbc也是通过这种方式加载驱动类
         */
        Class clazz3 = Class.forName("reflect.Person");

        /**
         * 总结：
         *  三种获取Class对象方法 1.Object类的getClass() 2.类名.class 3.Class.forName()
         *  判断实例对象类型 getClass与instance的区别在于getClass不考虑继承 intance的话如果是父类也属于
         *  在运行期间，一个类只有一个Class对象产生 Class是单例的 验证如下
         *  相比之下 1已经有对象了还要反射显得没啥意义 2要导入类 依赖强 3第三种实用性更强
         */
        //true意味着三个类对象是同一个类(Class)的实例
        System.out.println("class对象是否相等->" + (clazz1 == clazz2 && clazz1 == clazz3));//class对象是否相等->true
        System.out.println(p.getClass() == o.getClass());//false
        System.out.println(p instanceof Person && p instanceof Object); //true
    }
}
```

### 2.创建对象

这边提一下java创建对象的5种方式

```
1.使用new
2.使用java.lang.Class类的newInstance()
3.使用Constructor类的newInstance方法
4.使用clone方法 写法类似
Person person = Person.class.newInstance();
Person personClone = (Person)person.clone();
这边被克隆的类要实现Cloneable并重写clone方法 虽然Cloneable接口为空但也要实现作为标志，否则object的clone方法将报CloneNotSupportedException的错
5.使用反序列化 序列化后对象反序列化 
6.动态代理Proxy方式 Proxy的静态方法newProxyInstance
```

这边2，3，6是使用反射的机制来创建对象。

#### 样例代码

```java
public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
    /**
     * 利用反射机制中的newInstance方法创建对象
     */
    Class clazz = Class.forName("reflect.Person"); //先获取要创建对象的类的Class对象
    Object o = clazz.newInstance();
    System.out.println(o.getClass());//class reflect.Person
    /**
     * 通过Constructor类创建
     * 通过getConstructors方法获得构造器Constructor对象并创建
     */
    Object o1 = clazz.getConstructor().newInstance();
    System.out.println(o1.getClass());//class reflect.Person

    //可通过创建string对象
    Object o2 = String.class.getConstructor(String.class).newInstance("hello world");//class reflect.Person
    System.out.println(o2);//hello world
}
```



### 3.获取成员变量并调用

```java
public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException, NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        Class personClass = Class.forName("reflect.Person");
        //获取共有字段 如果是private修饰会报NoSuchFieldException
        Field f = personClass.getField("name");
        System.out.println(f); //public java.lang.String reflect.Person.name

        Field[] af = personClass.getFields();
        System.out.println(af);//对象数组 可通过遍历获取
        System.out.println(Arrays.stream(af).count()); //公有数目为3

        //公有私有都能获取
        Field f1 = personClass.getDeclaredField("sex");//private java.lang.String reflect.Person.sex
        System.out.println(f1);
        //获取公有字段并调用
        Object object = personClass.getConstructor().newInstance();
        f.set(object, "garwer");
        Person person = (Person)object;
        System.out.println((person.name));//garwer

        //获取私有字段并调用 强制变性 - -
        Person nan = (Person) personClass.newInstance();
        nan.setSex("女");
        f1.setAccessible(true);//暴力反射 解除私有限定 如果不加上这句会报错
        f1.set(nan,"男");
        System.out.println(nan.getSex()); //输出男
    }
```

### 4.获取方法(包含构造方法和成员方法)并调用

```java
package reflect;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class TestMethod {
    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        //获取Class对象
        Class personClass = Class.forName("reflect.Person");
        /**
         * （一）构造方法 
           名字定义与类名相同 无返回类型业务void 在创建一个对象使用new操作执行的 主要用于初始化
         * 不能被static、final、synchronized、abstract和native等修饰 构造方法不能被子类继承
          */
        System.out.println("=获取所有公有构造方法=");
        Constructor[] conArray = personClass.getConstructors();
        for(Constructor c : conArray){
            System.out.println(c);
        }

        System.out.println("=获取所有构造方法(含私有)=");
        conArray = personClass.getDeclaredConstructors();
        for(Constructor c : conArray){
            System.out.println(c);
        }

        /**
         * 打印结果
         *   =获取所有公有构造方法=
             public reflect.Person(java.lang.String,java.lang.String)
             public reflect.Person()
             =获取所有构造方法(含私有)=
             public reflect.Person(java.lang.String,java.lang.String)
             private reflect.Person(java.lang.String)
             public reflect.Person()
         */


        /**
         * （二）成员方法
         */
        //获取所有公有方法
        System.out.println("=获取所有公有方法=");
        personClass.getMethods();
        Method[] methodArray = personClass.getMethods();
        for(Method m : methodArray){
            System.out.println(m);
        }
        System.out.println("=获取所有方法 包括私有=");
        methodArray = personClass.getDeclaredMethods();
        for(Method m : methodArray){
            System.out.println(m);
        }
        System.out.println("=获取公有的Person()方法=");
        //getMethod(方法名,参数类型)
        Method m = personClass.getMethod("sayHello", String.class);
        System.out.println(m);
        //先实例化person类
        Object obj = personClass.getConstructor().newInstance();
        //invoke(要调用的对象，实参)
        m.invoke(obj, "hello world");
        System.out.println("=获取私有的say方法="); //getDeclaredMethod(方法名，参数类型)可以用于获取私有方法(也能获取公有) 得解除私有限定
        m = personClass.getDeclaredMethod("say", String.class);
        System.out.println(m);
        m.setAccessible(true);//解除私有限定
        Object result = m.invoke(obj,"say hi"); //调用方法并获取返回值
        System.out.println("返回值：" + result); //null 因为是void方法无返回 如果有返回类型将会有返回值

        /**
         * 打印结果
         * =获取所有公有方法=
         public java.lang.String reflect.Person.getName()
         public void reflect.Person.setName(java.lang.String)
         public java.lang.String reflect.Person.getWork()
         public java.lang.String reflect.Person.getSex()
         public void reflect.Person.setAge(java.lang.String)
         public java.lang.String reflect.Person.getAge()
         public void reflect.Person.setWork(java.lang.String)
         public void reflect.Person.setSex(java.lang.String)
         public void reflect.Person.sayHello(java.lang.String)
         public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
         public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
         public final void java.lang.Object.wait() throws java.lang.InterruptedException
         public boolean java.lang.Object.equals(java.lang.Object)
         public java.lang.String java.lang.Object.toString()
         public native int java.lang.Object.hashCode()
         public final native java.lang.Class java.lang.Object.getClass()
         public final native void java.lang.Object.notify()
         public final native void java.lang.Object.notifyAll()
         =获取所有方法 包括私有=
         public java.lang.String reflect.Person.getName()
         public void reflect.Person.setName(java.lang.String)
         public java.lang.String reflect.Person.getWork()
         public java.lang.String reflect.Person.getSex()
         public void reflect.Person.setAge(java.lang.String)
         public java.lang.String reflect.Person.getAge()
         public void reflect.Person.setWork(java.lang.String)
         public void reflect.Person.setSex(java.lang.String)
         private void reflect.Person.say(java.lang.String)
         public void reflect.Person.sayHello(java.lang.String)
         =获取公有的Person()方法=
         public void reflect.Person.sayHello(java.lang.String)
         公有，调用无参的构造方法
         公有方法说的话->hello world
         =获取私有的say方法=
         private void reflect.Person.say(java.lang.String)
         私有方法说的话->say hi
         返回值：null
         */
    }
}

```



### 5.反射main方法

这边为了方便写两个类用作测试

```java
//Test
package reflect;
public class Test {
    public static void main(String[] args) {
        System.out.println("执行了Test类里的main方法");
    }
}

//TestMain
package reflect;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
public class Testmain {
public static void main(String[] args) throws NoSuchMethodException,   ClassNotFoundException, InvocationTargetException, IllegalAccessException {
            //1、获取Test对象的字节码
            Class clazz = Class.forName("reflect.Test");
            //2、获取main方法 getMethod(方法名,方法形参类型)
            Method methodMain = clazz.getMethod("main", String[].class);
            //3、调用main方法 
            //第一个参数，对象类型，static静态的可为null，第二个参数是String数组，
            methodMain.invoke(null, (Object)new String[]{}); //输出 执行了Test类里的main方法
        }
    }
```



### 6.用于运行配置文件里的内容

本质也是先获取配置文件的内容之后，然后做相应实例化，调用方法等操作，比如说使用quartz需要配置quartz.properties

```properties
name = garwer
#经常看到一些jar包的配置文件在properties配置一些jar包路径 可能用于通过反射机制来实例化或者类加载用 也可记录方法名 类中的成员变量等
path = reflect.Person
```

```java
package reflect;
import java.io.FileReader;
import java.io.IOException;
import java.util.Properties;
public class testProperties {
    //通过key获取value值
    private static String getVal(String key) throws IOException {
        Properties properties = new Properties();
        FileReader in = new FileReader("src/main/resources/my.properties");//获取输入流 classpath:
        properties.load(in);
        in.close();
        return properties.getProperty(key);
    }
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        System.out.println(testProperties.class.getResource("").getPath());
        System.out.println(getVal("name")); //输出garwer
        String path = getVal("path"); //reflect.Person
        System.out.println(path);
        Object o = Class.forName(path);
        System.out.println(o); //class reflect.Person
    }
}
```

## 总结

> 反射主要用于在运行的时候，获取类的信息并操作它，广泛的用于在设计模式和jdk和各种框架中。