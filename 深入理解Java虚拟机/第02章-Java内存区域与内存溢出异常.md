
<!-- TOC -->

- [1. 第2章 Java内存区域与内存溢出异常](#1-第2章-java内存区域与内存溢出异常)
    - [1.1. 概述](#11-概述)
    - [1.2. 运行时数据区域](#12-运行时数据区域)
        - [1.2.1. 程序计数器](#121-程序计数器)
        - [1.2.2. Java虚拟机栈](#122-java虚拟机栈)
        - [1.2.3. 本地方法栈](#123-本地方法栈)
        - [1.2.4. Java堆](#124-java堆)
        - [1.2.5. 方法区](#125-方法区)
        - [1.2.6. 运行时常量池](#126-运行时常量池)
        - [1.2.7. 直接内存](#127-直接内存)
    - [1.3. HotSpot虚拟机对象](#13-hotspot虚拟机对象)
        - [1.3.1. 对象的创建](#131-对象的创建)
        - [1.3.2. 对象的内存布局](#132-对象的内存布局)
        - [1.3.3. 对象的访问](#133-对象的访问)
    - [1.4. 实战: OutOfMemoryError异常](#14-实战-outofmemoryerror异常)
        - [1.4.1. Java堆溢出](#141-java堆溢出)
        - [1.4.2. 虚拟机栈和本地方法栈溢出](#142-虚拟机栈和本地方法栈溢出)
        - [1.4.3. 方法区和运行时常量池溢出](#143-方法区和运行时常量池溢出)
        - [1.4.4. 本地内存溢出](#144-本地内存溢出)

<!-- /TOC -->


# 1. 第2章 Java内存区域与内存溢出异常

## 1.1. 概述

C、C++自己负责每个对象生命从开始到终结的维护，手动内存管理

Java在JVM的自动内存管理，不容易出现内存泄漏和内存溢出, 然而一旦出现了这些问题可能更隐蔽、更难解决


## 1.2. 运行时数据区域

### 1.2.1. 程序计数器

PC Register是较小的内存空间，当前线程执行字节码的位置指示器，选取下一条执行的字节码指令

每个线程都有自己的程序计数器

执行Java方法，则PC指向虚拟机字节码指令的地址，如果为Native方法，则PC为空(Undefined)

唯一在Java虚拟机规范中没有规定任何OutOfMemoryError的区域

### 1.2.2. Java虚拟机栈

Java Virtual Machine Stack是线程私有的，描述了Java方法执行的内存模型和栈帧，存储局部变量表，操作数栈、动态链接、方法出口等

局部变量表存放编译期间的各种基本类型和对象引用以及returnAddress(指向一条字节码指令的地址)，运行期间大小不会变

局部变量表中以变量槽Slot表示，64位的long和double需要占用两个变量槽，在编译期间分配完成后，局部变量表大小是完全确定的

JVM规范中，规定了两类异常状况：
1. 线程请求栈深度大于最高值，抛出StackOverflowError异常
2. 栈无法申请到足够内存会抛出OutOfMemoryError异常

### 1.2.3. 本地方法栈

Native Method Stacks是执行JVM使用的本地方法服务的

JVM规范对本地方法栈中的方法使用的语言、方式没有规定，不同的JVM可以根据需要自由实现，甚至HotSpot虚拟机将本地方法栈和虚拟机栈合二为一

本地方法栈也存在栈深度移除或者栈扩展失败分别抛出StackOverflowError和OutOfMemoryError异常

### 1.2.4. Java堆

Java Heap 是被所有线程共享的区域，几乎所有的对象和数组都在堆上分配

然而现在由于JIT编译的进步，随着逃逸分析等技术的强大，栈上分配、标量替换等优化手段,并且可能出现的值类型等，导致Java对象实例都分配在堆上不是那么绝对了

Java的大多数现代垃圾收集器分代，但这不是JVM规范中规定的，也不是某个Java虚拟机的固有实现布局, 只是在十年以前Hotspot虚拟机中的垃圾收集器都是基于经典分代实现的，然而G1出现之后包括最新的ZGC等发展，HotSpot中也出现了不采用分代设计的垃圾收集器

Java堆中划分出线程私有的分配缓冲区TLAB(默认开启)，提升对象的分配效率，Java堆的实现各种各样，只是为了更好的管理和回收内存

JVM规范中规定，Java对可以处于物理不连续空间，只要逻辑上连续即可，对于大对象（大数组等）多数虚拟机为了实现简单、存储高效，会要求连续空间

当前主流JVM的堆空间都是可以扩展的，通过-Xmx(最大堆), -Xms(初始就分配的最小堆), 如果达到上限无法在进行扩展时JVM抛出OutOfMemoryError异常

### 1.2.5. 方法区

Method Area, JVM规范中将方法区描述为堆的一个逻辑部分，但是有一个别名Non-Heap, 与堆区分开来

JDK8以前，Hotspot虚拟机将垃圾收集器的分代设计扩展至方法区，使用永久代(Permanent Generation)实现方法区，使得hotspot可以管理Java堆一样管理方法区，不需要专门编写代码去管理这一部分内存

但是对于其他虚拟机实现例如JRockit、IBM J9等不存在永久代, 而是使用Native Memory实现方法区，其实如何实现方法区并不受JVM规范管束，无统一要求，现在看来，当年使用永久代的实现并不好，容易导致Java程序内存溢出

此JDK6的时候Hotspot开发团队已经考虑放弃永久代，在JDK7的时候将放在永久代的字符串常量池、静态变量移出，在JDK8的时候完全放弃永久代，使用在本地内存中实现的元空间来替代，将JDK7中永久代剩余的内容移动到元空间(主要是类型信息)

JVM规范中规定方法区内存不足时，会出现OutOfMemoryError异常，这部分的垃圾回收效果不好，但是也存在对于常量池的回收，类型卸载等


### 1.2.6. 运行时常量池

Runtime Constant Pool 是方法区的一部分，Class文件中出了类版本、字段、方法、接口等还有常量池表，存放编译器的各种字面量和符号引用，类加载后会存放到方法区的运行时常量池中

运行期也可以将新的常量放入池中，例如String类的intern()方法，常量并不是一定在编译器产生

JDK6在永久代，JDK7在堆，JDK8及以后在元空间


### 1.2.7. 直接内存

Direct Memory 不是JVM规范中定义的内存区域，但是这部分内存也频繁使用，也可能导致OutOfMemoryError异常

JDK1.4中出现的NIO(New Input/Output)类引入Channel与Buffer的IO方式，使用Native函数直接分配堆外内存, 然后通过存储在Java堆中的对象DirectByteBuffer对象作为这块内存的引用进行操作, 可以提高性能，避免数据的复制



## 1.3. HotSpot虚拟机对象

### 1.3.1. 对象的创建

创建过程
1. 遇到字节码new指令是检查指令参数是否能在常量池中定位到类的符号引用, 检查是否被加载，如果没有则执行类加载过程
2. 类加载检查通过之后，为新生对象分配内存(大小在Class中确定), 从堆中划分一块内存
   - **指针碰撞(Bump The Pointer)**：规整内存只需要移动指针向空闲方向移动, Serial Parnew等带压缩整理的收集器使用指针碰撞
   - **空闲列表(Free List)**: 已使用和未使用内存交错存在，虚拟机通过维护一个列表记录可用内存的位置和大小，CMS等基于清除算法的收集器使用空闲列表
   - 创建对象非常频繁，位置指针或者空闲指针移动都需要保证线程安全，虚拟机实际采用CAS失败重试来保证指针更新的原子性，同时通过参数配置是否使用TLAB(Thread Local Allocation Buffer)线程本地分配缓冲区，只有本地缓冲使用完之后才在Heap中使用同步方式分配内存
   - TLAB默认开启，线程中创建一个对象首先会尝试在栈上分配，不能分配栈上其次尝试TLAB，再其次是老年代
3. 内存分配完之后，将分配的内存空间（不包含对象头）初始化为0，使用TLAB时，这项工作可以提前至TLAB分配时进行
4. 对于对象头进行设置：例如对象是哪个类的实例、元信息指针、哈希码（hashCode调用时设置)，GC分代信息，偏向锁信息等
5. new指令之后执行<init>()定义的构造函数, 进行初始化之后，对象才算完全构造完成
6. 将对象信息入栈, 执行下一条指令

### 1.3.2. 对象的内存布局

Java Object由对象头(Object Header)+实例数据（Instance Data)+对齐填充(Padding)组成

对象头Object Header由三者: Mark Word + Klass Pointer + 数组长度(可选)组成:
- Mark Word标记字：64bit, Java对象处于不同的五种状态时, Mark Word的解释形式不一样，包括正常,偏向锁,轻量级锁,重量级锁,GC标志
  - 正常情况下, unused 25, hashcode 31, unused 1 age4, biased_lock0, lock01, 其中age表示新生代GC阈值, hashcode在延迟执行计算并且写入到该对象头中, 在对象加锁之后，MarkWord 64字节需要用于其他用途，没有空间保存hashcode，该值会移动到Monitor中
  - 偏向锁时, 64bit分别为thread 54, epoch 2, unused 1, age 4, biased_lock1, lock01
- Klass Pointer 类型指针, 64bit，用于存储类型元数据的指针，指向方法区该类的字节码对象Class
- 数组长度: 如果对象为Java数组，则对象头还包括用于标记数组长度的数据

实例数据:
- Oops对父类继承和子类定义的字段进行记录，存储顺序受到虚拟机配置策略参数和字段在Java源码中定义的顺序决定
- HotSpot默认顺序是long/double、int、short/char、byte/boolean、oop(Ordinary Object Pointer)
- 在满足上述规则相同宽度字段放在一起的前提下，父类字段放在子类前面, 也可以通过参数配置(默认就为True)允许窄变量插入父类变量的空隙中

对齐填充Padding:
- 不是必然存在, HotSpot自动内存管理的对象起始地址必须要是8字节的整数倍
- 其中object header被设计为8的倍数，如果instance data没有对齐，则需要通过对齐填充来补全


### 1.3.3. 对象的访问

对象创建过程中, 栈上的reference数据来引用Heap上的具体对象
- HotSpot是直接使用指针访问Heap中的Object对象, Object head中的Klass pointer指向方法区的对象类型数据
- 句柄访问，Java堆中存在句柄池，保存对象实例指针和对象类型指针,这样reference存储的就是稳定的句柄地址，垃圾收集对象移动时referenc不需要变化


## 1.4. 实战: OutOfMemoryError异常

JVM规范中规定，除了程序计数器之外其它运行时区域都可能发生OutOfMemoryError(OOM)

### 1.4.1. Java堆溢出

参数配置
```
-Xmx20M  # 堆内存初始大小，默认物理内存的1/64
-Xms20M  # 堆内存最大容量，默认物理内存的1/4
-Xmn10M  # 新生代大小
-XX:+HeapDumpOnOutOfMemoryError # JVM出现内存溢出是Dump当前内存堆转储快照
-XX:-PrintGCDetails
```
通过不断创建对象，保证GC Roots到对象之间有可达路径避免垃圾回收机制来清除这些对象
```
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid3108.hprof ...
Heap dump file created [26236546 bytes in 0.147 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
```
通过JProfiler分析内存映像转储快照
- 内存泄漏: 通过查看泄漏对象到GC Roots的引用链, 找到泄漏对象是通过怎样途径与GC Roots关联才导致垃圾回收器无法回收它们，从而找到对象创建的位置
- 内存溢出: 如果内存中的对象确实都必须是存活的，则需要设置堆参数(-Xmx和-Xms)，查看机器物理内存是否还有扩展空间, 在检查代码逻辑是否使部分对象存活时间过长、存储结构设计不合理等，减少运行期间的内存消耗


### 1.4.2. 虚拟机栈和本地方法栈溢出

如果线程请求的栈深度大于JVM允许的最大栈深度，抛出StackOverflowError, 如果栈内存允许动态扩展并且无法申请到足够内存时抛出OutOfMemoryError(HotSpot虚拟机不支持栈动态扩展, 因此只会导致栈容量无法满足新的栈帧导致SOE而不是OOM)

-Xss128K设置栈容量，Linux 64位系统 JDK11中栈容量最小值不能低于228K

```
stack length: 11417
java.lang.StackOverflowError
	at edu.nju.parse.TestOOM.stackLeak(TestOOM.java:16)
	at edu.nju.parse.TestOOM.stackLeak(TestOOM.java:17)
	at edu.nju.parse.TestOOM.stackLeak(TestOOM.java:17)
	at edu.nju.parse.TestOOM.stackLeak(TestOOM.java:17)
```


### 1.4.3. 方法区和运行时常量池溢出

HotSpot从JDK7中将常量池移到Java Heap中，JDK8中用元空间取代永久代

String::intern()是一个本地方法，如果字符串常量池中已经包含一个等于此String()对象的字符串，则返回池中该字符串的引用，否则将String对象包含的字符串添加到常量池中，然后返回String对象的引用

```java
// String对象为"计算机软件"在Heap中
String str1 = new StringBuilder("计算机").append("软件").toString();
// str1.intern中返回的常量池中记录了首次出现的实例引用，因此两者指向同一个堆内存
System.out.println(str1.intern() == str1); // true

// "java"已经出现在了常量池中，因此str2.intern返回的常量池中"java"的地址，字符串对象在堆中的地址不一样
String str2 = new StringBuilder("ja").append("va").toString();
System.out.println(str2.intern() == str2); // false
```

JDK6中都是false,false, JDK7以上是true, false
- JDK6中intern是将首次遇到的字符串实例**复制**到永久代的常亮池中存储，返回永久代的字符串实例的地址引用, 而StringBuilder创建的字符串对象实例在Java堆上，因此不是同一个引用
- JDK7及以上HotSpot的intern()方法的实现不需要拷贝到永久代，既然字符串常量池已经移动到Java堆中，只需要在常量池中**记录首次出现的实例引用**就行，因此第一个intern()返回的引用和StringBuilder创建的字符串实例就是同一个, 而"java"这个字符串是在加载sum.misc.Version类时就已经进入常量池, 不符合首次遇到原则

方法区存放类型相关信息: 类名、访问修饰符、常量池、字段描述、方法描述

通过CGLib等生成动态类等需要更大的方法区来保证动态生成的新类型可以载入内存，并且很多运行在JVM上的语言Groovy等需要通过持续创建新类型来支持语言的动态性，因此方法区溢出的场景也会遇到.

JDK8中元空间默认设置下很难产生方法区溢出，但是HotSpot也提供了一些参数进行元空间的防御措施，例如指定初始大小和最大大小，进行类型卸载等
```
-XX:MaxMetaspaceSize: 最大值, -1表示只受本地内存限制
-XX:MetaspaceSize: 初始大小，操作之后可能进行类型卸载，直到最大大小
```

### 1.4.4. 本地内存溢出

Direct Memory本机直接内存容量也可以进行设置（-XX:MaxDirectMemorySize)，默认和Java堆最大值(-Xmx)一致

```
java.lang.OutOfMemoryError
	at sun.misc.Unsafe.allocateMemory(Native Method)
	at edu.nju.parse.TestOOM.main(TestOOM.java:44)
```