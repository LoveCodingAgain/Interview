## 1.1 JVM基础部分
> JVM组成
> - JVM包含两个子系统和两个组件,两个子系统为Class loader(类装载)、Execution engine(执行引擎);两个组件为Runtime data area(运行时数据区)、Native Interface(本地接口)。
> - 首先通过编译器把 Java 代码转换成字节码，类加载器（ClassLoader）再把字节码加载到内存中，将其放在运行时数据区（Runtime data area）的方法区内，而字节码文件只是 JVM 的一套指令集规范，并不能直接交给底层操作系统去执行，因此需要特定的命令解析器执行引擎（Execution Engine），将字节码翻译成底层系统指令，再交由 CPU 去执行，而这个过程中需要调用其他语言的本地库接口（Native Interface）来实现整个程序的功能。
> - Class loader(类装载)：根据给定的全限定名类名(如：java.lang.Object)来装载class文件到Runtime data area中的method area。
> - Execution engine（执行引擎）：执行classes中的指令。
> - Native Interface(本地接口)：与native libraries交互，是其它编程语言交互的接口。
> - Runtime data area(运行时数据区域)：这就是我们常说的JVM的内存。

> Java程序运行机制
> - 首先利用IDE集成开发工具编写Java源代码，源文件的后缀为.java;
> - 再利用编译器(javac命令)将源代码编译成字节码文件，字节码文件的后缀名为.class;
> - 运行字节码的工作是由解释器(java命令)来完成的;

> 类加载机制
> - 类的加载指的是将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个 java.lang.Class对象，用来封装类在方法区内的数据结构。

> JVM内存区域划分
> 根据JVM模型划分为为程序计数器、Java虚拟机栈、本地方法栈、方法区、堆区
> - 程序计数器（Program Counter Register）：当前线程所执行的字节码的行号指示器，字节码解析器的工作是通过改变这个计数器的值，来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能，都需要依赖这个计数器来完成.【线程私有】
> - Java 虚拟机栈（Java Virtual Machine Stacks）：每个方法在执行的时候也会创建一个栈帧，存储了局部变量，操作数，动态链接，方法返回地址。用于存储局部变量表、操作数栈、动态链接、方法出口等信息.也会抛出StackOverflowError和OutOfMemoryError【线程私有】
> - 本地方法栈（Native Method Stack）：与虚拟机栈的作用是一样的，只不过虚拟机栈是服务 Java 方法的，而本地方法栈是为虚拟机调用 Native 方法服务的.【线程私有】也会抛出StackOverflowError和OutOfMemoryError.
> - Java 堆（Java Heap）：被所有线程共享的一块内存区域，在虚拟机启动的时候创建，用于存放对象实例。Java 虚拟机中内存最大的一块，是被所有线程共享的，几乎所有的对象实例都在这里分配内存.无法继续分配内存会抛出OOM【线程共享】
> - 方法区（Methed Area）：被所有方法线程共享的一块内存区域，用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译后的代码等数据。【线程共享】

> JVM的垃圾回收停顿的影响
> - 假设我们的Minor GC要运行100ms，那么可能就会导致我们的系统直接停顿100ms不能处理任何请求.
> - 在这100ms期间用户发起的所有请求都会出现短暂的卡顿，因为系统的工作线程不在运行，不能处理请求.
> - 因为内存分配不合理，导致对象频繁进入老年代，平均七八分钟一次Full GC，而Full GC是最慢的，有的时候弄不好一次回收要进行几秒钟，甚至几十秒，有的极端场景几分钟都是有可能的.
> - 新的G1垃圾回收器，他更是将采用复杂的回收机制将回收性能优化到机制，尽可能更多的降低Stop the World的时间.
> - 其实JVM本身的迭代演进，就是不断的在优化垃圾回收器的机制和算法，优化内存分配和垃圾回收，尽可能减少垃圾回收的频率，降低垃圾回收的时间，尽可能的降低垃圾回收的过程对我们的系统运行的影响.

> JVM中哪些常见的垃圾回收期和算法
> - Serial和Serial Old垃圾器:分别用来回收新生代和老年代,是单线程执行垃圾回收,垃圾回收时候暂停系统线程,现在已经基本不用,只做了解即可.
> - ParNew和CMS:分别用来回收新生代和老年代,是多线程并发的机制,性能更好,线上常常搭配使用.
> - G1:同一新生代和老年代,堆分区为Region,提供垃圾回收可控暂停时间,更优秀的算法和执行机制.-XX:MaxGCPauseMillis.

> JVM的参数配置
- Tomcat独立部署的项目在catalina.sh中配置JVM参数以及GC日志.SpringBoot项目可以使用jar启动参数及脚本指定
> - -Xms --jvm堆的最小值
> - -Xmx --jvm堆的最大值
> - -XX:MaxNewSize  --新生代最大值
> - -XX:MaxPermSize=1028m  --永久代最大值
> - -XX:+PrintGCTimeStamps 输出GC的时间戳（以基准时间的形式）
> - -XX:+PrintGCDateStamps 输出GC的时间戳（以日期的形式，如 2013-05-04T21:53:59.234+0800）
> - -XX:+PrintGCDetails  --打印出GC的详细信息
> - -verbose:gc --开启gc日志
> - -Xloggc:d:/gc.log -- gc日志的存放位置
> - -Xmn -- 新生代内存区域的大小
> - -XX:SurvivorRatio=8 --新生代内存区域中Eden和Survivor的比例

> JVM的GC日志理解
> - Full GC:表示GC的类型,是YoungGC还是Full GC.
> - 0.0358134 secs:表示本次GC的耗时.
> - 方括号外的3324K->152K(11904K)，表示GC前后Java堆的内存容量变化情况.