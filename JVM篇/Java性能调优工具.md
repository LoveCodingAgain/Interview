## 1.1 问题背景
- OutOfMemoryError【内存不足】
- 内存泄露
- 线程死锁
- 锁争用（Lock Contention）
- Java进程消耗CPU过高

一、 jps(Java Virtual Machine Process Status Tool):基础工具
> ps -ef|grep java
> jps主要用来输出JVM中运行的进程状态信息, jps -l | grep name语法格式如下:  
jps [options]
命令行参数选项说明如下：
> -q 不输出类名、Jar名和传入main方法的参数
> -m 输出传入main方法的参数
> -l 输出main类或Jar的全限名
> -v 输出传入JVM的参数 

二、 jstack
> jstack主要用来查看某个Java进程内的线程堆栈信息。语法格式如下:
> jstack [option] pid

> jstack [option] executable core

> jstack [option] [server-id@]remote-hostname-or-ip

> 常用:jstack -l pid > XXX.txt

> 获取Java的十六进制的线程名称然后在堆栈中过滤.1、top -Hp pid 2、printf "%x" 21742 3、jstack 21711 | grep 54ee

三、jstat
> 这个指令用来查看jvm统计信息
> 包括类装载
  常用:jstat -class [pid] 1000 10
> 这个指令指的是每1秒（1000就是1000ms）查看一次类的装载信息，一共看十次。后面的1000和10可以不要

> 垃圾收集情况
-gc、-gcutil、-gccause、-gcnew、-gcold
jstat -gcutil pid 1000 10

> S0C、S1C、S0U、S1U：Survivor 0/1区容量（Capacity）和使用量（Used）
> EC、EU：Eden区容量和使用量
> OC、OU：年老代容量和使用量
> PC、PU：永久代容量和使用量
> YGC、YGT：年轻代GC次数和GC耗时
> FGC、FGCT：Full GC次数和Full GC耗时
> GCT：GC总耗时
> JIT编译

> 估算一个系统的运行期期间的信息
> - 新生代对象的增长速率
> - Young GC的频率和耗时
> - 每次YoungGC后还有多少对象存活下来
> - 每次YpuinGC有多少对象进入老年代
> - 老年代对象的增长速率
> - FullGC的频率和耗时

四、 jmap
> jmap导出堆内存快照，然后使用jhat来进行分析或者JProfiler来分析
> jmap语法格式如下：
> jmap [option] pid
> jmap [option] executable core
> jmap [option] [server-id@]remote-hostname-or-ip

> 使用jmap -heap pid
> 查看进程堆内存使用情况，包括使用的GC算法、堆配置参数和各代中堆内存使用情况.
> 使用jmap -histo:live pid | more查看堆内存中的对象数目、大小统计直方图，如果带上live则只统计活对象.
> class name是对象类型,说明如下:
- B  byte
- C  char
- D  double
- F  float
- I  int
- J  long
- Z  boolean
- [  数组，如[I表示int[]
- [L+类名 其他对象

> 还有一个很常用的情况是：用jmap把进程内存使用情况dump到文件中，再用jhat分析查看。jmap进行dump命令格式如下:
> jmap -dump:format=b,file=heapdump.hprof pid; format=b,表示以字节的形式
