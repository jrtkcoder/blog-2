---
title: JVM知识汇总——内存管理与GC
date: 2017-03-09 13:04:51
categories: [Programming,Java]
tags: [Java,JVM]
---
![](http://ojoba1c98.bkt.clouddn.com/img/jvm/structure.png)

# 前言
> 想要深刻地理解Java，那么就要深入地理解底层——JVM(Java Virtual Machine | Java虚拟机)。
> JVM（Java Virtual Machine）Java 虚拟机是整个 java 平台的基石，是 java 系统实现硬件无关与操作系统无关的关键部分，是保障用户机器免于恶意代码损害的屏障。Java开发人员不需要了解JVM是如何工作的，但是，了解 JVM 有助于我们更好的开（通）发（过） java（公司） 程（面）序（试）
> 博主经过一番查阅，找到了自认为写的好的一些文章，并记录总结，方便不定时的看。
> 希望每次看都会有新的领悟，不断提高自己。

<!--more-->

# Java虚拟机架构

## 什么是JVM
要想说明白什么 JVM 就不得不提另外两个概念，JRE 和 JDK，初学者总是把这几个概念搞混。
![](http://ojoba1c98.bkt.clouddn.com/img/jvm/java-tutorial.png)
JVM，JRE，JDK 都是 Java 语言的支柱，他们分工协作。但不同的是 **JDK 和 JRE 是真实存在的**，而 JVM 是一个**抽象**的概念，并不真实存在。

### JDK
JDK(Java Development Kit) 是 Java 语言的软件开发工具包（SDK）。JDK 物理存在，是 programming tools、JRE 和 JVM 的一个集合。
![](http://ojoba1c98.bkt.clouddn.com/img/jvm/jdk.png)

### JRE
JRE（Java Runtime Environment）Java 运行时环境，JRE 物理存在，主要由Java API 和 JVM 组成，提供了用于执行 Java 应用程序最低要求的环境。
![](http://ojoba1c98.bkt.clouddn.com/img/jvm/jre.png)

### JVM（Java Virtual Machine）
JVM(Java Virtual Machine) 是一种软件实现，执行像物理机程序的机器（即电脑）。
本来，Java被设计基于从物理机器分离实现WORA（ 写一次，随处运行 ）的虚拟机上运行，虽然这个目标已经几乎被遗忘。
JVM 并不是专为 Java 所实现的运行时，实际上只要有其他编程语言的编译器能生成正确 Java bytecode 文件，则这个语言也能实现在JVM上运行。
因此，JVM 通过执行 Java bytecode 可以使 java 代码在不改变的情况下运行在各种硬件之上。

JVM实现了Java语言最重要的特征：即平台无关性。
**平台无关性原理**：编译后的 Java程序（`.class`文件）由**JVM执行**。JVM**屏蔽了与具体平台相关的信息**，使程序可以在多种平台上不加修改地运行。Java虚拟机在执行字节码时，把字节码解释成具体平台上的机器指令执行。因此实现**Java平台无关性**。
## JVM结构图
![](http://ojoba1c98.bkt.clouddn.com/img/jvm/jvm-frame-diagram.png)
**JVM = 类加载器 classloader+ 执行引擎 executionengine + 运行时数据区域 runtime data area**
首先Java源代码文件被Java编译器编译为字节码文件，然后JVM中的**类加载器**加载完毕之后，交由JVM**执行引擎**执行。在整个程序执行过程中，JVM中的**运行时数据区（内存）**会用来存储程序执行期间需要用到的数据和相关信息。
因此，**在Java中我们常常说到的内存管理就是针对这段空间进行管理**（如何分配和回收内存空间）。

## ClassLoader
ClassLoader把硬盘上的class文件**加载到JVM中的运行时数据区域**，但是它不负责这个类文件能否执行，而这个是执行引擎负责的。
限于篇幅，**类加载器的组织结构，加载类的机制原理**等会在***[JVM——类加载器总结](#Java%E7%B1%BB%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6%E6%80%BB%E7%BB%93)***一文中描述。
**双亲委派模型**以及**自定义类加载器**会在***[JVM——自定义类加载器](#%E8%87%AA%E5%AE%9A%E4%B9%89%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8)***一文中描述。

## 执行引擎
作用：**执行字节码，或者执行本地方法。**

## Runtime DataArea
JVM运行时数据区 (JVM RuntimeArea)其实就是指 JVM在**运行期间**，其**对JVM内存空间的划分和分配**。JVM在运行时将数据划分为了**以下几个区域来存储**。
程序员写的所有程序都被加载到运行时数据区域中。
![](http://ojoba1c98.bkt.clouddn.com/img/jvm/jvm-runtime-area.png)
（图注：**JDK1.7已经把常量池转移到堆里面了！**）
### PC寄存器（The pc Register）
（1）每一个Java线程**都有一个PC寄存器**，用以**记录当前执行到哪个指令**。
（2）用于存储每个线程下一步将执行的JVM指令，如该方法是Java方法，则记录的是**正在执行的虚拟机字节码地址**，如该方法为native的，则**计数器值为空**。
（3）此内存区域是**唯一一个在JVM中没有规定任何OutOfMemoryError情况**的区域。

### JVM栈（Java Virtual Machine Stacks）
（1）JVM栈是**线程私有的，并且生命周期与线程相同**。并且当线程运行完毕后，相应内存也就被**自动回收**。
（2）栈里面存放的元素叫**栈帧**，每个函数从调用到执行结束，其实是对应一个栈帧的**入栈和出栈**。
（栈帧好像很复杂的样子，其实它很简单！）它里面具体存放的是执行的函数的一些数据，如**局部变量、操作数**栈（执行引擎计算时需要），**方法出口**等等。
（3）这个区域可能有两种异常：如果线程请求的栈深度大于虚拟机所允许的深度，将抛出`StackOverflowError`异常（如：将一个函数反复递归自己，最终会出现这种异常）。如果JVM栈可以动态扩展（大部分JVM是可以的），当扩展时无法申请到足够内存则抛出`OutOfMemoryError`异常。

下面来实现一个栈溢出：
```java
package org.antd.test;

public class StackOverFlowMock {
	int num;
	public int getNum() {
		return num;
	}
	public void stackOver(){
		num++;
		stackOver();
	}
	public static void main(String[] args) {
		StackOverFlowMock stackOverFlowTest = new StackOverFlowMock();
		try {
			stackOverFlowTest.stackOver();
		} catch (Throwable t) {
			System.out.println("迭代深度："+stackOverFlowTest.getNum());
			t.printStackTrace();
		}
	}

}

```
输出为：
```java
迭代深度：17781
java.lang.StackOverflowError
	at org.antd.test.StackOverFlowMock.stackOver(StackOverFlowMock.java:10)
	at org.antd.test.StackOverFlowMock.stackOver(StackOverFlowMock.java:10)
	at org.antd.test.StackOverFlowMock.stackOver(StackOverFlowMock.java:10)
	at org.antd.test.StackOverFlowMock.stackOver(StackOverFlowMock.java:10)
	at org.antd.test.StackOverFlowMock.stackOver(StackOverFlowMock.java:10)
	at org.antd.test.StackOverFlowMock.stackOver(StackOverFlowMock.java:10)

```

### 本地方法栈（Native Method Stacks）

（1）本地方法栈与虚拟机栈所发挥的作用很相似，他们的区别在于**虚拟机栈为执行Java代码方法服务，而本地方法栈是为Native方法服务**。
（2）和JVM栈一样，这个区域也会抛出`StackOverflowError`和`OutOfMemoryError`异常。

### 方法区（Method Area）
（1）在方法区中，存储了每个**类的信息**、**静态变量**等。如，当程序中通过`getName`、`isInterface`等方法来获取信息时，这些数据来源于**方法区**。
（2）方法区域是**全局共享**的，比如**每个线程都可以访问同一个类的静态变量**。
（3）由于使用**反射机制**的原因，虚拟机**很难推测哪个类信息不再使用**，因此这块区域的**回收很难**！另外，对这块区域主要是针对**常量池**回收，**值得注意的是JDK1.7已经把常量池转移到堆里面了**。
（4）同样，当方法区**无法满足内存分配需求**时，会抛出`OutOfMemoryError`。下面演示一下造成方法区内的OOM场景。
执行之前，可以把虚拟机的参数`-XXpermSize`和`-XX：MaxPermSize`限制方法区大小。
那么实现一下OOM：
```java
public class HeapOomMock {
	public static void main(String[] args) {
		List<byte[]> list = new ArrayList<byte[]>();
		int i = 0;
		boolean flag = true;
		while (flag) {
			try {
				i++;
				list.add(new byte[1024 * 1024]);// 每次增加一个1M大小的数组对象
			} catch (Throwable e) {
				flag = false;
				System.out.println("count=" + i);// 记录运行的次数
				e.printStackTrace();
			}
		}
	}
}

```
控制台输出：
```
count=3422
java.lang.OutOfMemoryError: Java heap space
	at org.antd.test.HeapOomMock.main(HeapOomMock.java:14)
```

### 运行时常量池（Runtime Constant Pool）
（1）存放类中固定的常量信息、方法引用信息等，其空间从方法区域（JDK1.7后为堆空间）中分配。
（2）Class文件中除了有类的版本、字段、方法、接口等描述等信息外，还有就是**常量表**(constant_pool table)，用于存放**编译期已可知的常量**，这部分内容将在**类加载后进入方法区（永久代）存放**。但是Java语言并不要求常量一定只有编译期预置入Class的常量表的内容才能进入方法区常量池，运行期间也可将新内容放入常量池（最典型的`String.intern()`方法）。
（3）当常量池无法在申请到内存时会抛出`OutOfMemoryError`异常。
再来一段代码展示以下：
```
//不断将字符串添加到常量池，最终导致内存不足抛出方法区的OOM 
List<String> list =new ArrayList<String>(); 
int i =0; 
while(true){ 
    list.add(String.valueOf(i).intern()); 
} 
```
String的intern函数的作用就不多赘述了，在这篇博文***[了解String类的intern()方法](#%E9%80%9A%E8%BF%87String%E7%B1%BB%E7%9A%84intern-%E4%BA%86%E8%A7%A3%E5%86%85%E5%AD%98%E5%BB%BA%E6%A8%A1%E4%B8%8E%E5%B8%B8%E9%87%8F%E6%B1%A0)***有所介绍。关于JDK1.6和JDK1.7之后常量池位置的变化对该函数的影响，也在链接文中阐述了。

### Java堆
（1）Java堆是JVM所管理的**最大的一块内存**。它是被**所有线程共享的一块内存区域，在虚拟机启动时创建**。
（2）几乎所有的**实例对象都是在这块区域中存放**。（JIT编译器貌似不是这样的）。
（3）Java堆是**垃圾搜集管理**的主要战场。所有Java堆可以细分为：**新生代**和**老年代**。再细致分就是把新生代分为：**Eden空间**、**FromSurvivor空间**、**To Survivor空间**。JVM具体的垃圾回收机制总结请查看***[JVM——内存管理和垃圾回收](#%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86%E5%92%8C%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6)***。
（4）根据**Java虚拟机规范**的规定，Java堆可以处于物理上**不连续的内存空间**中，只要**逻辑上是连续**的即可，就像我们的磁盘空间一样。如果在**堆中没有内存完成实例分配**，并且**堆也无法再扩展时**，将会抛出`OutOfMemoryError`异常。


## 堆和栈的区别
这是一个非常常见的面试题，主要从以下几个方面来回答。
### 各司其职
最主要的区别就是**栈内存用来存储局部变量和方法调用**。
而堆内存用来存储**Java中的对象**。无论是成员变量、局部变量还是类变量，它们指向的对象都存储在堆内存中。
### 空间大小
**栈的内存要远远小于堆内存**，如果你使用**递归**的话，那么你的栈很快就会充满并产生``StackOverFlowError``。
关于如何设置堆栈内存的大小，可以查看***[JVM——内存管理和垃圾回收](#%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86%E5%92%8C%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6)***中的相关介绍。
### 独有还是共享
栈内存归属于**线程的私有内存**，每个线程都会有一个栈内存，其存储的变量只能在其所属线程中可见。
而堆内存中的对象**对所有线程可见**，可以被所有线程访问。
### 异常错误
如果线程请求的**栈深度**大于虚拟机所允许的深度，将抛出`StackOverflowError`异常。
如果JVM栈可以动态扩展（大部分JVM是可以的），当扩展时**无法申请到足够内存**则抛出`OutOfMemoryError`异常。
而堆内存没有可用的空间存储生成的对象，JVM会抛出`java.lang.OutOfMemoryError`。

以上便是关于JVM架构的相关知识。

# 通过String类的intern()了解内存建模与常量池
## 引言
什么都先不说，先看下面这个引入的例子：
```java
String str1 = new String("SEU")+ new String("Calvin"); 
System.out.println(str1.intern() == str1);
System.out.println(str1 == "SEUCalvin"); 
```
JDK版本1.8，输出结果为：
```
true
true
```
再将上面的例子加上一行代码：
```java
String str2 = "SEUCalvin";//新加的一行代码，其余不变 
String str1 = new String("SEU")+ new String("Calvin"); 
System.out.println(str1.intern() == str1);
System.out.println(str1 == "SEUCalvin");
```
再运行，结果为：
```
false 
false 
```
是不是感觉莫名其妙，新定义的`str2`好像和`str1`没有半毛钱的关系，怎么会影响到有关`str1`的输出结果呢？其实这都是`intern()`方法搞的鬼！看完这篇文章，你就会明白。
在JVM架构一文中也有介绍，在JVM**运行时数据区中的方法区有一个常量池**，但是发现在**JDK1.6以后常量池被放置在了堆空间**，因此常量池位置的不同影响到了**String的intern()方法**的表现。深入了解后发现还是值得写下来记录一下的。为了确保文章的实时更新，实时修改可能出错的地方，请确保这篇是原文，而不是无脑转载来的“原创文”，原文链接为：***[http://blog.csdn.net/seu_calvin/article/details/52291082](http://blog.csdn.net/seu_calvin/article/details/52291082)***。

## 为什么要介绍intern()方法
`intern()`方法设计的初衷，就是**重用String对象**，以**节省内存消耗**。这么说可能有点抽象，那么就用例子来证明。
```java
    static final int MAX = 100000; 
    static final String[] arr = new String[MAX]; 
      
    public static void main(String[] args) throws Exception { 
        //为长度为10的Integer数组随机赋值  
        Integer[] sample = new Integer[10];  
        Random random = new Random(1000);  
        for (int i = 0; i < sample.length; i++) {  
            sample[i] = random.nextInt();  
        }  
        //记录程序开始时间  
        long t = System.currentTimeMillis();  
        //使用/不使用intern方法为10万个String赋值，值来自于Integer数组的10个数  
            for (int i = 0; i < MAX; i++) {  
                arr[i] = new String(String.valueOf(sample[i % sample.length]));  
                //arr[i] = new String(String.valueOf(sample[i % sample.length])).intern();  
            }  
            System.out.println((System.currentTimeMillis() - t) + "ms");  
            System.gc();  
    } 
```
这个例子也比较简单，**就是为了证明使用`intern()`比不使用`intern()`消耗的内存更少**。
先定义一个长度为10的Integer数组，并随机为其赋值，在通过for循环为长度为10万的String对象依次赋值，这些值都来自于Integer数组。两种情况分别运行，可通过`Window`>`Preferences`>`Java`>`InstalledJREs`设置JVM启动参数为`-agentlib:hprof=heap=dump,format=b`，将程序运行完后的hprof置于工程目录下。再通过**MAT插件**查看该`hprof`文件。
两次实验结果如下：
![](http://ojoba1c98.bkt.clouddn.com/img/jvm/hprof1.png)
![](http://ojoba1c98.bkt.clouddn.com/img/jvm/hrpof2.png)
从运行结果来看，不使用`intern()`的情况下，程序生成了101762个String对象，而使用了`intern()`方法时，程序仅生成了1772个String对象。自然也证明了**`intern()`节省内存的结论**。
细心的同学会发现使用了`intern()`方法后**程序运行时间有所增加**。这是因为程序中每次都是用了`new String`后又进行**`intern()`操作的耗时时间**，但是不使用`intern()`占用内存空间导致GC的时间是要远远大于这点时间的。 

## 深入认识intern()方法
**JDK1.7后**，**常量池被放入到堆空间中**，这导致`intern()`函数的功能不同，具体怎么个不同法，且看看下面代码，这个例子是网上流传较广的一个例子，分析图也是直接粘贴过来的，这里自己的理解去解释这个例子：
```java
String s = new String("1"); 
s.intern();
String s2 = "1"; 
System.out.println(s == s2); 
  
String s3 = new String("1") + new String("1"); 
s3.intern(); 
String s4 = "11"; 
System.out.println(s3 == s4); 
```
输出结果为：
```
JDK1.6以及以下：false false 
JDK1.7以及以上：false true
```
再分别调整上面代码2.3行、7.8行的顺序：
```java
String s = new String("1"); 
String s2 = "1"; 
s.intern(); 
System.out.println(s == s2); 
  
String s3 = new String("1") + new String("1"); 
String s4 = "11"; 
s3.intern(); 
System.out.println(s3 == s4); 
```
输出结果为：
```
JDK1.6以及以下：false false 
JDK1.7以及以上：false false 
```
下面依据上面代码对`intern()`方法进行分析
### JDK1.6
![](http://ojoba1c98.bkt.clouddn.com/img/jvm/jdk6-intern.png)
在JDK1.6中所有的输出结果都是 false，因为JDK1.6以及以前版本中，常量池是放在 Perm 区（属于方法区）中的，熟悉JVM的话应该知道这是和堆区完全分开的。
使用**引号声明的字符串都是会直接在字符串常量池**中生成的，而 **new 出来的 String 对象是放在堆空间中的**。所以两者的内存地址肯定是不相同的，即使调用了`intern()`方法也是不影响的。如果不清楚String类的`“==”`和`equals()`的区别可以查看这篇博文***[Java面试——从Java堆、栈角度比较equals和==的区别](http://blog.csdn.net/seu_calvin/article/details/52089040)***。
`intern()`方法在JDK1.6中的**作用**是：比如`String s=new String("SEU_Calvin")`，再调用`s.intern()`，此时返回值还是字符串`"SEU_Calvin"`，表面上看起来好像这个方法没什么用处。但实际上，在JDK1.6中它做了个小动作：检查字符串池里**是否存在`"SEU_Calvin"`这么一个字符串**，如果存在，就返回池里的字符串；如果不存在，该方法会把`"SEU_Calvin"`添加到字符串池中，然后再返回它的引用。然而在JDK1.7中却不是这样的，后面会讨论。

## JDK1.7
针对JDK1.7以及以上的版本，我们将上面两段代码分开讨论。先看第一段代码的情况：
![](http://ojoba1c98.bkt.clouddn.com/img/jvm/jdk7-intern1.png)
```java
String s = new String("1"); 
s.intern(); 
String s2 = "1"; 
System.out.println(s == s2); 
  
String s3 = new String("1") + new String("1"); 
s3.intern(); 
String s4 = "11"; 
System.out.println(s3 == s4); 
```
`String s = newString("1")`，生成了**常量池中的“1” 和堆空间中的字符串对象**。
`s.intern()`，这一行的作用是s对象去常量池中寻找后发现"1"**已经存在于常量池中了**。
`String s2 = "1"`，这行代码是生成一个s2的引用**指向常量池中的“1”对象**。
结果就是 s 和 s2 的**引用地址**明显不同。因此返回了`false`。

`String s3 = new String("1") + newString("1")`，这行代码在**字符串常量池中生成“1”** ，**并在堆空间中生成s3引用指向的对象（内容为"11"）**。注意此时常量池中是没有 “11”对象的。
`s3.intern()`，这一行代码，是将 s3中的**“11”字符串放入 String 常量池**中，此时常量池中不存在“11”字符串，JDK1.6的做法是直接在常量池中生成一个 "11" 的对象。
**但是在JDK1.7中，常量池中不需要再存储一份对象了，可以直接存储堆中的引用**。这份引用直接**指向 s3 引用的对象，也就是说`s3.intern() ==s3`会返回`true`**。
`String s4 = "11"`， 这一行代码会**直接去常量池中创建**，但是发现已经有这个对象了，此时也就是指向 **s3 引用对象的一个引用**。因此`s3 == s4`返回了`true`。

下面继续分析第二段代码：
![](http://ojoba1c98.bkt.clouddn.com/img/jvm/jdk7-intern2.png)
再把第二段代码贴一下便于查看：
```java
String s = new String("1"); 
String s2 = "1"; 
s.intern(); 
System.out.println(s == s2); 

String s3 = new String("1") + new String("1"); 
String s4 = "11"; 
s3.intern(); 
System.out.println(s3 == s4); 
```
`String s = newString("1")`，生成了**常量池中的“1” 和堆空间中的字符串对象**。
`String s2 = "1"`，这行代码是生成一个s2的引用**指向常量池中的“1”对象，但是发现已经存在了，那么就直接指向了它**。
`s.intern()`，这一行在这里就没什么实际作用了。因为"1"已经存在了。
结果就是 s 和 s2 的引用地址明显不同。因此返回了`false`。

`String s3 = new String("1") + newString("1")`，这行代码在**字符串常量池中生成“1” ，并在堆空间中生成s3引用指向的对象（内容为"11"）**。注意此时常量池中是没有 “11”对象的。
`String s4 = "11"`， 这一行代码会**直接去生成常量池中的"11"**。
`s3.intern()`，这一行在这里就没什么实际作用了。因为"11"已经存在了。
结果就是 s3 和 s4 的引用地址明显不同。因此返回了`false`。
为了确保文章的实时更新，实时修改可能出错的地方，请确保这篇是原文，而不是无脑转载来的“原创文”，原文链接为：***[http://blog.csdn.net/seu_calvin/article/details/52291082](http://blog.csdn.net/seu_calvin/article/details/52291082)***。

## 总结
终于要做Ending了。现在再来看一下开篇给的引入例子，是不是就很清晰了呢。
```java
String str1 = new String("SEU") + new String("Calvin"); 
System.out.println(str1.intern() == str1); 
System.out.println(str1 == "SEUCalvin");
```
`str1.intern()==str1`就是上面例子中的情况，`str1.intern()`发现常量池中不存在`“SEUCalvin”`，因此指向了str1。`"SEUCalvin"`在常量池中创建时，也就直接指向了str1了。两个都返回`true`就理所当然啦。
那么第二段代码呢：
```java
String str2 = "SEUCalvin";//新加的一行代码，其余不变 
String str1 = new String("SEU")+ new String("Calvin"); 
System.out.println(str1.intern() == str1); 
System.out.println(str1 == "SEUCalvin"); 
```
也很简单啦，str2先在常量池中创建了`“SEUCalvin”`，那么`str1.intern()`当然就直接指向了str2，你可以去验证它们两个是返回的true。后面的`"SEUCalvin"`也一样指向str2。所以谁都不搭理在堆空间中的str1了，所以都返回了`false`。
好了，本篇对intern的作用以及在JDK1.6和1.7中的实现原理的介绍就到此为止了。希望能给你带来帮助。
# 内存管理和垃圾回收
## 何为GC
Java与C语言相比的一个优势是，可以通过自己的JVM**自动分配和回收内存空间**。
垃圾回收机制是由垃圾搜集器Garbage Collection来实现的，GC是后台一个**低优先级的守护进程**。在内存中低到一定限度时才会自动运行，因此垃圾**回收的时间是不确定的**。

**为何要这样设计**：因为GC也要消耗CPU等资源，如果GC执行过于频繁会对Java的程序的执行产生较大的影响，因此实行**不定期的GC**。

**与GC有关的是**：JVM运行时数据区中的堆（对象实例会存储在这里）和 gabagecollector方法。
垃圾回收GC**只能回收通过new关键字申请的内存**（在堆上），但是堆上的内存并不完全是通过new申请分配的。还有一些本地方法，这些内存如果不手动释放，就会导致内存泄露，所以需要在finalize中用本地方法(nativemethod)如**free操作等，再使用gc**方法。
```java
System.gc();
```

## 何为垃圾
Java中那些**不可达**的对象就会变成垃圾。对象之间的引用可以抽象成树形结构，通过树根（GC Roots）作为起点，从这些树根往下搜索，搜索走过的链称为引用链。
**当一个对象到GC Roots没有任何引用链相连时，则证明这个对象为可回收的对象**。
**可以作为GC Roots的主要有以下几种**：
（1）**栈帧中的本地变量表**所引用的对象。
（2）方法区中**类静态属性和常量**引用的对象。 
（3）本地方法栈中JNI（**Native方法**）引用的对象。
```java
//垃圾产生的情况举例： 
//1.改变对象的引用，如置为null或者指向其他对象 
Object obj1 = new Object(); 
Object obj2 = new Object(); 
obj1 = obj2; //obj1成为垃圾 
obj1 = obj2 = null ; //obj2成为垃圾
```
```java
//2.引用类型 
//第2句在内存不足的情况下会将String对象判定为可回收对象，第3句无论什么情况下String对象都会被判定为可回收对象 
String str = new String("hello"); 
SoftReference<String> sr = new SoftReference<String>(new String("java")); 
WeakReference<String> wr = new WeakReference<String>(new String("world"));
```
```java
//3.循环每执行完一次，生成的Object对象都会成为可回收的对象 
 for(int i=0;i<10;i++) { 
        Object obj = new Object();  
        System.out.println(obj.getClass());  
    } 
```
```java
//4.类嵌套 
class A{ 
   A a; 
} 
A x = new A();//分配了一个空间 
x.a = new A();//又分配了一个空间 
x = null;//产生两个垃圾
```
```java
    //5.线程中的垃圾 
    calss A implements Runnable{ 
        void run(){  
        //....  
    } 
    }
      
    //main 
    A x = new A(); 
    x.start(); 
    x=null; //线程执行完成后x对象才被认定为垃圾 
```

##  四种引用类型
### 强引用
```java
Object obj = new Object();
```
这里的obj引用便是一个强引用，强引用不会被GC回收。即使抛出`OutOfMemoryError`错误，使程序异常终止。

### 软引用
如果内存空间足够，垃圾回收器就不会回收它，如果内存空间不足了，就会回收这些对象的内存。软引用可用来实现内存敏感的高速缓存。
软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。
 
### 弱引用
弱引用与软引用的区别在于：垃圾回收器一旦发现了弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些弱引用的对象。
弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中。

### 虚引用
虚引用与软引用和弱引用的一个区别在于：虚引用必须和引用队列（ReferenceQueue）联合使用。当垃圾回收器发现一个对象有虚引用时，就会把这个虚引用对象加入到与之关联的引用队列中。此时该对象并没有被GC回收。而是要等到引用队列被真正的处理后才会被回收。
程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。
（由于`Object.finalize()`方法的不安全性、低效性，常常使用虚引用完成对象回收前的资源释放工作。）
这里特别需要注意：当JVM将虚引用插入到引用队列的时候，虚引用执行的对象内存还是存在的。但是PhantomReference并没有暴露API返回对象。所以如果想做清理工作，需要继承PhantomReference类，以便访问它指向的对象。如NIO直接内存的自动回收，就使用到了sun.misc.Cleaner。

## 典型的垃圾回收算法

在确定了哪些垃圾可以被回收后，垃圾搜集器要做的事情就是开始进行垃圾回收，但是这里面涉及到一个问题是：**如何高效地进行垃圾回收**。
下面讨论几种常见的**垃圾搜集算法**。

### Mark-Sweep（标记-清除）算法
标记-清除算法分为两个阶段：标记阶段和清除阶段。
**标记阶段**的任务是标记出所有需要被回收的对象，**清除阶段**就是回收被标记的对象所占用的空间。
标记-清除算法实现起来比较容易，但是有一个比较严重的问题就是**容易产生内存碎片**，碎片太多可能会导致后续过程中需要为大对象分配空间时无法找到足够的空间而提前触发GC。

### Copying（复制）算法
Copying算法将可用内存**按容量划分为大小相等的两块**，**每次只使用其中的一块**。当这一块的内存用完了，就**将还存活着的对象复制到另外一块上面**，然后再把第一块内存上的空间一次清理掉，这样就**不容易出现内存碎片**的问题，并且**运行高效**。
但是该算法导致**能够使用的内存缩减到原来的一半**。而且，该算法的**效率跟存活对象的数目多少有很大的关系**，如果存活对象**很多**，那么Copying算法的**效率将会大大降低**。（这也是为什么后面提到的新生代采用Copying算法）

### Mark-Compact（标记-整理）算法
为了解决Copying算法的缺陷，充分利用内存空间，提出了Mark-Compact算法。
该算法标**记阶段标**记出所有需要被回收的对象，但是在完成标记之后不是直接清理可回收对象，而是**将存活的对象都移向一端**，然后**清理掉端边界以外的所有内存**（**只留下存活对象**）。

### 以上三种算法对比
它们的共同点主要有以下两点：
1、**三个算法都基于根搜索算法去判断一个对象是否应该被回收，而支撑根搜索算法可以正常工作的理论依据，就是语法中变量作用域的相关内容。因此，要想防止内存泄露，最根本的办法就是掌握好变量作用域，而不应该使用前面内存管理杂谈一章中所提到的C/C++式内存管理方式。**
2、**在GC线程开启时，或者说GC过程开始时，它们都要暂停应用程序（stop the world）。**
它们的区别按照下面几点来给各位展示。（>表示前者要优于后者，=表示两者效果一样）
**效率：复制算法>标记-整理算法>标记-清除算法（此处的效率只是简单的对比时间复杂度，实际情况不一定如此）。**
**内存整齐度：复制算法=标记-整理算法>标记-清除算法。**
**内存利用率：标记-整理算法=标记-清除算法>复制算法。**
可以看到标记-清除算法是比较落后的算法了，但是后两种算法却是在此基础上建立的，俗话说“吃水不忘挖井人”，因此各位也莫要忘记了标记-清除这一算法前辈。而且，在某些时候，标记-清除也会有用武之地。

到此我们已经将三个算法了解清楚了，可以看出，效率上来说，复制算法是当之无愧的老大，但是却浪费了太多内存，而为了尽量兼顾上面所提到的三个指标，标记-整理算法相对来说更平滑一些，但效率上依然不尽如人意，它比复制算法多了一个标记的阶段，又比标记-清除多了一个整理内存的过程。
难道就没有一种最优算法吗？
当然是没有的，这个世界是公平的，任何东西都有两面性，试想一下，你怎么可能找到一个又漂亮又勤快又有钱又通情达理，性格又合适，家境也合适，身高长相等等等等都合适的女人？就算你找到了，至少有一点这个女人也肯定不满足，那就是她不会爱上你。你是不是想说你比博主强太多了，那博主只想对你说，高富帅是不会爬在电脑前看技术文章的，0.0。
但是古人就是给力，古人说了，找媳妇不一定要找最好的，而是要找最合适的，听完这句话，瞬间感觉世界美好了许多。
算法也是一样的，**没有最好的算法，只有最合适的算法**。 
既然这三种算法都各有缺陷，高人们自然不会容许这种情况发生。因此，高人们提出可以根据对象的不同特性，使用不同的算法处理，类似于萝卜白菜各有所爱的原理。于是奇迹发生了，高人们终于找到了GC算法中的神级算法-----**分代搜集算法**。

### Generational Collection（分代搜集）算法
分代搜集算法是针对对象的不同特性，而使用适合的算法，这里面并没有实际上的新算法产生。**与其说分代搜集算法是第四个算法，不如说它是对前三个算法的实际应用**。
**分代搜集算法**是目前大部分JVM的垃圾搜集器采用的算法。
它的核心思想是**将堆区划分为老年代（Tenured Generation）和新生代（Young Generation）**，老年代的特点是每次垃圾搜集时只有**少量**对象需要被回收，而新生代的特点是每次垃圾回收时都有**大量**的对象需要被回收，那么就可以在**不同代的采取不同的最适合的搜集算法**。

目前大部分垃圾搜集器对于**新生代都采取Copying算法**，因为新生代中每次垃圾回收都要**回收大部分对象**，也就是说需要**复制的操作次数较少**，该算法效率在新生代也较高。但是实际中并不是按照1：1的比例来划分新生代的空间的，一般来说是将新生代划分为**一块较大的Eden空间**和**两块较小的Survivor空间**，**每次使用Eden空间和其中的一块Survivor空间**，当进行回收时，将还存活的对象复制到另一块Survivor空间中，然后清理掉Eden和A空间。在进行了第一次GC之后，使用的便是Eden space和**B空间**了，下次GC时会将存活对象复制到**A空间**，如此**反复循环**。

当对象在**Survivor区**躲过一次GC的话，其**对象年龄便会加1**，默认情况下，**对象年龄达到15时**，就会**移动到老年代中**。一般来说，大对象会被直接分配到老年代，所谓的大对象是指需要大量连续存储空间的对象，最常见的一种大对象就是大数组，比如：`byte[]` `data` `=` `new` `byte[4*1024*1024]`。
当然分配的规则并不是百分之百固定的，这要取决于当前使用的是哪种垃圾搜集器组合和JVM的相关参数。这些**搬运工作都是GC完成的**，GC不仅负责在Heap中搬运实例，同时负责回收存储空间。
最后，因为每次回收都只回收少量对象，所以**老年代一般使用的是标记整理算法**。

**注意**，在方法区中有一个**永久代**（Permanet Generation），它用来存储class类、常量、方法描述等。对永久代的回收主要回收两部分内容：**废弃常量**和**无用的类**。
![](http://ojoba1c98.bkt.clouddn.com/img/jvm/java-heap-memory.png)
有关查看垃圾回收信息的JVM常见配置方式：
```
-XX:+PrintGCDetails
```
最后介绍一下有关堆的JVM常见配置方式：
```
-Xss //选置栈内存的大小  
-Xms: //初始堆大小  
-Xmx: //最大堆大小  
-XX:NewSize=n: //设置年轻代大小  
-XX:NewRatio=n: //设置年轻代和年老代的比值。比如设置为3，表示年轻代与年老代比值为1：3  
-XX:SurvivorRatio=n: //年轻代中Eden区与两个Survivor区的比值。注意Survivor区有两个。比如设置为3，表示Eden：Survivor=3：2，一个Survivor区占整个年轻代的1/5。  
-XX:MaxPermSize=n: //设置持久代大小  
```

## 典型的垃圾回收器
垃圾搜集算法是内存回收的理论基础，而垃圾搜集器就是内存回收的具体实现。
下面介绍一下**HotSpot**（JDK 7)虚拟机提供的几种垃圾搜集器，用户可以根据自己的需求组合出各个年代使用的搜集器。

### Serial&Serial Old
`Serial`和`Serial Old`搜集器是最基本最古老的搜集器，是一个**单线程**搜集器，并且在它进行垃圾搜集时，必须**暂停所有用户线程**。`Serial`搜集器是**针对新生代**的搜集器，采用的是**Copying算法**，`Serial Old`搜集器是针对**老年代的搜集器**，采用的是**Mark-Compact算法**。它的优点是实现简单高效，但是缺点是会给用户带来停顿。

### ParNew
`ParNew`搜集器是`Serial`搜集器的**多线程版本**，使用多个线程进行垃圾搜集。

### Parallel Scavenge
`Parallel Scavenge`搜集器是一个**新生代的多线程搜集器（并行搜集器）**，它在回收期间**不需要暂停其他用户线程**，其采用的是**Copying算法**，该搜集器与前两个搜集器有所不同，它主要是为了达到一个**可控的吞吐量**。

### Parallel Old
`Parallel Old`是`Parallel Scavenge`搜集器的**老年代版本（并行搜集器）**，使用多线程和**Mark-Compact算法**。

### CMS
`CMS`（Current Mark Sweep）搜集器是一种以**获取最短回收停顿时间**为目标的搜集器，它是一种**并发**搜集器，采用的是**Mark-Sweep算法**。

### G1
G1搜集器是当今搜集器技术发展最前沿的成果，它是一款**面向服务端应用**的搜集器，它能充分利用**多CPU**、**多核**环境。因此它是一款**并行与并发**搜集器，并且它能建立**可预测的停顿时间**模型。
最后介绍一下有关搜集器设置的JVM常见配置方式：
```
-XX:+UseSerialGC: //设置串行搜集器  
-XX:+UseParallelGC: //设置并行搜集器  
-XX:+UseParalledlOldGC: //设置并行年老代搜集器  
-XX:+UseConcMarkSweepGC: //设置并发搜集器  
//并行搜集器设置  
-XX:ParallelGCThreads=n: //设置并行搜集器搜集时使用的CPU数，并行搜集线程数  
-XX:MaxGCPauseMillis=n: //设置并行搜集最大暂停时间  
-XX:GCTimeRatio=n: //设置垃圾回收时间占程序运行时间的百分比，公式为1/(1+n)  
//并发搜集器设置  
-XX:+CMSIncrementalMode: //设置为增量模式。适用于单CPU情况  
-XX:ParallelGCThreads=n: //设置并发搜集器年轻代搜集方式为并行搜集时，使用的CPU数。并行搜集线程数 
```

# 最后
个人觉得以上原文写的很好，起码自己看得懂。
下面附上参考

> **参考并转载于：**
> ***[http://blog.csdn.net/seu_calvin/article/details/51404589](http://blog.csdn.net/seu_calvin/article/details/51404589)***
> ***[http://blog.csdn.net/seu_calvin/article/details/51892567](http://blog.csdn.net/seu_calvin/article/details/51892567)***
> ***[http://blog.csdn.net/seu_calvin/article/details/52301541](http://blog.csdn.net/seu_calvin/article/details/52301541)***
> ***[http://www.importnew.com/23774.html](http://www.importnew.com/23774.html)***
> ***[http://www.importnew.com/23774.html](http://www.importnew.com/23780.html)***









