# JVM

------



[TOC]

------

## java程序运行过程

1. Java源文件被编译器编译成字节码文件
2. jvm将字节码文件编译成相对应操作系统的机器码
3.  机器码调用相应操作系统的本地方法库执行相应的方法

## JVM架构



JVM包括 **类加载子系统**，**运行时数据区**，**执行引擎**，**本地接口库**

类加载子系统：将编译好的字节码文件加载到jvm

运行时数据区：存储jvm运行过程中产生的数据

执行引擎：包括即时编译器，垃圾回收器

即时编译器JIT：将java字节码文件编译成具体的机器码

本地接口库：用于调用操作系统的本地方法库

## JVM 内存区域（运行时数据区）存储内容，作用

线程私有区域：程序计数器，虚拟机栈，本地方法栈

线程共享区域：方法区，堆

![img](../图片/132ba6ba720f2bfc6c69b1ce490f7c87693987.jpg)

* **本地方法栈** (Native Stack)：管理本地方法的调用，在 HotSopt JVM 中，直接将本地方法栈和虚拟机栈合二为一，Native方法Java 调用非 Java 代码的接口，比如 C/C++    

     Java虚拟机规范对该区域规定了StackOverflowError异常和OutOfMemoryError异常。

* **虚拟机栈** (VM Stack) ：管运行，**局部变量表（为8种基本类型的变量+引用对象+实例方法分配空间）** 生命周期随着线程，线程启动而产生，线程结束而消亡。用于**存储栈帧（java一个方法就是一个栈帧）**（存储局部变量表、操作数栈、动态连接、方法返回地址、附加信息等信息）

     Java虚拟机规范对该区域规定了StackOverflowError异常和OutOfMemoryError异常。

     会抛出Error错误 StackOverflowError：栈溢出 **原因 : 函数调用栈太深了,注意代码中是否有了循环调用方法而无法退出的情况**

* **程序计数器**(PC Register)：记录了方法之间的调用和执行情况，类似值日排班表，用它来存储指向下一条指令的地址，也就是即将要执行的指令代码，他是当前线程所执行的字节码的行号指示器

     此内存是唯一一个在java虚拟机规范中没有规定OutOfMemoryError情况的区域。

* **方法区(规范)**：**静态变量 + 常量 + 类信息(构造方法/接口定义) + 运行时常量池存在方法区中** ，是一种**规范**，不同虚拟机实现不同，实例变量在堆中，与方法区无关

     1.7 表现为**永久带** **1.8后改为元空间(在堆外内存上)**

     Java虚拟机规范对该区域规定了OutOfMemoryError异常。

* **heap Space 堆**：**储存对象实例和数组**（JDK7 已把**字符串常量池和类的静态变量**移动到 Java 堆）

     Java虚拟机规范对该区域规定了OutOfMemoryError异常。


* **运行时常量池**
    1. **运行时常量池(Runtime Constant Pool) 逻辑是方法区的一部分**(虚拟机规范)。 **Class文件中除了有类的版本, 字段,方法, 接口等描述信息外**, 还有一项信息是**常量池(Constant Pool Table)**, 用于存放编译期生成的**各种字面量和符号引用**, 这部分内容**将在类加载后存放到方法区的运行时常量池中**。
    2. 运行时常量池相对于Class文件常量池的另外一个重要特征是**具备动态性**, Java语言并不要求常量一定只能在编译期产生, 也就是并非预置入Class文件中常量池的内容才能进入方法区运行时常量池, 运行期间也可能将新的常量放入池中, 这种特性被开发人员利用的比较多的便是String类的intern() 方法。

* **直接内存**
    1. 在**JDK1.4 中新加入的NIO 类**，引入了**一种基于通道（Channel）和缓冲区（Buffer）的I/O 形式**，他可以使用Native 函数**直接分配堆外内存**，然后通过一个存储在Java 堆中的DirectByteBuffer 对象作为这块内存的引用进行操作。这样能在一些场所显著提高性能，因为**避免了在Java 堆和Native 堆中来回复制数据**。
    2. 直接内存(Direct Memory) 并不是虚拟机运行时数据区的一部分, 也**不是Java虚拟机规范中定义的内存区域**, 但是这部分内存也被频繁地使用, 而且也可能导致**OutOfMemoryError**异常出现。 显然, 本机直接内存的分配不会受到Java堆大小的限制, 但是, 既然是内存, 则肯定还是会受到本机总内存的大小及处理器寻址空间的限制。 服务器管理员配置虚拟机参数时, 一般会根据实际内存-Xmx等参数信息, 但经常会忽略到直接内存, 使得各个内存区域的总和大于物理内存限制(**包括物理上的和操作系统级的限制**), 从而导致动态扩展时出现OutOfMemoryError异常。

## 系统参数

cmd设置系统参数   java -D(参数名)=值     

程序中通过System.getProperty(参数名)获取值  

## jvm启动参数 

```java
  -Xms2g -Xmx2g -Xmn1g -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=256m -XX:SurvivorRatio=8 -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:CMSMaxAbortablePrecleanTime=5000 -XX:+CMSClassUnloadingEnabled -XX:CMSInitiatingOccupancyFraction=80 -XX:+DisableExplicitGC -verbose:gc -Xloggc:/data/logs/app-gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Dfile.encoding=UTF-8 -Djava.awt.headless=true -XX:+UseCompressedOops -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/data/logs/app.dump -XX:MaxDirectMemorySize=256m -XX:+UseCMSInitiatingOccupancyOnly -XX:+ExplicitGCInvokesConcurrent
```

> * **\- 标准参数**(所有的JVM实现都必须实现这些参数的功能，而且向后兼容)
>
> > * -server参数(服务端常使用	使用并行垃圾回收器 启动慢运行快) 
> > * -client参数(客户端常使用      使用串行垃圾回收器 启动快运行慢)
> >     `windows32位默认使用-client  64位只有-server参数不支持-client`
>
> * **-X  非标准参数**(这些参数不是虚拟机规范规定的。因此，不是所有VM的实现(如:HotSpot,JRockit,J9等)都支持这些配置参数。)
>
> > 				-X:int  (Interpreted mode) 解释模式 强制JVM执行所有字节码 会降低速度
> > 				-X:comp ()编译模式 第一次使用会将所有的字节码文件编译成本地文件,从而带来最大优化
> > 				-X:mixed 混合模式 jvm自己决定使用解释模式还是编译模式
>
> * **-XX (常用参数)非稳定参数，随时可能被修改或者移除。 主要用于jvm调优和debug操作 **
>
> > **使用方式:**  
> >
> > >1. boolean类型
> > >
> > >    ```
> > >    	-XX:[+/-]<name> 表示启动或禁用<name>属性
> > >    例如 -XX:+DisableExplicitGC  表示启动 禁用手动调用gc操作 System.gc()无效
> > >    ```
> > >
> > >    
> > >
> > >2. 非boolean类型
> > >
> > >    ```
> > >    通过	-XX:<name>=<value>	来配置
> > >    例如: -XX:NewRatio=1  表示新生代和老年代的比值 
> > >    ```
> > >
> 
> * **内存参数	-Xms参数与-Xmx参数  属于-XX参数**
> 
>     ```
>     -Xms512m(-XX:InitialHeapSize) 设置初始化堆内存参数为512m
>     -Xmx2048m(-XX:MaxHeapSize) 设置最大堆内存参数为2048m
>     适当调整jvm的内存大小,可以充分利用服务器资源,让程序跑的更快
>    ```

## 查看JVM运行参数

1.  运行java命令时打印出运行参数
          `java  -XX:+PrintFlagsFinal edu.zsq.jvm.TestJVM`
                   ` :=表示参数被修改  =表示参数默认值`

2.  修改java参数
               

    ```
      (修改boolean类型参数)   java  -XX:+PrintFlagsFinal -XX:+ZeroTLAB edu.zsq.jvm.TestJVM
      (修改非boolean类型参数)		java  	-XX:+PrintFlagsFinal -XX:hashCode=4 edu.zsq.jvm.TestJVM
    ```

3.   查看正在运行的jvm的参数
            查看所有参数 jinfo -flags <进程id>
            查看单个参数 jinfo -flag <参数名> <进程id>
     

## 堆内存

年轻代(8:1:1)		老年代 	元数据(1.7永久代)
				1			:		2

## jstat命令对堆内存进行统计分析

```
 1 查看class加载统计
        jstat -class <进程id>
        jstat -class 11548
        Loaded  Bytes  Unloaded  Bytes     Time
          3886  8367.6        1     0.9       2.19
 2 查看编译统计
        jstat -compiler <进程id>
        jstat -compiler 11516
        Compiled(编译数量) Failed Invalid(不可用的)   Time   FailedType(失败类型) FailedMethod(失败方法)
          2824               0       0              3.77          0
 3 垃圾回收统计
        jstat -gc <进程id>  (打印间隔)  (打印次数)
```

## Jmap命令对堆内存使用情况进行分析

* **查看内存使用情况**

    ```
     jmap -heap 端口号
            Heap Configuration:
               MinHeapFreeRatio         = 0
               MaxHeapFreeRatio         = 100
               MaxHeapSize              = 4261412864 (4064.0MB)
               NewSize                  = 88604672 (84.5MB)
               MaxNewSize               = 1420296192 (1354.5MB)
               OldSize                  = 177733632 (169.5MB)
               NewRatio                 = 2
               SurvivorRatio            = 8
               MetaspaceSize            = 21807104 (20.796875MB)
               CompressedClassSpaceSize = 1073741824 (1024.0MB)
               MaxMetaspaceSize         = 17592186044415 MB
               G1HeapRegionSize         = 0 (0.0MB)
    
            Heap Usage:
            PS Young Generation
            Eden Space:
               capacity = 102236160 (97.5MB)
               used     = 46662296 (44.500633239746094MB)
               free     = 55573864 (52.999366760253906MB)
               45.6416751176883% used
            From Space:
               capacity = 11010048 (10.5MB)
               used     = 10988408 (10.479362487792969MB)
               free     = 21640 (0.02063751220703125MB)
               99.80345226469494% used
            To Space:
               capacity = 11010048 (10.5MB)
               used     = 0 (0.0MB)
               free     = 11010048 (10.5MB)
               0.0% used
            PS Old Generation
               capacity = 119537664 (114.0MB)
               used     = 13861632 (13.219482421875MB)
               free     = 105676032 (100.780517578125MB)
               11.596037212171053% used
    
            17354 interned Strings occupying 1533672 bytes.
    ```

*  **查看内存中对象数量及大小**

    ```
     jmap -histo <pid>   B:byte  C:char D:double  F:float  I:int J:long Z:boolean  [表示数组 [L+类名 表示其他对象
               1:         21622       22101104  [I
               2:        136839       14875896  [C
               3:          7291       13887592  [B
               4:        109614        2630736  java.lang.String
               5:         44476        1423232  java.util.HashMap$Node
               6:         33313        1411672  [Ljava.lang.String;
            查看活跃对象数量及大小
             jmap -histo:live <pid>
    ```

* **将内存使用情况dump到.hprof文件(内存快照文件)中**

    ```
     jmap -dump:format=b,file=文件路径 <pid>
            6.1 对文件进行分析jhat命令
                  jhat -prot <端口> <file>
                  >jhat -11548 9999 E:\IDEA\project\test.hprof
    
            6.2 Idea2020支持打开.hprof文件进行简单分析 双击shift 输入 open Hprof Snapshot
    ```

* **内存溢出的定位于分析**

    ```
    运行TestJvmOutOfMemory类 并添加参数-Xms8m -Xmx8m -XX:+HeapDumpOutOfMemoryError
    ```

    

## jstack命令查看jvm中线程执行情况
    用法: jstack <pid>

# GC

* GC 本身有三种语义，下文需要根据具体场景带入不同的语义：
  * **Garbage Collection**：垃圾收集技术，名词。
  * **Garbage Collector**：垃圾收集器，名词。
  * **Garbage Collecting**：垃圾收集动作，动词。

## GC算法

1. 标记—清除算法	**Mark-Sweep**

    ​	标记可回收对象，清除可回收对象，有内存碎片化问题，会出现大对象无法获得连续可用空间的问题

    ​	回收过程主要分为两个阶段，第一阶段为追踪（Tracing）阶段，即从 GC Root 开始遍历对象图，并标记（Mark）所遇到的每个对象，第二阶段为清除（Sweep）阶段，即回收器检查堆中每一个对象，并将所有未被标记的对象进行回收，整个过程不会发生对象移动。整个算法在不同的实现中会使用三色抽象（Tricolour Abstraction）、位图标记（BitMap）等技术来提高算法的效率，存活对象较多时较高效

2. 复制算法  **Copying**

    ​	将内存划分为区域一和区域二，新生对象都存放在区域一，对区域一对象进行标记清除，存活下来的对象复制到区域二，清理整个区域一内存。有内存浪费的问题

    ​	将空间分为两个大小相同的 From 和 To 两个半区，同一时间只会使用其中一个，每次进行回收时将一个半区的存活对象通过复制的方式转移到另一个半区。有递归（Robert R. Fenichel 和 Jerome C. Yochelson提出）和迭代（Cheney 提出）算法，以及解决了前两者递归栈、缓存行等问题的近似优先搜索算法。复制算法可以通过碰撞指针的方式进行快速地分配内存，但是也存在着空间利用率不高的缺点，另外就是存活对象比较大时复制的成本比较高。

3. 标记整理算法 **Mark-Compact**

     在经过标记清除算法同样的流程后，将存活对象移动到内存另一端，然后清除该端对象，并释放内存。

     ​		这个算法的主要目的就是解决在非移动式回收器中都会存在的碎片化问题，也分为两个阶段，第一阶段与 Mark-Sweep 类似，第二阶段则会对存活对象按照整理顺序（Compaction Order）进行整理。主要实现有双指针（Two-Finger）回收算法、滑动回收（Lisp2）算法和引线整理（Threaded Compaction）算法等。

     ![img](../图片/07284b598f5a1006d9a6a6cdee7ed1b794177.png@1604w_294h_80q)

4. 分代收集算法

     新生代采用复制算法（因为新生代大量对象被回收，需要复制的对象少，更安全高效），老年代采用标记整理或者标记清除算法

## 如何确定垃圾 

1. 引用计数法**（Reference Counting）**

    为对象添加引用，引用计数加1，为对象删除引用，引用计数减1，如果引用计数为0，进行回收， 有循环引用的问题.

    解决循环引用: 

    * 通过 **Recycler** 算法解决,  但是在多线程环境下，引用计数变更也要进行昂贵的同步操作，性能较低，早期的编程语言会采用此算法。
    * 通过智能指针(强软引用)解决,智能指针在整个 Android 工程中使用很广泛**「若 A 强引用了 B，那 B 引用 A 时就需使用弱引用，当判断是否为无用对象时仅考虑强引用计数是否为 0，不关心弱引用计数的数量」**

2. 可达性分析**又称引用链法（Tracing GC）**

    定义一些GC Roots对象，以**GC Roots对象作为起点向下搜索**，如果在GC Roots和一个对象之间没有**可达路径**，则标记该对象为不可达，此时还不足以判断对象是否存活/死亡，需要经过**多次标记**才能更加准确地确定。整个**连通图之外**的对象便可以作为垃圾被回收掉。目前 Java 中主流的虚拟机均采用此算法。

## GC ROOT对象有哪些

1. 虚拟机栈和本地方法栈中的引用对象
2. 方法区中常量和类静态属性引用对象



## JVM中一次完整的GC ? 对象是如何晋升到老年代的?

分为新生代和老年代，新生代默认占总空间的 1/3，老年代默认占 2/3。 新生代使用复制 算法 ，有 3 个分区：Eden、To Survivor、From Survivor，它们的默认占比是 8:1:1。当新生代中的 Eden 区内存不足时，就会触发 Minor GC（YoungGC）。

新生代回收过程如下：

在 Eden 区执行了第一次 GC 之后，存活的对象会被移动到其中一个 Survivor 分区；

Eden 区再次 GC，这时会采用复制 算法 ，将 Eden 和 from 区一起清理，存活的对象会被复制到 to 区；

移动一次，对象年龄加 1，对象年龄大于一定阀值会直接移动到老年代。GC年龄的阀值可以通过参数 -XX:MaxTenuringThreshold 设置，默认为 15。

Survivor 区相同年龄所有对象大小的总和 > (Survivor 区内存大小 * 这个目标使用率)时，大于或等于该年龄的对象直接进入老年代。其中这个使用率通过 -XX:TargetSurvivorRatio 指定，默认为 50%

Survivor 区内存不足会发生担保分配，超过指定大小的对象可以直接进入老年代。

老年代回收（FullGC）触发条件：

大对象将直接进入老年代。要控制大对象的阀值可以通过 -XX:PretenureSizeThreshold 参数设置，但是它只对 Serial 和 ParNew 回收器生效，对 Parallel Scavenge 不生效。

无法放入Survivor区直接进入老年代。YoungGC时，如果eden区+ from survivor 区存活的对象无法放到 to sur vivo r 区了，这个时候会直接将部分对象放入到老年代。

每次晋升到老年代的对象平均大小 > 老年代剩余空间

System.gc() 可能会引起.

## 七大垃圾收集器

针对新生代

1. serial 单线程复制算法。进行垃圾收集时，暂停其他所有工作线程，直到结束
2. ParNew 多线程 复制算法，和serial一致，不过是多线程的
3. Parallel Scavenge 多线程复制算法。可以提高新生代垃圾收集效率，系统吞吐量上有很大优化，可以更高效的利 用cpu完成垃圾回收任务 

针对老年代

4. Serial Old 单线程标记整理
5. Parallel Old 多线程 标记整理
6. CMS 多线程标记清除 不需要暂停用户线程。并发度和效率大大提升
7. G1 多线程标记整理

### CMS

CMS（Concurrent Mark Sweep）是以一种获取最短回收停顿时间为目标的收集器。重视响应，可以带来好的用户体验，被sun称为并发低停顿收集器】

```css
启用CMS：-XX:+UseConcMarkSweepGC
```

正如其名，CMS采用的是"标记-清除"(Mark Sweep)算法，而且是支持并发(Concurrent)的

它的运作分为4个阶段

```css
（1）初始标记：暂停所有的工作线程。标记 GC Roots能直接关联的对象，速度很快
（2）并发标记：和用户线程一起工作，进行 GC Roots 跟踪标记
（3）重新标记：暂停所有的工作线程，标记出那些在并发标记过程中遗漏的，或者内部引用发生变化的对象
（4）并发清除：和用户线程一起工作，清除 GC Roots 不可达对象
由于耗时最长的在于并发标记和并发清除过程，GC线程可以和用户线程一起并发工作，所以总体上来看CMS 是并发的。
```

以上初始标记和重新标记需要stop the word(停掉其它运行java线程)

之所以说CMS的用户体验好，是因为CMS收集器的内存回收工作是可以和用户线程一起并发执行。

总体上CMS是款优秀的收集器，但是它也有些缺点。

> 1.cms堆cpu特别敏感，cms运行线程和应用程序并发执行需要多核cpu，如果cpu核数多的话可以发挥它并发执行的优势，但是cms默认配置启动的时候垃圾线程数为 (cpu数量+3)/4，它的性能很容易受cpu核数影响，当cpu的数目少的时候比如说为为2核，如果这个时候cpu运算压力比较大，还要分一半给cms运作，这可能会很大程度的影响到计算机性能。
>
> 2.cms无法处理浮动垃圾，可能导致Concurrent Mode Failure（并发模式故障）而触发full GC
>
> 3.由于cms是采用"标记-清除“算法,因此就会存在垃圾碎片的问题，为了解决这个问题cms提供了 **-XX:+UseCMSCompactAtFullCollection**选项，这个选项相当于一个开关【默认开启】，用于CMS顶不住要进行full GC时开启内存碎片合并，内存整理的过程是无法并发的，且开启这个选项会影响性能(比如停顿时间变长)



```undefined
浮动垃圾:由于cms支持运行的时候用户线程也在运行，程序运行的时候会产生新的垃圾，这里产生的垃圾就是浮动垃圾，cms无法当次处理，得等下次才可以。
```

### G1收集器

G1(garbage first:尽可能多收垃圾，避免full gc)收集器是当前最为前沿的收集器之一(1.7以后才开始有)，同cms一样也是关注降低延迟，是用于替代cms功能更为强大的新型收集器，因为它解决了cms产生空间碎片等一系列缺陷。

> 摘自甲骨文:适用于 Java HotSpot VM 的低暂停、服务器风格的分代式垃圾回收器。G1 GC 使用并发和并行阶段实现其目标暂停时间，并保持良好的吞吐量。当 G1 GC 确定有必要进行垃圾回收时，它会先收集存活数据最少的区域（垃圾优先)
>
> g1的特别之处在于它强化了分区，弱化了分代的概念，是区域化、增量式的收集器，它不属于新生代也不属于老年代收集器。
>
> 用到的算法为标记-清理、复制算法



```css
G1垃圾收集算法是在Java6中引入，Java7中开始支持，jdk1.7,1.8的都是默认关闭的，当然在Java9中已经将G1设置为默认的垃圾收集算法了。
开启选项 -XX:+UseG1GC 
比如在tomcat的catania.sh启动参数加上
```

g1是区域化的，它将java堆内存划分为若干个大小相同的区域【region】，jvm可以设置每个region的大小(1-32m,大小得看堆内存大小，必须是2的幂),它会根据当前的堆内存分配合理的region大小。每个Region是逻辑连续的一段内存，每个Region被标记了E、S、O和H（H-当新建对象大小超过Region大小一半时，直接在新的一个或多个连续Region中分配，并标记为H）

g1通过并发(并行)标记阶段查找老年代存活对象，通过并行复制压缩存活对象【这样可以省出连续空间供大对象使用】。

g1将一组或多组区域中存活对象以增量并行的方式复制到不同区域进行压缩，从而减少堆碎片，目标是尽可能多回收堆空间【垃圾优先】，且尽可能不超出暂停目标以达到低延迟的目的。

g1提供三种垃圾回收模式 young gc、mixed gc 和 full gc,不像其它的收集器，根据区域而不是分代，新生代老年代的对象它都能回收。

几个重要的默认值，更多的查看官方文档[oracle官方g1中文文档](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.oracle.com%2Ftechnetwork%2Fcn%2Farticles%2Fjava%2Fg1gc-1984535-zhs.html)



```shell
g1是自适应的回收器，提供了若干个默认值，无需修改就可高效运作
-XX:G1HeapRegionSize=n  设置g1 region大小，不设置的话自己会根据堆大小算，目标是根据最小堆内存划分2048个区域
-XX:MaxGCPauseMillis=200 最大停顿时间 默认200毫秒
```

## Minor GC、Major GC、FULL GC、mixed gc

### Minor GC

> 在年轻代`Young space`(包括Eden区和Survivor区)中的垃圾回收称之为 Minor GC,Minor GC只会清理年轻代.

###  Major GC

> Major GC清理老年代(old GC)，但是通常也可以指和Full GC是等价，因为收集老年代的时候往往也会伴随着升级年轻代，收集整个Java堆。所以有人问的时候需问清楚它指的是full GC还是old GC。

###  Full GC

> full gc是对新生代、老年代、永久代【jdk1.8后没有这个概念了】统一的回收。
>
> 【知乎R大的回答:收集整个堆，包括young gen、old gen、perm gen（如果存在的话)、元空间(1.8及以上)等所有部分的模式】

### Mixed GC【g1特有】

> 混合GC
>
> 收集整个young gen以及部分old gen的GC。只有G1有这个模式

### 相比与 **CMS** 收集器，G1收集器两个最突出的改进是

1. 基于标记-整理算法，不产生内存碎片。

2. 可以非常精确控制停顿时间，在不牺牲吞吐量前提下，实现低停顿垃圾回收。

