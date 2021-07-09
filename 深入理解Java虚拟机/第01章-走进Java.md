<!-- TOC -->

- [1. 第一章 走进Java](#1-%e7%ac%ac%e4%b8%80%e7%ab%a0-%e8%b5%b0%e8%bf%9bjava)
  - [1.1. 概述](#11-%e6%a6%82%e8%bf%b0)
  - [1.2. Java技术体系](#12-java%e6%8a%80%e6%9c%af%e4%bd%93%e7%b3%bb)
  - [1.3. Java发展史](#13-java%e5%8f%91%e5%b1%95%e5%8f%b2)
  - [1.4. Java虚拟机家族](#14-java%e8%99%9a%e6%8b%9f%e6%9c%ba%e5%ae%b6%e6%97%8f)
  - [1.5. 展望Java技术的未来](#15-%e5%b1%95%e6%9c%9bjava%e6%8a%80%e6%9c%af%e7%9a%84%e6%9c%aa%e6%9d%a5)
    - [1.5.1. 无语言倾向](#151-%e6%97%a0%e8%af%ad%e8%a8%80%e5%80%be%e5%90%91)
    - [1.5.2. 新一代即时编译器](#152-%e6%96%b0%e4%b8%80%e4%bb%a3%e5%8d%b3%e6%97%b6%e7%bc%96%e8%af%91%e5%99%a8)
    - [1.5.3. 向Native迈进](#153-%e5%90%91native%e8%bf%88%e8%bf%9b)

<!-- /TOC -->

# 1. 第一章 走进Java

## 1.1. 概述

一次编写，到处运行: Write Once Run Anywhere

自动内存管理

热点代码检测和运行时编译优化，Java程序随着运行时间的增长而获得更高的性能

## 1.2. Java技术体系

JDK = Java程序设计语言 + Java虚拟机 + Java类库

JRE = Java虚拟机 + Java类库中的Java SE API子集

## 1.3. Java发展史

1991年James Gosling开发了Oak语言，在1995年蜕变成Java语言: Write Once, Run Anywhere

1999年4月, HotSpot虚拟机诞生

2004年, JDK1.x命名方式改为JDK x方式， JDK5中添加了很多语法特性

2006年 JDK在GPL v2协议下公开源码， OpenJDK7， 在JDK7的Update 4中 G1收集器开始商用

2014年 JDK8发布， 添加了JEP来定义和管理功能特性

2017年 JDK9发布，主要添加了Jigsaw模块化规范, 并且每年3月和6月将发布一个大版本，并且每六个版本划出一个长期支持LTS版本,2021的JDK17

2018年 Google和Oracle冠以Android的Java API版权问题最终判决, 3月Oracle宣布Java EE成为历史, 赠送给Eclipse基金会, 改名为Jakarta EE

2018年9月 JDK11发布，LTS版本JDK, ZGC革命性的垃圾收集器

Oracle收购Sun 是 Java 发展史的分界线，发展速度变快，引起竞争的同时带来新的活力

## 1.4. Java虚拟机家族

Sun Classic/Exact VM: 虚拟机始祖，精确内存管理

HotSpot VM是OracleJDK和OpenJDK中的默认虚拟机, 热点代码探测

Google Android Dalvik VM: Android的核心组件，使用寄存器架构而不是栈架构，不过在Android4.4中ART虚拟机支持提前编译，取代了Dalvik虚拟机, Android 5 ART就变为了默认虚拟机

## 1.5. 展望Java技术的未来

### 1.5.1. 无语言倾向

2018年Oracle Labs发布Graal VM，"Run Programs Faster Anywhere", 在Hotspot虚拟机增强的跨语言全栈虚拟机

支持Java、Scala、Groovy、Kotlin等基于JVM上的语言，以及C、C++、Rust等基于LLVM的语言，也支持JavaScript、Ruby、Python、R等语言

Graal VM基本原理是将语言源代码(Javascript)或者中间代码格式(LLVM字节码)通过解释器转换为Graal VM的中间表示(IR)


### 1.5.2. 新一代即时编译器

长时间运行的应用经过充分预热，热点代码会被HotSpot探测定位，将其直接编译为物理硬件可直接执行的机器码，JIT编译

JDK10中加入了全新的即时编译器Graal Compiler，代替了服务器端编译器C2（HotSpot Server Compiler），Graal编译器使用Java编写而不是像C2使用C++编写

Graal编译器的开发效率和扩展性几乎可以追平C2，甚至部分反超C2编译器，但是目前仍然带着"实验状态"的标签，需要用开关参数来启用Graal编译器

### 1.5.3. 向Native迈进

Java启动时间较长，并且需要预热才能得到更高的性能, 和微服务以及无服务(AWS Lambda允许的最长运行时间为15分钟）相悖

为了使用最新的软件服务，JDK逐步提出AppCDS提高下次启动速度、无操作垃圾收集器、提前编译等

提前编译是将部分bytecode直接预编译为二进制代码，减少JIT的预热时间，但是破坏了Java的Write Once Run Anywhere的原则

Substrate VM是Graal VM 0.20版本中的极小型运行时环境，包括许多独立的机制，进行提前编译AOT, 显著降低内存占用和启动时间，但相应地，原理上也决定了Substrate VM必须要求目标程序是完全封闭的，即不能动态加载其他编译期不可知的代码和类库。基于这个假设，Substrate VM才能探索整个编译空间，并通过静态分析推算出所有虚方法调用的目标方法。



