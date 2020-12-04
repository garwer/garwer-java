# 1.概述

>HashMap是日常java开发中常用的类之一，是java设计中非常经典的一个类，它巧妙的设计思想与实现，还有涉及到的数据结构和算法，，值得我们去深入的学习。

>简单来说，HashMap就是一个散列表，是基于哈希表的Map接口实现，它存储的内容是键值对 (key-value) 映射，并且键值允许为null(键的话只允许一个为null)。

### 1.1  注意事项
>①根据键的hashCode存储数据。(String，和Integer、Long、Double这样的包装类都重写了hashCode方法，String比较特殊根据ascil码还有自己的算法计算，Double做位移运算【具体看源码的hashcode实现】，Integer，Long包装类则是自身大小int值)，
HashMap中的结构不能有基本类型，一方面是基本类型没有hashCode方法，还有HashMap是泛型结构，泛型要求包容对象类型，而基本类型在java中不属于对象。
②HashMap的存储单位是Node<k,v>,可以认作为节点。
③Hashmap中的扩容的个数是针对size(内部元素(节点)总个数)，而不是数组的个数。比如说初始容量为16，第十三个节点put进来，不管前面十二个占的数组位置如何，就开始扩容。

### 1.2 hashmap几个特征

| 特征            | 说明                                       |
| :------------ | ---------------------------------------- |
| 是否允许重复数据      | key如果重复会覆盖，value允许重复                     |
| hashMap是否有序   | 无序，这里的无序指的是遍历HashMap的时候，得到的顺序大都跟put进去的顺序不一致 |
| hashMap是否线程安全 | 非线程安全，因为里面的实现不是同步的，如果想要线程安全，推荐使用ConcurrentHashMap          |
| 键值是否允许为空      | key和value都允许为空，但键只允许一个为空                  |

# 2.一些概念

### 2.1.位运算

位运算是对整数在内存中的二进制位进行操作。

在java中 >> 表示右移 若该数为正，则高位补0，若为负数，高位补1

<<表示左移 跟右移相反 如果是正数在低位补0

例如20的二进制为0001 0100  20>>2为  结果为5(0000 0101) (左高右低)

20<<2 为 0101 0000 则为80

**java中>>>和>>的区别**

```
>>>表示无符号右移，也叫逻辑右移。不管数字是正数还是负数，高位都是补0
```

在hashMap源码中有很多使用位运算的地方。例如:

```java
//之所以用1 << 4不直接用16，0000 0001 -> 0001 0000 则为16，如果用16的话最后其实也是要转换成0和1这样的二进制，位运算的计算在计算机中是非常快的，直接用位运算表示大小以二进制形式去运行，在jvm中效率更高。
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;  //初始化容量
```

**注意:左移没有<<<运算符**

------



### 2.2 位运算符-(与(&)、非(~)、,或(|)、异或(^))

##### ①与运算(&)

我们都知道&在java中表示与操作&表示按位与，这里的位是指二进制位。都为1才为真(1),否则结果为0，举个简单的例子

```java
System.out.println(9 & 8); //1&1=1，1&0 0&1 0&0都=0，因此1001 1000 -> 1000 输出为8
```
##### ②非运算(~)

**源码 -> 取反 -> 反码 -> 加1 -> 补码 -> 取反 -> 按位非值**

在Java中，所有数据的表示方法都是以补码的形式表示，如果没有特殊说明，Java中的数据类型默认是int,int数据类型的长度是8位，一位是四个字节，就是32字节，32bit.

例如5的二进制为0101

补码后为  00000000 00000000 00000000 00000101

取反后为  11111111 11111111 11111111 11111010

【因为高位为1 所以源码为负数，负数的补码是其绝对值源码取反，末尾再加1】

所以反着来末尾减1得到反码然后再取负数 

末位减1：11111111 11111111 11111111 11111001

【后八位前面4位不动 后面 减1 1010减1  相当于 10-1为9 后四位就是 1001 】

取反后再负数： 00000000 00000000  00000000  00000110  为-6

```java
System.out.println(~ 5); //输出-6
```



##### ③或运算(|)

**只要有一个为1，结果为1，否则都为0**

```java
System.out.println(5 | 15); //输出为15，0101或上1111,结果为1111
```



##### ④异或运算(^)

**相同为0(假)，不同为真(1)**

```java
System.out.println(5 ^ 15); //输出10 0101异或1111结果为1010
```
------



### 2.3 hashcode

hash意为散列，hashcode是jdk根据对象的地址或者字符串或者数字算出来的int类型的数值，顶级父类Object类中含hashCode方法(native本地方法，是根据地址来计算值)，有一些类会重写该方法，比如String类。

重写的原因。为了保证一致性，如果对象的equals方法被重写，那么对象的hashcode()也尽量重写。

简单来说 就是hashcode()和equals()需保持一致性，如果equals方法返回true，那么两个对象的hashCode 返回也必须一样。

否则可能会出现这种情况。

假设一个类重写了equals方法，其相等条件为属性相等就返回true，如果不重写hashcode方法，那么依据就是Object的依据比较两个对象内存地址，则必然不相等，这就出现了equals方法相等但是hashcode不等的情况，这不符合hashcode的规则，这种情况可能会导致一系列的问题。

**因此，在hashMap中，key如果使用了自定义的类，最好要合理的重写Object类的equals和hashcode方法。**

------



### 2.4 哈希桶

哈希桶的概念比较模糊，个人理解是数组表中一块区域结果下面的单向链表组成的，在hashmap中，这个单向链表的头部是所在数组上第一个元素，单向链表如果过长超过8，那么这个"桶"就可能变成了红黑树(前提是数组长度达到64）。

------



### 2.5 hash函数

在程序设定中，把一个对象通过某种算法或者说转换机制对应到一个整形。

主要用于解决冲突的。

------



### 2.6 哈希表

也称为散列表，这也是一种数据结构，可以根据对象产生一个为整数的散列码(hashCode)。


#### 

### hash冲突

HashMap之所以有那么快的查询速度，是因为他的底层是由数组实现，通过key计算散列码(hashCode)决定存储的位置，HashMap中通过key的hashCode来计算hash值，只要hashCode相同，hash值也一样，但是可能存在存的对象多了，不同对象计算出的hash值相同，这就是hash冲突。

**举个例子**

```java
HashMap<String,String> map = new HashMap<String,String>();
map.put("Aa", "haha");
map.put("BB","heihei");
System.out.println("Aa".hashCode()); //2112
System.out.println("BB".hashCode()); //2112
//这里的Aa和BB为String型，String类重写了hashCode方法(根据ascil码和特定的算法来计算，虽然很巧妙但也难以避免不对对象hashCode相同的情况)，Aa和BB的hashCode值相同，相同的HashCode的hash值相同 
//根据源码就算key不相同 但key.hashCode()相同 则会返回相同的hash，导致hash冲突
static final int hash(Object key) {//取关键key的hash值
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);//任何小于2的16次方的数 右移16位都为0 2的16次方>>>16刚好为1 任何一个数和0按位异或都为这个数本身(1和0为1 0和0为0)，所以这个hash()函数对于null的hash值 仅在hashcode大于2的16次方才会调整值,这边16设计的很巧妙，因为int刚好是32位的取中间位数
}
```

###  2.7 二叉查找树和红黑树

红黑树是一种自平衡二叉查找树。是一种数据结构，又称二叉b树，（→_→ 2b树？），红黑树本质上也是二叉查找树。所以先理解下二叉查找树。

#### 2.7.1二叉查找树

```
二叉查找树，又称有序二叉树，已排序二叉树
它的三大特点如下
1.左子树上所有结点的值均小于或等于它的根结点的值。
2.右子树上所有结点的值均大于或等于它的根结点的值。
3.左、右子树也分别为二叉排序树。
```

![二叉树.png](https://upload-images.jianshu.io/upload_images/10006199-7241fa6b9af7271d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


####2.7.2 红黑树(RBTree)

```
由于二叉查找树可能存在难以平衡呈线性的缺陷，所以出现的红黑树的概念。顾名思义，红黑树是只有红色和黑色节点的二叉树。
它的5大性质如下。
1.节点是红色或黑色。
2.根节点是黑色。
3.每个叶子节点都是黑色的空节点（NIL节点）。
4 每个红色节点的两个子节点都是黑色。(从每个叶子到根的所有路径上不能有两个连续的红色节点)
5.从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点。
```

简单来说红黑树是一种自平衡二叉查找树，相比于普通的二叉查找树，它的数据结构更为复杂，但是在复杂的情况也能通过自平衡(变色，左右旋转)保持良好的性能。

关于红黑树，很形象的一组漫画，查看[这里](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653191832&idx=1&sn=12017161025495c6914b5ab9397baa59&chksm=8c990c42bbee8554ba02eb83d839123bd3bead6ffc736111456ea77367a3df75750cf88016e0&scene=21#wechat_redirect)

在线模拟红黑树增删的地址[地址1](https://www.cs.usfca.edu/~galles/visualization/RedBlack.html)、 [地址2](https://sandbox.runjs.cn/show/2nngvn8w)

**红黑树的时间复杂度为【吐槽下简书这边如果用数学公式太蛋疼了】：**
>O(logn)

**它的高度为:[logN,logN+1]（理论上，极端的情况下可以出现RBTree的高度达到2*logN，但实际上很难遇到）。**

此外，由于它的设计任何不平衡将在三次旋转内解决。

```
红黑树和avl树(最早的自平衡二叉树)的比较：
avl更加平衡，查询速率稍强于红黑树，但是插入和删除红黑树完爆avl树，可能由于hashMap的增删也挺频繁的，所以综合考虑而选择红黑树。
```

**总结：红黑树是种可以通过变色旋转的自平衡二叉查找树，对于hashMap来说，使用红黑树的好处在于，当有多个元素hash相同在同一数组下标的时候，使用红黑树在查找这些hash冲突的元素更快，它的时间复杂度从遍历链表O(n)降到O(logN)。**



### 2.8 复杂度

算法复杂度分时间复杂度和空间复杂度。

```
时间复杂度：执行算法所需要的计算工作量
空间复杂度：执行算法所需要内存空间大小
时间和空间都是计算机资源的体现，算法的复杂性体现在运行该算法时计算机所需资源的大小。
```

**这里重点讲下时间复杂度**

```
(1)时间频度
用T(n)表示
一个算法执行所消耗的时间，理论上不能算出来而是通过运行测试得知，但不可能也没必要对每个算法都做上机测试，只需知道哪个算法花费时间多哪个花费少即可。在算法中一个算法花费的时间和这个算法执行的次数成正比。
在一个算法中，语句执行次数称为时间频度(或称为语句频度)，记做为T(n)，这里的n代表问题的规模。暂且不考虑这个T是啥，把它理解为一个函数。
(2)时间复杂度 
用Ｏ(f(n))表示
当n变化时，时间频度T(n)也会不断变化，但是它是个不确定的函数，我们想知道它呈现的规律是什么样的。这个时候引入了时间复杂度的概念。
前面说T(n)是个不确定的函数，它代表算法中基本操作重复执行的次数是问题规模n的某个函数。
假设有某个辅助函数f(n),当n趋近∞，T(n)/f(n)的极限值不为0切位常数，那么可以认为f(n)和T(n)为同一数量级的函数，记做为T(n)=Ｏ(f(n)),称Ｏ(f(n)) 为算法的渐进时间复杂度，简称时间复杂度。

f(n)虽然没有规定但一般都尽可能取简单的函数
例如 O(2n²+n +1) = O (3n²+n+3) = O (7n² + n) = O ( n² ) 省去了系数,只保留最高阶项。
时间频度不同时，时间复杂度有可能相同，例如T(n)=n²+3n+4与T(n)=4n²+2n+1它们的频度不同，但时间复杂度相同，都为O(n²)。

总结两者关系:时间复杂度就是对时间频度函数的一层包装，它的特点(大O表示法)为
①省去系数为1处理②保留最高项
如果把T(n)当做为一棵树，那么O(f(n))只关心其主干部分。
```

**常见算法的时间复杂度从小到大依次为**
![复杂度比较](https://upload-images.jianshu.io/upload_images/10006199-b84d873793e9f4d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
求解算法的时间复杂度具体步骤为：
①找出算法中执行次数最多的基本语句，一般是最内层的循环体。
②计算基本语句的数量级
③将基本语句执行次数的数量级放入大O记号中
```

**举几个例子**

**O(1),又称常数阶，一般来说算法中没有循环体，执行次数为常数那么时间复杂度就为O(1)，例如**

```java
int sum = 0,n = 100; //执行一次  
sum = (1+n)*n/2; //执行一次  
System.out.println (sum); //执行一次 
//上面的算法运行次数为f(n)=3,那么根据大O表示法，该算法的时间复杂度为O(1)
```

#####  

**为什么O(logN)，对数阶不用底数**

**如红黑树的查找复杂为O(logN)**

**这里面有个可能存在的疑问，有时候时间复杂度都用包含O(logN)这样的描述 但是没有明确说明n的底数是多少，通常底数为2来计算**

这种描述其实也是合理的，算法中log级别的时间复杂度都是由于使用了分治思想,这个底数直接由分治的复杂度决定。当n趋近于无穷大，两个大小比较也只是一个常数，所以这种时候O(logN)统一代表对数复杂度。
<!-- $$
\lim_{n\rightarrow+\infty}    Ο(\log_x{n})/Ο(\log_y{n}) = C
$$ -->

**其它简单举例**

| 描述   | 增长数量级 | 典型代码                                     | 说明                                       |
| ---- | ----- | ---------------------------------------- | ---------------------------------------- |
| 常数阶  | 1     | a = b + c                                | 普通简单算法操作                                 |
| 对数阶  | logN  | 二叉树中的二分法                                 | 二分策略                                     |
| 线性级别 | N     | for(int i = 0;i < 10; i++) {...}         | 普通单层循环算法                                 |
| 平方级别 | N²    | for(int i = 0;i < 10; i++) {for(int j = 0; j < 10) {...}} | 双层循环，例如冒泡排序                              |
| 指数级别 | 2的n次方 | 一个背包大小一定时，找出不大于背包所有物品组合，假设有3个物品，a，b，c，可能的组合有8种。(a,b,c,ab,ac,bc,abc+空(背包太小一个都容纳不下)) | 穷举查找(背包问题https://www.cnblogs.com/tinaluo/p/5264190.html) |

****



# 3. HashMap的内部实现(基于jdk1.8)

> 刚开始看hashMap源码的时候，感觉思路很乱不知道写的啥东西，所以还是得从它的【数据结构】开始入手。

![](https://upload-images.jianshu.io/upload_images/10006199-2e0da8e8dfab6a4d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
不同于一般类的数据结构，从结构来讲 HashMap = 数组 + 链表 + 红黑树(1.8开始加入，大程度的优化了HashMap的性能)
arrayList  数组
linkedList 双向链表 查询效率慢，需通过遍历，新增或删除快，比如说删除一个元素 知道那个元素的上下引用 并改变关联上下元素的引用指向即可。
```

### 3.1 数组和链表

![数组和链表.png](https://upload-images.jianshu.io/upload_images/10006199-7500f3506ef6f1d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 3.2 HashMap数据结构(数组+链表+红黑树)

> 在jdk8以前，如果发生频繁碰撞的话，查找时间复杂度是O(1) + O(n) (先找在数组的位置再找链表)，n如果比较大则严重影响了查找性能，而到了jdk8引入红黑树,O(1) + O(logN)。

![hashmap.png](https://upload-images.jianshu.io/upload_images/10006199-8bd24ce7006bb985.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 大致思路

```java
①数组的优点是查询快，链表的优点是增删快，红黑树查询性能较好，hashMap的存储方式结合了它们的优点，那么hashMap的存储单元又可以在数组里，又可以在某个数组下的链表里。还有可能在红黑树当中。
②我们已经知道HashMap是键值对的存在，且可以为各种类型，那么它又是以键值对的方式存在，它的最小存储单位是以Node节点为存储单位。
这个Node结构大概有Key，Value，记录所在数组索引，以及记录链表指针的东西。
大概结构如下
static class Node<K,V> implements Map.Entry<K,V> {
  final int hash;
  final K key;
  V value;
  Node<K,V> next;
  ...
}

③新来的Node节点怎么放?
HashMap利用hashcode来确定存放的位置，但是又有个疑问，假设map对象key为String型
HashMap<String, String> map = new HashMap<String, String>();
map.put("1", "first");

//这个时候看put方法 
put方法的大致思路为
①对key做hash运算，通过hash值计算index下标位置
②如果没冲突直接放在桶上
③如果冲突了，以链表的形式存在桶里面，达到一定条件链表变为红黑树
④如果节点已经存在，则替换旧的value(保证唯一性)
⑤如果桶的个数超过了 加载因子乘当前容量，则做resize操作

//可以注意到有个hash函数
public V put(K key, V value) {
   return putVal(hash(key), key, value, false, true);
}

//hash函数 
static final int hash(Object key) {
   int h;
   return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

//上述代码String类型的1的Hashcode为49超过了HashMap的初始长度16，这个时候"1"这个key放在哪。这里
//通过巧妙的设计存放在合适的位置 4.3.3做分析
p = tab[i = (n - 1) & hash]，


//这里的p为Node<K,V>对象，n为当前哈希桶数组长度，进行与运算后，因为这是第一个插入的元素，无需扩容长度为16,那么49 & 15 = 1，说明在的第二个位置。

④新节点插入后什么时候开始扩容
接下来不断的插入的元素 经过hash函数和计算索引位置后，都可以根据它的散列性插入到不同的16个位置，
当元素个数达到16 * 0.75 即12时，继续插入新的时候，开始扩容。
【这里注意一下并不是说占满12个位置才开始扩容，而是12个节点，根据散列性分布12个节点，占...5，6，7，8...个位置都有可能,比如说key为Integer类型，假如key为Integer类型，有五个节点key分别为3，19，12，28，44这个时候3，19在同一个位置，12，28，44在同一个位置，这个时候5个节点就占了两个位置】


⑤resize()方法进行扩容操作。
1.先判断节点数组是否为空，并取它的容量(节点个数)，创建新数组，大小时新的capacity
如果不为空：
如果容量超过最大值不做扩容，否则位运算一位做容量乘2处理，
如果为空：
桶数组容量为默认容量16，即有默认放16个桶，阈值默认为默认容量乘默认加载因子 12
2.将旧数组的元素放到新数组中，重新做映射
如果旧的数组不为空，则遍历桶数组，并将键值对映射到新的桶数组中[树节点和链表节点做不同操作]
```

# 4.源码分析

### 4.1 基本存储单位Node节点

```java
static class Node<K,V> implements Map.Entry<K,V> { //实现Entry接口 存储的是键值对的映射
    final int hash; //hash值，用于记录数组所在位置
    final K key; //用于匹配
    V value; //值
    Node<K,V> next; //用于记录单链表下一节点 用于解决hash冲突(即hash值一样该存在哪里的问题)
    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }
    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }
    public final V setValue(V newValue) {//赋值
        V oldValue = value;
        value = newValue;
        return oldValue;
    }
    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

### 4.2 HashMap中的几个重要实现：hash函数，put、get、resize

```java
//put
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    //哈希表数组节点 
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //如果为空 调用resize以默认大小16扩容 
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //通过(n - 1) & hash计算存放索引位置 此处设计很巧妙
    if ((p = tab[i = (n - 1) & hash]) == null)
      //如果tab[i]为空 该下标下没有节点 则直接新建一个Node放在该位置 
        tab[i] = newNode(hash, key, value, null);
    else {
        //下标上有节点 说明有hash冲突
        Node<K,V> e; K k;
        //如果插入的新节点key已经存在，那么直接覆盖整个节点
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        //如果为红黑树节点
        else if (p instanceof TreeNode)
            //调用红黑树插入键值对的putTreeVal方法
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //不管tab[index]是否为空，p节点已经为 tab[index]上
            //如果有冲突 且不为红黑树节点 那么此时遍历链表节点 binCount计算链表长度
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                   //链表长度大于8，调用treeifyBin对链表进行树化 -1为第一个
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                //遍历链表时发现重复 覆盖并跳出循环
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    //插入成功后 再根据实际判断是否到到阈值 比如说现在容量16(桶的个数16) 正在插第13个元素时 到达则扩容 
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

### get方法

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    //先定位键值对在所在桶的位置
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node 
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                //如果是红黑树节点 通过红黑树查找方法查找
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                //对链表查找
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```



### 4.4.5 resize()

扩容就是重新定义容量，在hashmap中，如果不断的put元素，而hashMap对象中的数组无法装得下更多对象时，对象就需要进行扩容，扩大数组长度。这边注意的是：

**①假如初始大小为默认值16，什么时候扩容，我们可以知道阈值是16*0.75即12，这个12是指hashMap的size(全局变量，每次put+1.remove-1)，put后为大于12即13时开始执行resize方法扩容。**

②**在java中数组是不能够自动扩容的，是采用一个新的大容量数组代替原有的小数组，就好比用一个小桶装水，如果想用一个桶装更多的水，就换一个大桶再把原来小桶的水装过去。**

**③扩容后，普通链表上的节点包括红黑树都得重新映射。**


>对于hashmap来说
什么时候换大桶：达到阈值的时候
换多大的桶：原有小桶的两倍大小
但桶的大小也是有限的，对于hashMap，最大的桶能容纳包含2^30个数，大于的话就不再扩容，就随里面碰撞了。(实际上也很难用到这么大的容量)


```java
final Node<K,V>[] resize() {
    //table为全局变量transient Node<K,V>[] table; 赋值给oldTab
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;//旧表数组个数
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) { //如果旧容量大于0    
        //超过最大值就不扩容了，随它碰撞去吧 -。-
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        //×2还没超过最大值，新数组就扩容为原来两倍 阈值也做×2处理
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold 
    }
    //如果原来的阈值 > 0且旧容量为0，则将新容量设为原来的阈值，初始化有参给threshold赋值会有此情况
    else if (oldThr > 0) 
        newCap = oldThr;
    else { // zero initial threshold signifies using defaults
        //默认初始化无参构造的情况 
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    //如果
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"}) //屏蔽无关紧要的警告
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    //如果旧数组不为空 
    if (oldTab != null) {
        //遍历数组
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            //数组中的节点不为空
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                //如果该桶只有一个节点(说明下面没有链表，或者说只有一个链表节点)
                if (e.next == null)
                    //e.hash & (newCap - 1)确定元素存放位置
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    //红黑树节点
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { 
                    //链表节点且当前链表节点不止1个
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        //根据e.hash & oldCap 判断节点存放位置
                        //如果为0 扩容还在原来位置 如果为1 新的位置为 旧的index + oldCap 下面如何扩容有做介绍
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);//旧链表迁移到新链表
                    if (loTail != null) {
                        loTail.next = null;//将链表的尾节点的next设置为空
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;// 将链表的尾节点 的next 设置为空
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

### 4.3 HashMap经典代码 p = tab[i = (n - 1) & hash])

```
p = tab[i = (n - 1) & hash])
```

当hashCode小于65536，散列是很规律的，基本上索引的位置就是 

因为小于这个数右移16为都为0，且和占位符都为0的值异或后的hashcode就是自身的值。

这个值比较特殊

转换为二进制：00000000000000010000000000000000，右移16的话00000000000000000000000000000001并不全为0

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

key的hashcode为65536

转为二进制：h=key.hashCode()  00000000000000010000000000000000

跟右移16位的再做异或操作          00000000000000000000000000000001

hash = h ^(h>>>16)			 00000000000000010000000000000001	

​			 

计算hash					 00000000000000010000000000000001	

​							 00000000000000000000000000001111

结果                                                 1

但是65536 % 16 = 0



key的hashcode为17 异或相同为0 不同为假

转为二进制：h=key.hashCode()  00000000000000000000000000010001

跟右移16位的再做异或操作          00000000000000000000000000000000

hash = h ^(h>>16)			 00000000000000000000000000010001		

计算hash                                         00000000000000000000000000010001	

​							 00000000000000000000000000001111

​							 00000000000000000000000000000001



做个小测试，假设这个时候桶的个数为16，代码如下

```java
for (int key = 65533; key < 65543; key++) { //从65536开始变得有点"特别"
    System.out.println("key为：" + key +  "，索引位置：" + ((key ^ (key >>> 16)) & 15));//假设初始容量为16 测试没扩容时这些数的索引位置
}
//输出结果为，可以发现从65536开始不为0而是1，有点特殊，然后相邻两个索引位置呈1,3的增长，具体可画图尝试
i为：65533，输出13
i为：65534，输出14
i为：65535，输出15
i为：65536，输出1
i为：65537，输出0
i为：65538，输出3
i为：65539，输出2
i为：65540，输出5
i为：65541，输出4
i为：65542，输出7
```

这段代码主要是计算索引位置的，HashMap 底层数组的长度总是 2 的 n 次方

当 length 总是 2 的倍数时，h& (length-1)，将是一个非常巧妙的设计：

| hash值 | length(假设长度为16) | h & length - 1 |
| ----- | --------------- | -------------- |
| 5     | 16              | 5              |
| 6     | 16              | 6              |
| 15    | 16              | 15             |
| 16    | 16              | 0              |
| 17    | 16              | 1              |

**可以看到计算得到的索引值总是位于 table 数组的索引之内。并且通常分布的比较均匀**



### 4.4 树形化treeifyBin()

在jdk8以前，如果发生频繁碰撞的话，查找时间复杂度是O(1) + O(n) (先找在数组的位置再找链表)，n如果比较大则严重影响了查找性能，而到了jdk8引入红黑树,O(1) + O(logN)。

jdk1.8中，如果一个桶中元素个数超过TREEIFY_THRESHOLD(8)时，就用红黑树替换链表以提升速度(主要是查找)

```java
//将桶内所有链表节点换成红黑树节点
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    //如果当前哈希表为空 或者哈希表中元素 MIN_TREEIFY_CAPACITY默认为64，对于这个值可以认为，如果节点数组长度小于64，就没必要去进行结构转换，而是通过resize()操作，这样原先一个链表的元素可能会进行重新分配。
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize(); //扩容
  	//大于等于64 就树化 链表上的普通节点变成树节点
    else if ((e = tab[index = (n - 1) & hash]) != null) {      
        TreeNode<K,V> hd = null, tl = null; //定义首、尾节点
        do {
            TreeNode<K,V> p = replacementTreeNode(e, null); //普通节点 -> 树节点
            if (tl == null) //如果尾节点为空 说明还没有根节点
                hd = p; //首节点(根节点) 指向当前节点
            else { //尾节点不为空 
                p.prev = tl; //当前树节点前一个节点指向尾节点
                tl.next = p; //尾节点后一个节点 指向当前节点
            }
            tl = p; 
        } while ((e = e.next) != null); //继续遍历链表
      
        //这个时候只是把Node对象变成TreeNode对象，把单向链表变成双向链表
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```


# 5.思考

### 1.HashMap和HashTable的区别是什么

HashMap和Hashtable都实现了Map接口

>HashMap功能上几乎可以等价于Hashtable，除了HashMap是非synchronized的，并可以接受null(HashMap可以接受为null的键值(key)和值(value)，而Hashtable则不行)。
HashMap是非synchronized，而Hashtable是synchronized，这意味着Hashtable是线程安全的
由于Hashtable是线程安全的也是synchronized，所以在单线程环境下它比HashMap要慢。如果你不需要同步，只需要单一线程，那么使用HashMap性能要好过Hashtable。
HashMap不能保证随着时间的推移Map中的元素次序是不变的。


由于性能问题，以及HashTable处理Hash冲突比HashMap逊色很多，现在HashTable已经很少使用了。但由于线程安全以及以前的项目还在使用，SUN依然还保留着它并没有加Deprecated过时注解。

摘自hashtable源码

> If a thread-safe implementation is not needed, it is recommended to use HashMap in place of Hashtable. If a thread-safe highly-concurrent implementation is desired, then it is recommended to use java.util.concurrent.ConcurrentHashMap in place of Hashtable.

简单来说就是不需要线程安全，那么使用HashMap，如果需要线程安全，那么使用ConcurrentHashMap。

### 2.HashMap为什么线程不安全，如果想要线程安全怎么做

因为hashmap为了性能，它的put，resize等操作都不是同步的，假设两个线程同一时间做put操作,可能最后计算的size并不正确，值得一提的是jdk1.8以前多线程put甚至会导致闭环死循环，1.8开始不会有这个问题但依然存在线程安全问题。

jdk8前的闭环死循环。

这种问题在单线程下不存在，但在多线程下可能引起死循环导致cpu占用过高。


如在rehash的过程中

单线程:5，9在同一个桶中,线程安全

![Snip20190225_1](images/Snip20190225_1.png)

死循环情况

多线程rehash

假设同时加入，触发rehash，假设在线程1,5的下一个指针指向9【刚好线程2获取时间片，做完rehash操作】

线程1接着执行链表指向5，接着处理9

线程2处理的时候9后面已经新增9

如果hash冲突大，同一链表下下有多个节点容易出现这种问题。具体参考[老生常谈，HashMap的死循环](https://www.jianshu.com/p/1e9cf0ac07f4)

jdk1.7使用头插法容易导致死循环
1.8已改为尾插法，虽然避免了死循环问题但仍然线程不安全，所以不要在并发环境下使用HashMap


```
若想要线程安全
1、使用ConcurrentHashMap。(线程安全的hashMap)
2、使用Collections.synchronizedMap(Mao<K,V> m)方法把HashMap变成一个线程安全的Map。
```



### 3.HashMap是怎么解决Hash冲突的

```
在实际应用中，无论怎么构造哈希函数，冲突也难以完全避免。
HashMap根据链地址法(拉链法)来解决冲突,jdk8中如果链表长度大于8且节点数组长度大于64的时候，就把链表下所有节点转为红黑树，位于数组上的节点为根节点，来维护hash冲突的元素，链表中冲突的元素可以通过key的equals()方法来确定。
```



### 4.HashMap是怎么扩容的

先写个例子测试hashMap有没有在扩容。

```java
public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
    HashMap<Integer,String> o = new HashMap<>(1);
    System.out.println(o.size()); //0 size为元素个数
    //扩容条件是 如果没有定义初始容量 默认扩容至16 如果没有 根据put的情况扩容
    //put的过程中 如果插入一个元素过后的size > 阈值(加载因子 * 最近容量)
    /**
     * 代码体现 put后执行
     *   if (++size > threshold)
     *         resize();
     */
    //有定义容量的话会采用大于这个数的最小二次幂 第一次初始化为1 则输出为2 4 5 11  111 11
    HashMap<Integer,String> map = new HashMap<>(1);
    map.put(1, "一");
    //由于方法由final修饰 利用反射机制获取容量值
    Class<?> mapType = map.getClass();
    Method capacity = mapType.getDeclaredMethod("capacity");
    capacity.setAccessible(true); //由于capacity方法由final修饰 暴力获取
    System.out.println("capacity : " + capacity.invoke(map)); //capacity : 2
 
    map.put(2, "二");
    capacity = mapType.getDeclaredMethod("capacity");
    capacity.setAccessible(true);
    System.out.println("capacity : " + capacity.invoke(map)); //capacity : 4 当前容量为2 插入该元素后size为 2 > 2 * 3/4 开始扩容

    //当前容量为4 此时已有2个 3 = 4 * 3/4 不进行扩容
    map.put(3, "三");
    capacity = mapType.getDeclaredMethod("capacity");
    capacity.setAccessible(true);
    System.out.println("capacity : " + capacity.invoke(map)); //capacity : 4 当前容量为2 插入该元素后size为 3 = 4 * 3/4 不扩容

    map.put(4, "四");
    capacity = mapType.getDeclaredMethod("capacity");
    capacity.setAccessible(true);
    System.out.println("capacity : " + capacity.invoke(map));//capacity : 8  当前容量为4 此时已有4个 4 > 4 * 3/4 开始扩容
}
```

上面的例子可以看出put后，hashmap确实有进行扩容，hashMap的扩容机制与其它的集合边长不太一样，它是通过当前hash桶个数乘2进行扩容

hashMap主要是通过resize()方法扩容

假设oldTable的key的hash为15，7，4，5，8，1，hashMap为初始容量为8的数组桶，存储位置如下

| index | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    |
| ----- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| hash  | 8    | 1    |      |      | 4    | 5    |      | 7，15 |

当put一个新元素 假设为9，且加载因子使用默认的0.75，在内存空间中新的存储位置如下

| index | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    | 10   | 11   | 12   | 13   | 14   | 15   |
| ----- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| hash  |      | 1    |      |      | 4    | 5    |      | 7    | 8    | 9    |      |      |      |      |      | 15   |

可以看到扩容之后8跑到了第9个位置，15跑到了第16个位置，旧的8，1，4，5在各自的链表上只有一个节点

根据  **e.hash & (newCap - 1)**  相当于  与上15后，都为自己本身所以位置保持不变

但是链表上不止有一个节点的情况，比如说上面的7，15存放的位置

这个时候是先根据 **e.hash & oldCap**判断元素在数组的位置是否需要移动

比如说 7  & 8  = 0111 & 1000 = 0  ; 15  & 8 = 1111 & 1000 = 1，规律是比较高位的第一个 比如说15为高位，第一个为1，如果高位为1那么与后结果也为1

**当e.hash & oldCap == 0时**

链表上节点位置保持不变

**当e.hash & oldCap == 1时**

链表上节点的位置为原位置的index + oldCap 比如说15，新的索引位置为7+8为15

> 值得一提的是，jdk1.8的resize()方法相比与之前做了点优化，JDK1.7中rehash的时候，旧链表迁移新链表的时候，如果在新表的数组索引位置相同，则链表元素会倒置，但JDK1.8不会倒置，jdk8通过e.hash & oldCap，通过0和1的值均匀把之前的冲突的节点分散到新的bucket了，这样做更为高效。

代码见【4.4.5 resize()方法】

### 5.loadFactor加载因子为何为0.75f

>加载因子是哈希表在其容量自动增加之前可以达到多满的一种尺度，它衡量的是一个散列表的空间的使用程度，负载因子越大表示散列表的装填程度越高，反之越小。
简单来说就是如果加载因子太小，空间利用率低，且太容易扩容对性能不太友好，设置太高，不及时扩容容易导致冲突几率大，将提高了查询成本。所以0.75是很合适的值，经过试验，在理想情况下,使用随机哈希码,节点出现的频率在hash桶中遵循泊松分布【在频率附近发生概率高，向两边对称下降。】
详细见 [为什么HashMap中默认加载因子为0.75](https://www.cnblogs.com/DarrenChan/p/8854859.html)

### 6.hashMap中一般使用什么类型的元素作为key，为什么？
>常用String，Integer这样的key
主要原因为
这些类是Immutable(不可变的)，String和基本类型的包装类规范的重写了hashCode()和equals()方法。作为不可变类天生是线程安全的，而且可以很好的优化比如可以缓存hash值，避免重复计算等等，如果采用可变的对象类型，可能出现put进去就无法查询到的情况。
如果想用自定义的类型作为键，那么需要遵守equals()和hashCode()方法的定义规则且不可变，对象插入到map后就不会再改变。


[HashMap的key可以是可变对象吗？](http://www.cnblogs.com/0201zcr/p/4810813.html)

### 7.源码中为什么要用transient修饰桶数组table

```java
transient Node<K,V>[] table;
```

在java中，被transient关键字修饰的变量不会被默认的序列化机制序列化。

hashMap实现了Serializable接口，通过实现`readObject/writeObject`两个方法自定义了序列化的内容，size不用多说了，一般涉及到大小可以直接计算的就没必要再序列化。

为什么不序列化table？原因有下

> 1.table大多数情况是无法存满的。比如说桶数组容量是16，只put了一个元素，这会造成序列化未使用的部分。造成浪费。
>
> 2.同一个键值对在不同jvm下，所处桶的位置可能是不同的，在不同的jvm下反序列化可能发生错误。(hashmap的get/put/remove等方法刚开始都是通过hash找到键所在的桶位置，就是数组下标，但如果键没有重写hashCode方法，就会调用Object的hashCode方法，而Object的hashcode方法是navtive(本地方法)的，这里的hashcode是对对象内存地址的映射得出的int结果，具体怎么计算不得而知，但是在不同jvm下，可能有不同的hashcode实现，这样产生的hash也不一样)。

### 8.HashMap的key如果为null，怎么查找值

我们知道hashMap只允许一个为null的key，如果key为null，因为key为null，那么hash为0，那么p = tab[i = (n - 1) & hash 也一定为0，所以是从数组上第一个位置的链表下查找。

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

# 6.使用建议

1.默认情况下HashMap的容量是16，但是，如果用户通过构造函数指定了一个数字作为容量，那么Hash会选择大于该数字的第一个2的幂作为容量。(1->2、7->8、9->16)

> 在初始化HashMap的时候，应该尽量指定其大小。尤其是当你已知map中存放的元素个数时。（《阿里巴巴Java开发规约》）

这边可以看下hashMap的4个构造方法，一般采用3，但如果已经知道个数，建议用2(加载因子0.75很合适不建议改动)

```java
//1 自定义传初始容量和加载因子
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}

//2 自定义初始大小 调1构造方法，加载因子使用默认大小
public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

//3 最常用的无参构造方法
public HashMap() {
	this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

//4 将别的map对象映射到自身存储，很少用
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```



这边讲解一下tableSizeFor方法。简述一下该方法的作用：

如果自定义容量大小时(调1或2的构造方法)，传入一个初始容量大小，**大于输入参数且最近的2的整数次幂的数**。比如10，则返回16，75返回128

> 不这么做的缺点
>
> 假设HashMap需要放置1024个元素，由于没有设置初始容量大小，随着元素不断增加，容量7次被迫扩大。而resize过程需要重建hash表，这会严重影响性能。

```java
/**
 * Returns a power of two size for the given target capacity.
 */
static final int tableSizeFor(int cap) {
  	//cap-1的目的是因为如果cap是2的幂数不做-1操作的话 那么最后执行完右移操作的话，返回的值将会是原有值得两倍。如果n为0的话，即cap=1，经过后面几次操作返回的为0，最后返回的capacity仍然为1(最后有加1的操作)
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

**解释一下这段代码**

在java中，|=的作用是比较两个对象是否相等

a|=b的意思就是把a和b按位或然后赋值给a

以10为例整体流程大致如下

![算法流程](https://upload-images.jianshu.io/upload_images/10006199-a84c34cce64688e6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

简单来说，这种运算最后会导致1占满了它自己所占位，比如说250，它的二进制为

11111010，经过上面的或运算之后，最终将变为11111111，这种情况在加上1，就是大于这个数的最小二次幂。



# 7.总结

HashMap的设计与实现十分的巧妙。jdk8更是有很多提升，还没写这篇博客对于HashMap的理解仅仅只在表面。阅读源码后才发现里面还有不少的学问，由于本人水平有限，虽然花了很多时间写了很多但还有很多细节并不了解，比如说红黑树的代码实现细节，也有可能有几个地方描述错误或者不到位，如果文章有误请指正，以便于我及时修改和学习。


# 8.参考链接

[HashMap 源码详细分析(JDK1.8) ](http://www.coolblog.xyz/2018/01/18/HashMap-%E6%BA%90%E7%A0%81%E8%AF%A6%E7%BB%86%E5%88%86%E6%9E%90-JDK1-8/)

[HashMap resize方法的理解（一)](https://www.cnblogs.com/woniu4/p/8301099.html)

[JDK 源码中 HashMap 的 hash 方法原理是什么](https://www.zhihu.com/question/20733617)

[hashMap死循环问题](https://www.cnblogs.com/dongguacai/p/5599100.html)

[浅谈jdk8为何线程不安全](https://blog.csdn.net/LovePluto/article/details/79712473)

