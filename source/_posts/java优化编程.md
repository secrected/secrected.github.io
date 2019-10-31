---
title: java优化编程
date: 2018-11-21 15:33:49
categories: [Java,bookNotes]
tags: java
---

> 参考自《java优化编程》 —Java optimize Programming

#### 内存管理

1、垃圾对象：一个对象创建被放入jvm的堆内存中，当永远不在引用这个对象时，它将被jvm在heap中回收。被创建的对象不能再生，无法通过程序语句释放。（对象在jvm运行空间无法通过根集合找到时，该对象为垃圾对象）
2、 栈内存：存静态方法； 堆内存：对象的实例与变量；
3、heap存储的对象被jvm自动回收，不能通过外部手段回收；不能频繁强制系统做垃圾回收，jvm优先完成垃圾回收工作影响性能。
<!--more-->
4、对象的生命周期：
`创建阶段Creation：`
   &nbsp;&nbsp; 创建过程：分配存储空间》构造对象》递归调用超类的构造方法》对象实例初始化和变量初始化》执行构造方法体
    &nbsp; 创建规则：避免在循环体中创建对象；不要对一个对象多次初始化；不要过深的继承层次；访问本地变量由于类中的变量；
`应用阶段Using`
    系统至少维护着对象的一个强引用；除非显示使用了软引用、弱引用或虚引用（辅助finalize函数）
`不可视阶段Invisible`
    对象使用完置为null,可以帮助jvm及时发现该垃圾对象，及时回收。
`不可到达阶段阶段Unreachable`
    通常是所有线程栈中的临时变量，但并不能直接被gc回收。
最后阶段：> `可收集阶段Collected`> `终结阶段Finalized`> `释放阶段Free`
对象情况：该对象不可达，finalize方法已被执行，对象空间已被重用。
5、实现Serializable接口使用transient瞬间值：在远程调用时加快传输速度，瞬间值不会被传递，提高性能。
6、主动清除对象引用 对象=null，可以加速jvm对垃圾内存的回收。内存回收就是回收jvm管理的。
查看jvm回收内存的信息：java -verbosegc 类名
**三种编译方式影响类文件大小：**

| ------------ | 就像西方不能没有耶路撒冷------------ |                                                                                            |
| ------------ |:------------------------ | ------------------------------------------------------------------------------------------ |
| 默认编译         | javac A.java             | 代码（code）、源文件信息（sourcefile infor'ma'tion）、代码行序列表（linenumbertable）                           |
| 调试编译         | javac -g A.java          | 代码（code）、源文件信息（sourcefile infor'ma'tion）、代码行序列表（linenumbertable）、本地变量表（localvariabletable） |
| 代码编译         | javac -g:none  A.java    | 代码（code）                                                                                   |

#### 表达式与保留字

- 使用 == 比equals()方法更快速，适当的前提下可以使用。

- 字符串驻留：String类的 intern() 方法》创建一个唯一字符串集合的驻留池。intern（）方法执行效率高以提高系统性能。

- final关键字：类的常量、方法的常量参数、不可继承的方法、不可继承的类；（类、方法声明final，起到内联的作用，编译器可以将该方法的代码展开插入到调用者代码处，由此可以加快运行速度提高性能。）

- null instanceof 对象：null不是任何对象的引用，返回false；

- 数组拷贝使用
  `System.arraycopy(a,0,c,0,c.length)； // 比循环完成数组拷贝效率高。`

当没有使用Jit或者hotspot时，for循环尽量使用0作为终结条件以提高循环语句的性能：

```
int b = c.length - 1;
for（int i = b;i>=0;i--) {
}
```

#### 散列表类：

- 多线程下要求线程安全：

`List list = Collections.synchronizedList(new ArrayList());﻿​ //vector保存大量内部元素，占内存多50%`

- 处理容量较大数组时：提前调用ensureCapacity()方法初始化ArrayList对象容量，提高应用性能：list.ensureCapacity(N);

- StringBuffer中的append方法累加字符串比用操作符串联的性能高。

- 获取字符：调用toCharArray()方法转成字符数组比调用charAt()省时。（字符串.length在循环条件中使用不会影响性能，属性非调方法。）

- 字符串转数字比数字直接转数字耗时长：new Double(3.45); 优于 new Double(“3.45”

- 输入输出流缓冲数组长度设置（最好512的倍数）：返回的实际可读字节数inputStream.available()；

- 处理压缩文件-压缩流提高效率：java.util.zip.ZipInputStream；

- 通过非阻塞I/o 优化应用性能
  jdk1.4推出的java.nio（新输入/输出） 软件包，有通道、通过缓冲器、字符集、文件映射等进行文件输入输出的新方法。以助于数据处理有更强的性能。

- 数据格式化–》采用操作符格式化数据是首选，次之是预编译，最后非预编译。
  

书中设计风格：方法之间，块行注释之前空一行，逗号后，运算符前折行

> 还有jni程序优化，jsp的优化等，现在几乎用不着不做考究，看了七八章，估计看了本假的书，就像买了盗版的手机，需要习惯


