

## 先谈谈JVM

这篇文章主要是讲class文件和类加载机制，但是整个过程都和jvm密切相关，所以先从jvm说起。

![JVM.jpg](https://upload-images.jianshu.io/upload_images/10006199-f1c1c9bc291bb1bd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



Java之所以跨平台【我们把处理器和操作系统的整体称为平台】，得益于java编译后生成的存储字节码的文件，即class文件以及虚拟机的实现。

#### JVM

JVM(Java Virtual Machine) 就是我们平时一直说的java虚拟机，是一种软件实现，是整个java实现跨平台的核心部分，可以运行class格式的类文件，jvm屏蔽了平台，使得java程序只需要在各自平台上的虚拟机上运行，可以实现同一个class文件跨平台运行。



#### JVM、JDK、JRE的关系

```
JRE： Java Runtime Environment   --java运行环境,  不包含开发工具(编译器,调试器),包含jvm
JDK：Java Development Kit  --java开发工具包 包含jre
```

JRE: 包含了java虚拟机，java基础类库。是使用java语言编写的程序运行所需要的软件环境，是提供给想运行java程序的用户使用的。
JDK:编写java程序所需的开发工具包，供开发使用。JDK包含了JRE，同时还包含了编译java源码的编译器javac，还包含了很多java程序调试和分析的工具：jconsole，jvisualvm等工具软件，还包含了java程序编写所需的文档和demo例子程序。

运行java程序，安装JRE就可以了。

编写java程序，安装JDK即可。

> 三者关系是jdk包含jre，jre包含jvm，如果要用图片描述，如图所示。(其中jre除了包含jvm，还包含了类似rt.jar这样的基础类库和其它软件环境，jdk除了包含jre还包含上述的其它工具等)

![关系图.png](https://upload-images.jianshu.io/upload_images/10006199-f0a68b5501b14151.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)







#### JVM实例和jvm执行引擎实例

- jvm实例 对应一个独立运行的java程序，进程级别当一个java程序启动时，一个jvm实例诞生，当该程序关闭时，jvm实例随之消亡。如果一个计算机运行多个java程序，每个java程序都运行它自己的jvm实例。
- jvm执行引擎实例则是对应所属程序的线程，线程级别

#### jvm生命周期

- 启动：当启动一个java程序时，一个jvm实例就诞生了，任何一个拥有main方法的class都可以作为jvm实例运行的起点。
- 运行：main()函数作为程序初始线程起点，其它线程由该线程启动，包括守护线程(daemon)和non-daemon(普通线程)。守护线程是JVM自己使用的线程比如GC线程就是个守护线程，只要这个jvm实例还有普通线程执行，就不会停止，但是可以用exit()强制终止程序。
- 消亡：所有非守护线程退出时，JVM实例结束生命，若安全管理器允许，程序也可以使用java.lang.Runtime类或者System.exit(0)来退出。实际上exit也是用到Runtime类来退出，Runtime是个神奇的类，它还可以用于启动和关闭非java进程。

```java
//查看java.lang.System类的部分源码    
public static void exit(int status) {    	
	Runtime.getRuntime().exit(status);   
}
```



## 再谈谈Class文件

> 各种不同平台的虚拟机和平台都统一使用一种存储格式-字节码，是和平台无关的基石

```
大致过程为
java编译器将java源程序生成与平台无关的字节码文件(即class文件)，然后jvm对字节码文件解释执行，不同系统的jvm解释器将其解释执行为各自的机器编码。
可以看出：java依赖于jvm，jvm给java提供了运行环境，但解释器与平台相关，不同系统有不同的jvm，因此可以说java是跨平台的【虽说跨平台，但也要考虑jdk版本的差异性】，但jvm不是跨平台的。
```

图片[来源](https://blog.csdn.net/luanlouis/article/details/50529868#comments)

![平台无关性.png](https://upload-images.jianshu.io/upload_images/10006199-42be803fc0148b06.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

虚拟机不关心class的来源是什么语言，java，scala，JRuby这些基于jvm的语言等通过各自的编译器都可以生成class文件。jvm和java准确来说没啥关系，准确来说应该是和class文件有关系，但是java生成class需要jvm。

![](https://upload-images.jianshu.io/upload_images/10006199-d8c2957d6d5b0532.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 一些概念

### 字节码(Byte-code)

不同平台的虚拟机与所有平台统一使用的程序存储格式。是一种包含执行程序，由一序列 op 代码/数据对组成的二进制文件，是一种中间码。*字节*是电脑里的数据量单位。



### class文件

class文件是一组以8位字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑排在class文件，中间无分隔符，这使得class文件里几乎都是程序运行的必要数据。【当遇到需占用8位以上字节的数据时，则会按高位在前的方式分割成多个8位字节存储】

```
class文件是一组以8位字节为基础单位的二进制流。
```

> 这边以前一直有个问题困扰着我，class文件是不是二进制文件。因为在看深入理解jvm虚拟机的时候看到这么一句话，但是用16进制查看工具查看class文件的时候又出现包括有CAFEBABE等内容。而且如果class文件如果是二进制，那么计算机直接就能运行了，为什么还需要jvm呢。

```
计算机最后只认0和1(即二进制)，即使是复杂的程序最终到cpu执行层面还是一串串0和1的指令。
【class文件其实是特殊的二进制文件，用UltraEdit打开确实是0和1，用hex这样的16进制工具打开为16进制】Class文件中包含了Java虚拟机指令集和符号表以及若干其他辅助信息。

class文件并不是机器语言而是二进制文件
机器语言指的是硬件能直接运行的二进制指令代码
```



### 魔数与class文件

每个Class文件的头4个字节称为魔数（Magic Number),它唯一作用就是用来确定文件是否能被虚拟机接受。

很多文件存储标准中都用魔数进行身份标识，如图片gif，jpeg都在文件头部中存储着魔数。使用魔数而不是用扩展名来进行识别主要是基于安全考虑，因为扩展名我们可以随意通过重命名等方式改动。

> 有趣的是class文件的魔数很贴切java的图标含义-CAFFBABE，即咖啡宝贝。之所以说是4个字节，因为这是用16进制打开的，比如说CA实际上是一个字节。

紧跟在魔数4个字节后的是版本号，这里第五个字节和第六个字节是次版本号，第七和第八字节是主版本号。



按十进制的话，jdk1.7是51，jdk1.8是52，这里我使用jdk1.8编译生成的某个class文件用Hex Fiend打开，这边是1.8说明可以被1.8以上版本的虚拟机运行，但是不能被以下的运行。



下面编译一个基础的java类



![魔数和版本号](https://upload-images.jianshu.io/upload_images/10006199-2ebec11ac45c8224.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图上主版本号为第7，8字节，就是00 34，之所以两位代表一个字节是因为在16进制中

```
1、1字节 = 8位（8个二进制位） 1Byte = 8bit；
2、一个十六进制 = 4个二进制位
3、1字节 = 2个十六进制
```



class文件采用一种伪结构存储数据，这种结构只有两种数据类型。1.无符号数 2.表

| 存储类型 | 含义                                  | 举例[类型-名称-数目]                      |
| ---- | ----------------------------------- | --------------------------------- |
| 无符号数 | 基本数据类型，以u1，u2，u4，u8一定字节数目的无符号数，     | u4-magic-1   u2-methods_count-1   |
| 表    | 由多个无符号数或者其它表作为数据项构成的复合数据类型，以_info结尾 | method_info-methods-methods_count |

无论是无符号数还是表。当需要描述同一类型但不明确数量时，经常使用前置的容量计数器加上若干个连续的数据项形式【如列表中methods_info类型】



###jvm常量池

常量池跟在主次版本后，跟着是常量池入口【由于常量的数量不固定，所以先放个u2类型的数据代表常量池容量计数器。】

**常量池中每一项常量都是一个表,jdk1.7有14种结构不同的表结构，这14个表有个共同特点，就是表开始的第一位都是一个u1类型的标志位，JVM根据这个标志位[tag]来确定某个常量池项表示什么类型的字面量，比如tag为1就是指CONSTANT_utf8_info**

![常量池.png](https://upload-images.jianshu.io/upload_images/10006199-f2919f82736e3919.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


常量池类型表【可以看到为】

| 类  型                             | 标   志 | 描  述        |
| -------------------------------- | ----- | ----------- |
| CONSTANT_utf8_info               | 1     | UTF-8编码的字符串 |
| CONSTANT_Integer_info            | 3     | 整形字面量       |
| CONSTANT_Float_info              | 4     | 浮点型字面量      |
| CONSTANT_Long_info               | 5     | 长整型字面量      |
| CONSTANT_Double_info             | 6     | 双精度浮点型字面量   |
| CONSTANT_Class_info              | 7     | 类或接口的符号引用   |
| CONSTANT_String_info             | 8     | 字符串类型字面量    |
| CONSTANT_Fieldref_info           | 9     | 字段的符号引用     |
| CONSTANT_Methodref_info          | 10    | 类中方法的符号引用   |
| CONSTANT_InterfaceMethodref_info | 11    | 接口中方法的符号引用  |
| CONSTANT_NameAndType_info        | 12    | 字段或方法的符号引用  |
| CONSTANT_MethodHandle_info       | 15    | 表示方法句柄      |
| CONSTANT_MothodType_info         | 16    | 标志方法类型      |
| CONSTANT_InvokeDynamic_info      | 18    | 表示一个动态方法调用点 |

[该图片出处](http://www.sohu.com/a/131458551_504186)

![常量池各个表结构](https://upload-images.jianshu.io/upload_images/10006199-0bb9d543827070f4.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里举个例子，这里编写个非常简单的java类

```java
package jvm;
public class Pool {
    private String b = "常量池";
}
```

编译成class过后，用16进制工具打开，文本大概是这样的

```
cafe babe 0000 0034 0016 0a00 0500 1108
0012 0900 0400 1307 0014 0700 1501 0001
6201 0012 4c6a 6176 612f 6c61 6e67 2f53
7472 696e 673b 0100 063c 696e 6974 3e01
0003 2829 5601 0004 436f 6465 0100 0f4c
696e 654e 756d 6265 7254 6162 6c65 0100
124c 6f63 616c 5661 7269 6162 6c65 5461
626c 6501 0004 7468 6973 0100 0a4c 6a76
6d2f 506f 6f6c 3b01 000a 536f 7572 6365
4669 6c65 0100 0950 6f6f 6c2e 6a61 7661
0c00 0800 0901 0009 e5b8 b8e9 878f e6b1
a00c 0006 0007 0100 086a 766d 2f50 6f6f
6c01 0010 6a61 7661 2f6c 616e 672f 4f62
6a65 6374 0021 0004 0005 0000 0001 0002
0006 0007 0000 0001 0001 0008 0009 0001
000a 0000 0039 0002 0001 0000 000b 2ab7
0001 2a12 02b5 0003 b100 0000 0200 0b00
0000 0a00 0200 0000 0300 0400 0400 0c00
0000 0c00 0100 0000 0b00 0d00 0e00 0000
0100 0f00 0000 0200 10
这边u4类型的魔数为cafebabe，u2类型的次版本号为0034，16进制的34转为10进制为52，说明是jdk1.8编译生成的class文件，
次版本号的后面跟着就是常量池的东西了
首先入口是u2类型的0016说明有22-1即21个常量
```

|                常量池对应16进制                 | tag                          | 对应结构                        | 除了tag其它说明【对照表格】                          | 顺序   |
| :--------------------------------------: | ---------------------------- | --------------------------- | ---------------------------------------- | ---- |
|               0a 0005 0011               | 0a[ CONSTANT_Methodref_info] | {tag:u1 index:u2 index:u2}  | 第一个u2类型的描述指向CONSTANT_Class_info的index[5]，第二个 u2类型描述指向CONSTANT_NameAndType_info的index[此处11转10进制为17]组成 | 1    |
|                 08 0012                  | 08[ CONSTANT_String_info]    | {tag:u1 index:u2}           | 这里的u2描述指向字符串字面量索引(18),说明这个字面量的位置在第18个常量项中 | 2    |
|               09 0004 0013               | 09[CONSTANT_Fieldref_info]   | {tag:u1 index:u2 index:u2}  | u2:指向CONSTANT_Class_info的索引项[4] u2:指向CONSTANT_NameAndType_info的索引项[19] | 3    |
|                 07 0014                  | 07[CONSTANT_Class_info]      | {tag:u1 index:u2}           | u2:指向类的全限定名的索引[20]                       | 4    |
|                07    0015                | 07[CONSTANT_Class_info]      | {tag:u1 index:u2}           | u2:指向类的全限定名的索引[21]                       | 5    |
|              01    0001 62               | 01[CONSTANT_utf8_info]      | {tag:u1 length:u2 bytes:u1} | 长度为1的utf-8编码字符串 这里b的utf-8编码就是62(这里两个为一个字节) | 6    |
| 01 0012 4c6a 6176 612f 6c61 6e67 2f53 7472 696e 673b | 01[CONSTANT_utf8_info]      | {tag:u1 length:u2 bytes:u1} | 长度为length(18)的utf-8编码字符串，就是从4c6a到673b共有18个字节，而4c6a 6176 612f 6c61 6e67 2f53 7472 696e 673b的utf-8编码刚好为Ljava/lang/String;具体可到http://hexutf8.com/验证 | 7    |
|  01  0006 3c69 6e69 743e 0100 0328 2956  | 01[CONSTANT_utf8_info]      | {tag:u1 length:u2 bytes:u1} | 长度为6， 3c69 6e69 743e  即<init>            | 8    |
|              01 0003 282956              | 01[CONSTANT_utf8_info]      | {tag:u1 length:u2 bytes:u1} | length:3 282956 即()V                     | 9    |
|            01 0004 436f 6465             | 01[CONSTANT_utf8_info]      | {tag:u1 length:u2 bytes:u1} | length:4 436f 6465 即Code                 | 10   |
|                 ...依次类推                  | ...                          | ...                         | ...                                      |      |
|        01 0008 6a 766d 2f50 6f6f         | 01[CONSTANT_utf8_info]      | {tag:u1 length:u2 bytes:u1} | length:8 6a 766d 2f50 6f6f 即jvm/Pool     | 20   |
| 01 0010 6a61 7661 2f6c 616e 672f 4f626a65 6374 | 01[CONSTANT_utf8_info]      | {tag:u1 length:u2 bytes:u1} | length:16 6a61 7661 2f6c 616e 672f 4f626a65 6374 即java/lang/Object | 21   |

常量池作为占用class文件空间最大的数据项目之一【数目不固定，可以很大】，明明就写了几行代码却包含这么多各种信息，如果一一看16进制文件再比对含义会十分繁琐。

好在oracle提供了javap -verbose的命令可以帮助我们输出常量表【还是以上面的Pool.class为例】

输出结果如下
!](https://upload-images.jianshu.io/upload_images/10006199-1bd26a98e62b9622.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 注意点

```
1.常量池可以理解为class文件中的资源仓库，有很多种类型，主要存放两大常量
①.字面量 
字面量就是通俗理解的java常量，如文本字符串，final修饰的常量值等
②.符号引用
符号引用属于编译原理的概念，主要包含以下三种
Ⅰ类和接口的全限定名
Ⅱ字段的名称和描述符
Ⅲ方法的名称和描述符
2.不同与其它计数器，常量池容量计数器它是以1开头，之所以这样是因为要满足需要表达"不引用任何一个常量池项目"这样的特定情况，这种情况用0表示。

值的一提的是，class文件中的方法、字段都需要引用CONSTANT_Utf8_info型常量来描述名称，所以受它最大长度的限制【u2型能表示的最大长度为65535bytes即64kb(注意这里指的是长度的大小)】，因此超过64KB的变量或方法名，将无法编译【正常也没人写那么长 = =】。
```





###运行时常量池、字符串常量池的区别

##### 运行时常量池: 就是上述javap -v后后描述的的常量池，包括有字面量引用和符号引用。

运行时常量池存在方法区中，相较于class常量池，运行时常量池更具有动态性。class文件中除了有类的版本字段方法接口等描述信息外，还有一项信息是常量池用于存放编译器生成的字面量和符号引用

- 字面量：1.文本字符串 2.八种基本类型的值 3.被声明为final的常量等 【字面量可以使用string.intern()动态添加】
- 符号引用：1.类和方法的全限定名  2.字段的名称和描述符  3.方法的名称和描述符 

![引用类型.png](https://upload-images.jianshu.io/upload_images/10006199-8519e8da13c62d9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 字符串常量池

```
在目JDK1.7的HotSpot中，已经把原本放在永久带的字符串常量池移除，之后的字符串常量池应该是放在堆中。
```



举个很常见的例子

```java
 public static void main(String[] args) {
   String s1 = "A";
   String s2 = new String("A");
   System.out.println(s1 == s2);// false
 }
```



### 访问标志

常量池后面紧跟的是两个字节的访问标志【u2】。它的作用是正如其名，用于识别一些类或者接口的访问信息，比如这个Class是类还是接口【class文件也有可能代表的是接口】，是否声明为public，是否为abstract类型，如果是类是否被声明为final。

访问标志占两字节，共16位。

在<<深入理解jvm>>一书中是用一系列标志值来表示，参考了[访问标志、类索引、父类索引、接口索引集合](https://blog.csdn.net/luanlouis/article/details/41039269)一文，感觉用2进制的方式更好理解点，比如说ACC_ABSTRCT【是否为abstract类型】用0x0400表示，统一都是转换为16进制的方式，可以更细化点用2进制的方式表示。

0x0400 = 0000 0100 0000 0000

访问标志实际上就是一系列组合，因为有16位所以共有16个标志可以使用，但是目前就定义了8个，不知道jdk9和10是否也是。这8个如图所示。

![访问标志含义.png](https://upload-images.jianshu.io/upload_images/10006199-45d8d4d7636b119a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![访问标志.png](https://upload-images.jianshu.io/upload_images/10006199-10ed7fa914e974b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

还是以上述class文件为例，常量池结束后，0021即为访问标志，转为二进制后为 0000 0000 0010 0001

为0000 0000 0010 0000【ACC_SUPER】与上0000 0000 0000 0001【ACC_PUBLIC】的组合，

通过javap -verbose命令也可以看到访问标志

![](https://upload-images.jianshu.io/upload_images/10006199-c1fb0fc6cd242383.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 类索引、父类索引、接口索引集合

```
类索引:一个java类可能有多个类，这样编译出来的class文件不只是一个，但每个class文件只表示一个类，类索引就是用来确定这个类叫什么名字【全限定名】。
父类索引:java不支持多继承，就是除了Object，每个类都会有且只有一个父类【如果没有继承除了Object的类，就默认继承Object】，父类索引的作用就是描述类继承自哪个类。
接口索引集合:与其他两项不同，接口索引是集合，因为一个类可以实现多个接口，接口索引集合分两部分，入口是索引数，用一个u2类型表示索引的容量，如果类没有实现任何接口，则该计数器值为0，后面的索引表不占任何字节。如果有实现n个接口，后面就用n个索引分别描述各自的接口索引位置。
```

还是以刚才的class文件为例

类索引【u2】、父类索引【u2】、接口索引集合【u2容量+对应索引】

0004 0005 0000



这边类索引【0004】对应常量池中的第四个，而第四个的CONSTANT_Class_info又指向第20个，即jvm/Pool，说明它的全限定名就是这个。



父类索引【0005】对应常量池中的第五个，第五个指向21，即它的全限定名为java/lang/Object。该类没有继承其它类，因此默认继承Object这个祖宗。



因为刚才那个类没有实现任何接口，所以此处为0000后续也都不是接口索引集合的范畴内



这边再举一个有实现接口的例子

简要写一个类和一个接口

```java
package jvm;
public class Aimpl implements A {
}
```

```java
package jvm;
public interface A {
}
```

Aimpl.class文件二进制如下

```
cafe babe 0000 0034 0012 0a00 0300 0e07
000f 0700 1007 0011 0100 063c 696e 6974
3e01 0003 2829 5601 0004 436f 6465 0100
0f4c 696e 654e 756d 6265 7254 6162 6c65
0100 124c 6f63 616c 5661 7269 6162 6c65
5461 626c 6501 0004 7468 6973 0100 0b4c
6a76 6d2f 4169 6d70 6c3b 0100 0a53 6f75
7263 6546 696c 6501 000a 4169 6d70 6c2e
6a61 7661 0c00 0500 0601 0009 6a76 6d2f
4169 6d70 6c01 0010 6a61 7661 2f6c 616e
672f 4f62 6a65 6374 0100 056a 766d 2f41
0021 0002 0003 0001 0004 0000 0001 0001
0005 0006 0001 0007 0000 002f 0001 0001
0000 0005 2ab7 0001 b100 0000 0200 0800
0000 0600 0100 0000 0300 0900 0000 0c00
0100 0000 0500 0a00 0b00 0000 0100 0c00
0000 0200 0d
```

根据刚才的经验，先javap -verbose Aimpl找到常量池最后一个,是 jvm/A,在转hex编码后为

0021开始是访问标志，0000 0000 0010 0001说明为ACC_PUBLIC, ACC_SUPER的组合。后面则是这几个索引和集合信息。

```
类索引【u2】、父类索引【u2】、接口索引集合【u2容量+对应索引】
分别对应0002 0003 0001(1个) 0004(接口位于常量池第四个)
这里不一一查看16进制编码，对应javap -verbose Aimp做验证。类为jvm/Aimpl，父类为java/lang/Object，接口数目为1，接口全限定名为jvm/A
Constant pool:
   #1 = Methodref          #3.#14         // java/lang/Object."<init>":()V
   #2 = Class              #15            // jvm/Aimpl
   #3 = Class              #16            // java/lang/Object
   #4 = Class              #17            // jvm/A
   #5 = Utf8               <init>
   #6 = Utf8               ()V
   #7 = Utf8               Code
   #8 = Utf8               LineNumberTable
   #9 = Utf8               LocalVariableTable
  #10 = Utf8               this
  #11 = Utf8               Ljvm/Aimpl;
  #12 = Utf8               SourceFile
  #13 = Utf8               Aimpl.java
  #14 = NameAndType        #5:#6          // "<init>":()V
  #15 = Utf8               jvm/Aimpl
  #16 = Utf8               java/lang/Object
  #17 = Utf8               jvm/A
```



### 字段表集合

字段表用于描述接口或类中声明的变量、字段，包括类级变量及实例级变量，**不包含局部变量【方法内的变量】**

在java中，我们可以用【public、private、protected】描述字段的作用域，可以用static描述是不是类变量，final描述其可变性，volatile描述可见性【是否强制从主存读写】，transient是否可以被序列化，这些都可以用是否来描述，jvm用访问标识的方式来确认这些信息。

```
字段表集合是由若干个字段表【field_info】组成的集合,jdk编译成class文件后将先把字段个数计算好并设到字段计数器中，在把相应字段信息依次设到字段表集合【类似数组】的结构中。
```

JVM规范了field_info表，来描述字段，结构如图所示
![field_info](https://upload-images.jianshu.io/upload_images/10006199-3f97da43dcab9eac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




#### 字段访问标志

如同类，平时我们描述字段的时候也有访问标志，但是因为是描述字段用的又有些不同。字段访问标志共有9种，具体如图所示
![字段访问标志](https://upload-images.jianshu.io/upload_images/10006199-eb9d754011752df6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


用16位的方式各自位置为1说明被标志位对应的修饰符修饰。

```java
//测试简单java类
package jvm;
public class Field {
    public String a = "a";
    private volatile String b = "b";
}

编译后
cafe babe 0000 0034 0019 0a00 0700 1408
0008 0900 0600 1508 000a 0900 0600 1607
0017 0700 1801 0001 6101 0012 4c6a 6176
612f 6c61 6e67 2f53 7472 696e 673b 0100
0162 0100 063c 696e 6974 3e01 0003 2829
5601 0004 436f 6465 0100 0f4c 696e 654e
756d 6265 7254 6162 6c65 0100 124c 6f63
616c 5661 7269 6162 6c65 5461 626c 6501
0004 7468 6973 0100 0b4c 6a76 6d2f 4649
656c 643b 0100 0a53 6f75 7263 6546 696c
6501 000a 4649 656c 642e 6a61 7661 0c00
0b00 0c0c 0008 0009 0c00 0a00 0901 0009
6a76 6d2f 4649 656c 6401 0010 6a61 7661
2f6c 616e 672f 4f62 6a65 6374 0021 0006
0007 0000 0002 0001 0008 0009 0000 0042
000a 0009 0000 0001 0001 000b 000c 0001
000d 0000 0043 0002 0001 0000 0011 2ab7
0001 2a12 02b5 0003 2a12 04b5 0005 b100
0000 0200 0e00 0000 0e00 0300 0000 0300
0400 0400 0a00 0500 0f00 0000 0c00 0100
0000 1100 1000 1100 0000 0100 1200 0000
0200 13
javap -verbose Field后
Constant pool:
   #1 = Methodref          #7.#20         // java/lang/Object."<init>":()V
   #2 = String             #8             // a
   #3 = Fieldref           #6.#21         // jvm/FIeld.a:Ljava/lang/String;
   #4 = String             #10            // b
   #5 = Fieldref           #6.#22         // jvm/FIeld.b:Ljava/lang/String;
   #6 = Class              #23            // jvm/FIeld
   #7 = Class              #24            // java/lang/Object
   #8 = Utf8               a
   #9 = Utf8               Ljava/lang/String;
  #10 = Utf8               b
  #11 = Utf8               <init>
  #12 = Utf8               ()V
  #13 = Utf8               Code
  #14 = Utf8               LineNumberTable
  #15 = Utf8               LocalVariableTable
  #16 = Utf8               this
  #17 = Utf8               Ljvm/FIeld;
  #18 = Utf8               SourceFile
  #19 = Utf8               FIeld.java
  #20 = NameAndType        #11:#12        // "<init>":()V
  #21 = NameAndType        #8:#9          // a:Ljava/lang/String;
  #22 = NameAndType        #10:#9         // b:Ljava/lang/String;
  #23 = Utf8               jvm/FIeld
  #24 = Utf8               java/lang/Object
  
根据经验，字段为0002，有两个，且field_info结构为访问标志+名称索引+描述索引+属性计数器 都为u2类型
①0001 0008 0009 0000 表示ACC_PUBLIC,a,Ljava/lang/String; 0
②0042 000a 0009 0000 访问标志0000000001000010即ACC_VOLATILE和ACC_PRIVATE,b,Ljava/lang/String;0
```

```
在字段表中，变量修饰符使用标志位表示，字段数据类型和字段名称则引用常量池中常量表
```



### 方法表集合

类似字段表集合，方法表集合是由若干个**方法表（method_info）**组成的集合，然后有2个字节的方法计数器，后面依次是每个method_info的信息。

如图所示，方法表集合是由**访问标志(access_flags)、名称索引(name_index)、描述索引(descriptor_index)、属性表集合(attribute_info)**组成。

#### 访问标志

![方法表.png](https://upload-images.jianshu.io/upload_images/10006199-ddda9caa9426eaaa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

类似类、字段的访问标志，方法表集合也有访问标志，不过不同的是volatile、transient不能修饰方法所以没有这两项，与之相对的是多了synchronized、native、stricfp可以修饰方法的关键字。

#### 名称索引

指向常量池

举个简单的例子

```

```



一个类中代码量最多的往往也就是方法，方法的访问标志、名称索引、描述符索引都描述很清楚，可是方法内部的大量代码在哪，实际上是放在方法表集合中一个名为"Code"的属性当中,这部分归属在属性表集合中。



### 属性表集合

在Class文件、字段表、方法表都可以携带自己的属性表集合，用于描述某些场景专有的信息。

属性表集合相对其他class文件，对顺序要求不再那么高。

属性表占着非常大的一部分且定义了众多属性【jdk7中有21项，[21项简要介绍](https://www.cnblogs.com/lrh-xl/p/5351182.html),更高的版本就不懂了】

这边重要学习了两种属性

#### Code属性

code属性比较复杂，它是经过编译器编译成字节码指令之后的数据。就是说java程序中的方法体经过javac编译器处理后，最终变成字节码存储在Code属性内。

```
并非所有方法表都有这个属性，接口和抽象类就没有【没有方法体】。
```

#### Code属性表结构

| 类型             | 名称                     | 数量               | 描述                                       |
| -------------- | ---------------------- | ---------------- | ---------------------------------------- |
| u2             | attribute_name_index   | 1                | 指向CONSTANT_UTF_8常量索引。固定为Code，表示属性名称      |
| u4             | attribute_length       | 1                |                                          |
| u2             | max_stack              | 1                | 操作数栈深度最大值                                |
| u2             | max_locals             | 1                | 局部变量表所需存储空间，单位为Slot，对于byte，short等长度不超过32位的数据用1Slot，double和long这样的用两Slot |
| u4             | code_length            | 1                | 存储java源程序编译后字节码指令长度【虽然是u4类型，但虚拟机规范要求它用两个字节，超过java编译器则会拒绝编译】 |
| u1             | code                   | code_length      | java源程序编译后字节码，根据长度知道范围，单指令u1字节，最多可以存256种，当前jvm规范提供约200种编码表示对应指令含义，需对应[虚拟机字节码指令表](https://segmentfault.com/a/1190000008722128?utm_source=tag-newest) |
| u2             | exception_table_length | 1                | 异常表长度                                    |
| exception_info | exception_table        | exception_length | 异常                                       |
| u2             | attributes_count       | 1                | 属性长度                                     |
| attribute_info | attributes             | attributes_count | 属性表【包含LineNumberTable、LocalVariableTable、SourceFile、ConstantValue、Deprected等等属性】 |

```
Code属性是Class文件中最重要的一个属性，在Class文件中，Code属性用于描述代码，所有的其它数据项目都用来描述元数据，了解code属性对了解字节码执行引擎来说是必要基础。
```



#### ConstantValue属性

```
之所以学习这个，是因为后面类加载机制有联系到这个属性
```

这个属性的作用是通知虚拟机为静态变量赋值，只要被static修饰的变量才有这个属性，【有该属性的字段必须有ACC_STATIC访问标志，反过来不一定】。

```
对于 "int x = 123" 和 "static int x =123"这类代码在日常编写中很常见，但虚拟机对这两种变量赋值的时刻却不同。
对于非static变量[实例变量]，是在实例构造器<init>进行
对于类变量,有两种方式选择
①在类构造器<clinit>方法中赋值
②使用ConstantValue属性初始化
目前Sun javac编译器是这么做的【具体咋做不知道 = =】，如果同时使用final和static修饰一个变量[这种修饰就相当于个常量],并且是String或基本类型，就使用②，如果没有被final修饰或不是基本类型和String，就选择①在<clinit>方法中初始化
```

####[<clinit>和<init>是什么](https://blog.csdn.net/sujz12345/article/details/52590095)

<clinit> 类构造器

<init> 实例构造器

简单来说

 java在编译后会在字节码文件生成clinit方法，称为类构造器，会将静态变量的初始化收敛【放在】clinit方法里执行，<clinit> 方法在类加载过程中执行

 java在编译后会在字节码文件生成init方法，称为实例构造器



## Class文件整体结构

结合所学习的和个人理解画了class文件的大致结构图。

![各个字段.png](https://upload-images.jianshu.io/upload_images/10006199-f7de2ccc4abee98b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





# 类加载机制

### 1.加载

类加载第一步，

1.通过一个类的全限定名获取定义此类的二进制流

2.将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。

3.在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问接口。

```
这个的Class对象不一定从硬盘上的class文件获取，也有可能从【jar、war】中获取，也可以在程序运行时通过动态代理的方式生成，也可以从jsp文件转成的class文件中获取。

【谈一谈jsp:我们知道jsp中可以编写java代码，而且jsp请求时，会先转换成servlet【以前一直以为是项目启动的时候编译jsp，其实是访问jsp时，并且判断是否为第一次】，然后编译生成lass文件。jsp也是种servlet，也有jspInit()、jspDestroy()、jspService()等方法在不同阶段调用，也有自己的生命周期，这边参考 https://www.cnblogs.com/labing/p/5869745.html】
```

在win下用weblogic启动稍微测试了下

发现路径放在weblogic的domain下的模块区域，**由此推测jsp是web中间件帮忙编译的**。

![jsp](https://upload-images.jianshu.io/upload_images/10006199-0b7b94b566609e4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![weblogic中的jsp](https://upload-images.jianshu.io/upload_images/10006199-f69a04d200018985.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





### 2.验证

该环节主要用于确保加载进来的字节流是否符合jvm规范，不危害jvm安全。如果验证错误。将抛出诸如java.lang.VerifyError或其子类异常【常见于jar包冲突导致类加载时验证失败】。

```
java本身是安全的语言，因为java会拒绝编译一些"危险"的操作[比如使用纯粹的java代码无法做到访问数组边界外的数据，跳转到不存在的对象]，如果这么做编译器将拒绝编译或报相应的错，但是class文件不一定要由java源码过来，java代码无法做的事可以通过class文件来表达，虚拟机如果不验证检查输入的字节流，对其完全信任，很有可能会因为载入有害的字节流造成不良影响，所以验证也是保护虚拟机的一项重要工作。
```

根据java虚拟机规范，验证按顺序有4个阶段

|   验证类型   | 目的                                       | 说明【举例】                                   | 其它注意点                                    |
| :------: | :--------------------------------------- | :--------------------------------------- | ---------------------------------------- |
| 1.文件格式验证 | 确保输入的字节流可以正确解析并存储到方法区中，通过这个阶段验证后，**后续三个阶段都是基于方法区进行验证的**。 | （1）是否是class文件【魔数以 CAFEBABE开头】，试过javap执行非class文件，报了找不到类的错（2）版本号在当前jvm里是否支持，否则将报类似java.lang.UnsupportedClassVersionError: org/apache/lucene/store/Directory : Unsupported major.minor version 51.0的错 （3）常量池的常量中是否有不被支持的常量类型  (4) Class文件各个部分和本身是否有被删除或附加其它信息  [5]CONSTANT_Utf8_info型常量中是否有不符合UTF8编码的数据。。。 | 第一阶段远不止这些，文件验证主要目的还是保证输入的字节流能够正确解析存储在方法区内。 |
| 2.元数据验证  | 对类的元数据语义校验，保证符合Java语言规范要求                | （1）某个类是否有父类【除了Object，所有类都应当有父类(默认继承Object)】（2）某个类是否继承了不允许被继承的类【一般指被final修饰的类】（3）如果类是抽象类，是否实现它父类的方法或者它实现的接口的方法 | 验证java语言规范，保证不存在不符合java语言规范的元数据信息        |
| 3.字节码验证  | 验证程序语义是否符合逻辑，保证字节码流可以被jvm安全执行。           | （1）保证跳转指令不会跳到方法体以外的字节码指令上  （2）保证方法体中的类型转换有效，例如可以把子类对象类型赋给父类是安全的【向上转型，但是相反过来就不安全了，向上转型会使子类覆盖的方法缺失，但是它还是安全的，而向下转型则是不安全的，比如说人是动物，但动物不一定是人。】，甚至把对象赋值给跟它毫无关系的数据类型，这是危险的( 3 )保证任意时刻操作数栈的数据类型与指令代码都能配合工作，例如不会出现类似在操作栈放一个int的数据，使用时按long类型来加载到本地变量表中。 | 最为复杂的一个阶段，**如果一个类方法体的字节码没有通过字节码验证，那肯定是有问题的，如果通过了也不一定是安全的**。 |
|  符号引用验证  | 判断引用是否符合规定                               | （1）符号引用中通过字符串描述的全限定名是否能够找到对应的类。（2）在指定类中是否存在符合方法的字段描述符以及简单名称所描述的方法和字段。（3）符号引用中的类、字段、方法的访问性（private、protected、public、default）是否可被当前类访问。 | 符号引用主要是确保解析动作能正常执行，如果没通过符号验证将抛出IncompatibleClassChangeError的子类，类似NoSuchMethodError之类的错。 |

> 值得一提的是，虽然对于jvm的类加载机制来说，验证是非常重要【可以有效防止恶意代码攻击】但不是一定必要的阶段【对运行期不影响，而且从性能来讲，验证阶段的工作量在类加载的过程中，占比较大的工作量】,所以对于很有把握，反复验证过代码【包括自己写的和第三方包】,可以设置-Xverify:none参数来关闭类验证，缩短类加载时间。



### 3.准备

准备阶段主要作用

1.在方法区内为类变量【被static修饰的变量，不包括实例变量】分配内存

2.设置其初始值【通常情况下是数据类型的初始值，比如int是0，boolean是false，除去基本类型的reference为null】

这边刚开始还不大懂初始值设置是什么个情况，看了网上资料是这么说的



#### 准备阶段中静态变量[static]和静态常量[static final]有何区别

普通原始类型静态变量和引用类型（即使是常量），

```java
//比如说某个要加载的类有这么一段代码
public class ABC {
    public int i = 0; 
    public static int a = 1; //静态变量 变量a在准备阶段过后初始值是0而不是1，因为这时候还未执行java方法，把a赋值为1的putstatic指令是程序编译后，存放在类构造器<clinit>之中，所以赋值的动作在初始化阶段之后才进行
    public static final int b = 2; //静态常量 有final和static的情况，编译时javac会为b生成ConstantValue属性，在准备阶段就会根据这个属性赋值
    public static final Integer c = Integer.valueOf(3);//静态引用类型常量 无ConstantValue
}
```

```shell
#编译后查看jvm指令集 可以看到b变量有ConstantValue属性，并且对于变量b无赋值等指令，经javac编译成class文件的时候就已赋值
javap -p ABC.class
```

![](https://upload-images.jianshu.io/upload_images/10006199-a20805d1123eff2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

经过putstatic、invokestatic指令过后，才进行赋值，这个指令是在类加载初始化阶段执行，而不是准备阶段调用，而原始类型常量不需要此步骤。

总结

```
putstatic指令是程序被编译后，存放于类构造器<client>方法之中，在编译阶段生成ConstantValue属性，在准备阶段虚拟机会根据ConstantValue属性赋值。
```



### 4.解析

解析阶段是虚拟机常量池内的符号引用替换为直接引用的过程。

| 引用类型 | 描述                                       |
| ---- | ---------------------------------------- |
| 符号引用 | 一组来描述所引用的目标对象，这里的符号可以是各种形式的字面量或者说是字符串，用于无歧义的定位到目标。 |
| 直接引用 | 直接引用可以是(1)直接定位到目标的指针 ( 2 )偏移量【 指向实例变量，实例方法的直接引用是通过偏移量，通过这个偏移量虚拟机可以直接在该类的内存区域中找到方法字节码的起始位置】 ( 3 )可以直接定位到目标的句柄。 |

#### 符号引用和直接引用的区别

符号引用一般是具有某个含义的字面量或者说是字符串，符号引用的字面量形式明确定义在Java虚拟机规范的Class文件格式中，在编译期间并提前订好命名规范用于给jvm做识别。符号引用目的是确保解析动作能正常执行。如果没通过，将报IncompatibleClassChangeError子类的错，如java.lang.NoSuchFieldError、java.lang.NoSuchMethodError等错误。

直接引用是直接和内存做挂钩的，同一符号在不同系统、版本的jvm编译出来的直接引用不一定相同。

> 简单来说，就是符号引用和虚拟机内存布局无关，引用的目标不一定加载到内存中，而有了直接引用，那引用的目标必定已经被加载入内存中了。符号引用是字面量，会被jvm识别进而转为可以直接使用的"直接引用",比如说一个类【org.simple.People】要引用另外一个类【举个例子org.simple.Food， 实际是以二进制形式的完全限定名放在class文件中】,编译时并不知道实际地址，所以先用某个符号表示，接着再让jvm翻译成自己能直接引用的内存地址。



### 5.初始化

虚拟机规范严格规定有5中情况必须立即对类进行"初始化"，

初始化阶段是类加载最后一步，到了初始化阶段才正在开始执行java代码【或者说是字节码】，初始化阶段是执行性类构造器<clinit>()方法的过程。

关于<clinit>()

```
1.<clinit>()方法是由编译器自动收集类中的变量赋值动作和静态语句块合并产生的，编译器收集顺序是源文件出现顺序决定的，后面定义的变量，在前面静态块中可以赋值但不能访问。
2.<clinit>()方法和类的构造函数【或者说实例构造器<init>】不同，不需要显示调用父类构造器，虚拟机会自动保证子类的<clinit>()方法执行前先执行完父类的<clinit>()方法，所以虚拟机中第一个被执行<clinit>()方法方法的类肯定是Object。
3.由于父类的<clinit>()方法比子类先执行，这就意味着父类的static块执行顺序优于子类。
4.<clinit>()对于类和接口不是必须的，如果没有静态语句块，也没有对变量赋值操作，可以不生成这个方法。
5.接口不能使用静态语句块，但可以赋值，虽然不是必须的但接口也可以生成<clinit>()，但跟类不同，执行接口的<clinit>()不需要先执行父接口的<clinit>()方法，只有父接口定义的变量使用时才会初始化父接口。
6.<clinit>()方法在多线程中会被正确的做加锁同步操作，如果多线程去初始化一个类，只有一个类去执行这个<clinit>()，其它线程都需要等待知道执行<clinit>()完毕。
```

1听起来很绕，如图所示

![demo](https://upload-images.jianshu.io/upload_images/10006199-6a1e41030fd7f305.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



# 类加载器

> 虚拟机团队吧类加载阶段中"通过一个类的全限定名来获取类的二进制字节流"这个动作放在虚拟机外部实现。至于如果决定去获取需要的类【可能有和jdk自带类一样全限定名的类】，实现这部分的代码就是类加载器。

#### 三种类加载器【按顺序】

```
1. 启动类加载器 Bootstrap CLassloder 
2. 扩展类加载器 Extention ClassLoader 
3. 应用程序类加载器 AppClassLoader
```

关于类加载器网上文章很多，这篇写的不错，[一看你就懂，超详细java中的ClassLoader详解](https://blog.csdn.net/briblue/article/details/54973413)

对于java虚拟机，有两种不同类加载器，启动类加载器和其它加载器

启动类加载器在HotSpot虚拟机中用c++实现，是虚拟机的一部分 ,第一个类加载器是BootStrap,比较特殊，不需要被加载，而是嵌套在java虚拟机内核内，jvm启动时bootstarp已经启动，用c++写的二进制代码(非字节码)，它可以去加载别的类。

除了启动类其它类加载器全由java实现，全部继承自java.lang.ClassLoader,独立于虚拟机外部

启动类加载器：它的作用是将JAVA_HOME/lib目录下的类加载到内存中。需要注意的是**由于启动类加载器涉及到虚拟机本地的实现细节，开发人员将无法直接获取到启动类加载器的引用，所以不允许直接通过引用进行操作。**

标准扩展扩展类加载器：由Sun的ExtClassLoader实现的，它的作用是将JAVA_HOME/lib/ext目录下或由系统变量 java.ext.dir指定位置中的类加载到内存中，它可以由开发人员直接使用。

应用程序类加载器：它是由Sun的AppClassLoader实现的，它的作用是将classpath路径下指定的类加载到内存中。它也可以由开发人员使用。

自定义类加载器：自定义的类加载器继承自ClassLoader，并覆盖findClass方法，它的作用是将特殊用途的类加载到内存中。

```
一旦一个类被加载了，将不会再次被加载。
```

### 双亲委派机制

#### 工作流程

如果一个类加载器受到类加载的请求，它不会自己先去加载这个类，而是把请求委派给父加载器执去完成，依次往上递归直到传到最顶层，只有当最顶层的加载器反馈无法完成这个加载的请求，子加载器才会尝试自己去加载。

```
比如说java.lang.Object，它存在rt.jar中，无论哪一个类加载器要加载这个类最终都是委派给最顶端的启动类加载器进行加载。因此Object类在程序的各种类加载器环境下都是同一个类。
```



#### 作用

这个机制的作用是用于确定java虚拟机要加载一个类时用哪个类加载器加载，有个重要的作用是防止内存中出现多份同样的字节码。

比如说两个类分别为A类和B类，两个类都有用到System类，如果都自己加载这个类而不是委托上层类加载器加载，将导致内存中出现两份System字节码。如果使用委托，会递归到父类最后由**Bootstrap**加载，找不到的情况下才向下，如果Bootstrap发现已经加载过了就直接返回内存中的**System**而不需要重新加载。



### 一个类要用哪个类加载器去加载

如果类A引用了类B，虚拟机将使用加载A的类加载器加载B，如果这里的B类是rt.jar下的一些关键基础类【比如Object】，加载A的类加载器会向上委托去加载，这个类Bootstrap能处理，就交给它处理【假设处理不了Object才会向下请求让儿子加载。事实是Object就是由Bootstrap加载器加载】





**委托是从下向上，然后具体查找过程却是自上至下。**

![双亲委派机制.png](https://upload-images.jianshu.io/upload_images/10006199-dae1a63984c253b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





### 注意点

##### 父加载器不是父类

getParent()不在URLClassLoader而在ClassLoader中 返回的是ClassLoder(抽象类)对象 

boostrap classloader由c/c++编写 本身也是虚拟机的一部分 无法在java中获取它的引用

geParent()实际返回个ClassLoader对象的parent，parent的赋值在ClassLoader对象的构造方法中

ClassLoader由getSystemClassLoader()生成。



### 关于类加载器的疑问

##### 1.是否自己写个类叫`java.lang.System`

**答案：**通常不可以，但可以采取另类方法达到这个需求。 
**解释：**为了不让我们写System类，类加载采用委托机制，这样可以保证爸爸们优先，爸爸们能找到的类，儿子就没有机会加载。而System类是Bootstrap加载器加载的，就算自己重写，也总是使用Java系统提供的System，**自己写的System类根本没有机会得到加载。**

但是，我们可以**自己定义一个类加载器来达到这个目的**，为了避免双亲委托机制，这个类加载器也必须是特殊的。由于系统自带的三个类加载器都加载特定目录下的类，如果我们自己的类加载器放在一个特殊的目录，将会出现类正常编译但无法被加载运行【即使自定义的类加载器，也是会抛出java.lang.SecurityException异常】



##### 2.如何判断一个类是不是相同的类

初始java的时候，以为类名一样就是同样的类名就是相同的类【太年轻】，后来发现全限定名一样的类就是相同的类，并且一直都这么认为，实际上比较两个类是否“相等”，只有在两个类由同一个类加载器加载的前提下才有意义，否则即使两个类来源于同一个class文件，被同一个虚拟机加载，只要加载它们两的类加载器不同，就不是两个相同的类。





## 总结

class文件是java虚拟机执行引擎的数据入口，里面的结构顺序是固定的，各自有不同的含义，个人感觉就像是一辆车有多个部件组成，最后再由引擎(jvm)做驱动运行。

class文件是java技术体系基础构成之一，不依赖特定平台【只要对应平台有合适版本的jvm】，是java实现跨平台的重要支柱，了解class文件的结构以及类加载过程对进一步了解jvm有重要的意义，不然写了代码但不知道运行的过程，总感觉会缺了点什么，知其然而不知其所以然。

写博客主要也是以做笔记加深记忆为主，希望对这块内容有更深刻的理解，不断进步。由于本人水平不高，文章有不对的地方，请批评指正。



## 参考资料

<<深入理解jvm虚拟机>>

[从一个class文件深入理解Java字节码结构](https://www.jianshu.com/p/2c106b682cfb)

[JDK、JRE的关系](https://www.zhihu.com/question/20317448/answer/14735258)

[JVM-String常量池与运行时常量池](https://blog.csdn.net/Sugar_Rainbow/article/details/68150249)

[《Java虚拟机原理图解》 1.2.2、Class文件中的常量池详解（上)](https://blog.csdn.net/luanlouis/article/details/39960815?utm_source=blogxgwz4)

[JVM（三）：类加载机制（类加载过程和类加载器](https://blog.csdn.net/boyupeng/article/details/47951037)

