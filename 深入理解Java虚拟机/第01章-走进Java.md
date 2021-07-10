
- [1. 第一章 走进Java](#1-第一章-走进java)
  - [1.1. 概述](#11-概述)
  - [1.2. Java技术体系](#12-java技术体系)
  - [1.3. Java发展史](#13-java发展史)
  - [1.4. Java虚拟机家族](#14-java虚拟机家族)
  - [1.5. 展望Java技术的未来](#15-展望java技术的未来)
    - [1.5.1. 无语言倾向](#151-无语言倾向)
    - [1.5.2. 新一代即时编译器](#152-新一代即时编译器)
    - [1.5.3. 向 Native 迈进](#153-向-native-迈进)
    - [1.5.4. 灵活的胖子](#154-灵活的胖子)
  - [1.6. 实战：自己编译JDK](#16-实战自己编译jdk)

# 1. 第一章 走进Java

## 1.1. 概述

一次编写，到处运行

内存管理

热点代码检测和运行时编译优化，Java程序随着运行时间的增长而获得更高的性能

## 1.2. Java技术体系

JDK = Java程序设计语言 + Java虚拟机 + Java类库

JRE = Java虚拟机 + Java SE API子集

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

Google Android Dalvik VM: Android的核心组件，使用寄存器架构而不是栈架构，不过在Android4.4中ART虚拟机支持提前编译，取代了Dalvik虚拟机

## 1.5. 展望Java技术的未来

### 1.5.1. 无语言倾向

2018年Oracle Labs发布Graal VM，"Run Programs Faster Anywhere", 在Hotspot虚拟机增强的跨语言全栈虚拟机

支持Java、Scala、Groovy、Kotlin等基于JVM上的语言，以及C、C++、Rust等基于LLVM的语言，也支持JavaScript、Ruby、Python、R等语言

Graal VM基本原理是将语言源代码(Javascript)或者中间代码格式(LLVM字节码)通过解释器转换为Graal VM的中间表示(IR)


### 1.5.2. 新一代即时编译器

长时间运行的应用经过充分预热，热点代码会被HotSpot探测定位，将其直接编译为物理硬件可直接执行的机器码，JIT编译

JDK10中加入了全新的Graal编译器，代替了服务器端编译器C2（HotSpot Server Compiler），Graal编译器使用Java编写而不是像C2使用C++编写

Graal编译器的开发效率和扩展性几乎可以追平C2，甚至部分反超C2编译器，但是目前仍然带着"实验状态"的标签，需要用开关参数来启用Graal编译器

### 1.5.3. 向 Native 迈进

Java启动时间较长，并且需要预热才能得到更高的性能, 和微服务以及无服务(AWS Lambda允许的最长运行时间为15分钟）相悖

为了使用最新的软件服务，JDK逐步提出AppCDS提高下次启动速度、无操作垃圾收集器、提前编译等

提前编译是将部分bytecode直接预编译为二进制代码，减少JIT的预热时间，但是破坏了Java的Write Once Run Anywhere的原则

Substrate VM是Graal VM 0.20版本的极小型运行时环境，包括许多独立的机制，进行提前编译AOT, 显著降低内存占用和启动时间. 但相应地，原理上也决定了Substrate VM必须要求目标程序是完全封闭的，即不能动态加载其他编译期不可知的代码和类库。基于这个假设，Substrate VM才能探索整个编译空间，并通过静态分析推算出所有虚方法调用的目标方法。

### 1.5.4. 灵活的胖子

HotSpot编译时指定一系列特性开关，反应在源代码阶段不只是条件编译，更关键的是接口和实现分离

- JDK5: 抽象出虚拟机工具接口
- JDK9: 开放了java语言级别的编译器接口, Graal编译器可以移植到hotSpot中
- JDK10: 重构了Java虚拟机的垃圾收集器接口，因此垃圾收集器可以退役，JDK12中的Shenandoah这样由第三方开发的垃圾收集器才可以进入Hotspot中


## 1.6. 实战：自己编译JDK

下载源码和 Bootstrap JDK

```shell
# 安装Command Line Tools
xcode-select --install
# 下载openjdk12源代码
https://hg.openjdk.java.net/jdk/jdk12/
# 国内网速较慢可以在github下载
https://github.com/openjdk/jdk/
# 安装包方式下载安装OpenJDK11
https://adoptopenjdk.net/index.html?variant=openjdk11&jvmVariant=hotspot
```


安装编译环境
```shell
# homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
# 加速编译
brew install ccache
# 字体引擎，编译过程使用
brew install freetype
brew install autoconf
```
