# JVM

## Java监控工具

* jcmd：打印Java进程所设计的基本类、线程和VM信息
* jconsole：打开图形化试图
* jhat：读取内存堆转储
* jmap：提供堆转储和其他JVM内存使用的信息
* jinfo：查看JVM系统属性
* jstack：转储Java进程的栈信息
* jstat：GC和类装载活动信息
* jvisualvm：监视JVM的GUI工具

## JIT编译器

### 编译型

像C++这样的语言被称为编译型语言。程序以编译后的二进制形式交付：先写程序，然后用编译器静态生成二进制文件。而且这个二进制文件是针对特定的CPU的，一些新版本的CPU会加入一些新指令，而这部分指令无法在老版本CPU中执行。

### 解释型

而像PHP这样的语言被称为解释型语言。只要机器上有合适的解释器，相同的代码就可以在任何CPU上运行。执行程序时，解释器会将相应的代码转换为二进制代码。

### 即时编译

Java试图走一条中间路线。Java应用会被编译，但不是编译成特定CPU所专用的二进制代码，而是Java字节码，在代码执行时，编译器会将其编译成平台特定的二进制代码。

JVM在执行代码时，并不会立即编译代码，因为如果代码只执行一次，编译完全是浪费精力，而且解释执行会比编译后再执行的速度快。

但如果代码是经常调用的方法，或者是运行很多次迭代的循环，JVM就会对这段代码进行编译，并且对这段代码进行大量优化，即**热点编译**。

## 垃圾回收

### 分代垃圾收集器

根据不同的情况，将堆划分为不同的不同的代（老年代和新生代，而新生代又细分为Eden空间和Survivor空间）。

对象首先在新生代（Eden空间）中分配，当新生代满了之后，垃圾收集器暂停所有应用线程，回收新生代空间。不再使用的对象会被回收，仍然使用的对象会被移动到其他地方，这种操作被称为**Minor GC**。

对象不断地被移动到老年代，最终老年代的空间也会填满，JVM需要找出老年代中不再使用的对象，对它们进行回收。

直接停掉所有的应用线程，找出不再使用的对象，对其进行回收，接着对堆空间进行整理，这个过程被称为**Full GC**，通常会导致应用程序线程长时间停顿。

而通过更加复杂的计算，在应用线程运行的同时就找出不再适用的对象，采用这种方式的收集器被称为**Concurrent垃圾收集器**（CMS和G1就是采用的这种方式）。它们将应用线程停顿的时间降到了最小，也被称为低停顿（Low-Pause）收集器。如果CPU足够强劲，使用这种收集器可以避免发生Full GC，从而让任务运行的更快。

### GC算法

1. Serial垃圾收集器
   
   Serial收集器使用单线程清理堆内容，无论是进行Minor GC还是Full GC，都会暂停所有的应用线程。进行Full GC时，会对老年代的空间进行压缩整理。
   
   通过`-XX:+UseSerialGC`标志启用，指定另一种垃圾收集器来停用。

2. Throughput垃圾收集器
   
   采用多线程回收新生代空间，Minor GC的速度会比Serial收集器快得多。Full GC也能使用多线程方式。JDK7u4之后的版本可以通过`-XX:+UseParallelOldGC`来开启。因为使用多线程，所以也常常被称为Parallel收集器。
   
   Minor GC和Full GC时会暂停所有的应用线程，同时在Full GC时对老年代空间进行压缩整理。
   
   它已经是默认的收集器，如有需要，可以通过`-XX:+UseParallelGC`、`-XX:+UseParallelOldGC`来手动开启。

3. CMS收集器
   
   使用新的算法来收集新生代对象（`-XX:+UseParNewGC`）。
   
   Full GC不再暂停应用程序，而是使用若干个后台线程定期对老年代空间进行扫描，及时回收不再使用的对象。因为不再进行整理操作，所以堆会逐渐变得碎片化。如果CMS的后台线程无法获得完成它们任务搜需要的CPU资源，或者堆变得过度碎片化以至于无法找到连续空间分配对象，CMS就蜕化到Serial收集器行为，之后又恢复到并发运行，再次启动后台线程，直到下一次变得过度碎片化。
   
   通过`-XX:+UseConcMarkSweepGC`、`-XX:+UseParNewGC`可以启用CMS垃圾收集器。

4. G1垃圾收集器
   
   设计初衷是为了尽量缩短处理超大堆（大于4GB）时产生的停顿。
   
   G1收集算法将堆划分为若干个区域（依旧属于分代收集器），老年代的垃圾手机工作由后台线程完成，大多数工作不需要暂停应用线程。由于老年代被划分到不同的区域，G1收集器通过将对象从一个区域复制到另一个区域，完成对象的清理工作，这意味着在正常的处理过程中，G1收集器完成了（一部分）堆的压缩整理。所以使用G1收集器不大容易发生堆大量碎片化。
   
   通过`-XX:+UseG1GC`启动G1垃圾收集器。

**触发及禁用显示的垃圾收集**

通过`System.gc()`可以主动触发垃圾回收（Full GC），但这不是一个好的操作，尽量要避免（性能监控或基准测试除外）。可以通过`-XX:+DisableExplicitGC`显示地禁止这种类型的GC。

RMI作为分布式垃圾收集器的一部分，每隔一个小时它会调用`System.gc()`一次，可以通过设置`-Dsun.rmi.dgc.server.gcInterval=N`和`-Dsun.rmi.dgc.client.gcInterval=N`进行修改。N值的单位以毫秒记。

## GC调优

### 基础指令

**-Xms**

初始堆大小

**-Xmx**

最大堆大小

**-XX:NewRatio=N**

设置新生代与老年代的空间占用比例

**-XX:NewSize=N**

设置新生代的初始大小

**-XX:MaxNewSize=N**

设置新生代空间的最大大小

**-XmnN**

将NewSize和MaxNewSize设定为同一个值的快捷方法

> `Initial Young Gen Size = Initial Heap Size / (1 + NewRatio)`
> 
> 默认情况下，新生代空间大小是初始堆的33%

### 永久代和元空间

**-XX:PermSize=N**

永久代初始大小

**-XX:MaxPermSize=N**

永久代最大大小

**-XX:MetaspaceSize=N**

元空间初始大小

**-XX:MaxMetaspaceSize=N**

元空间最大大小

> 永久代和元空间没有保存类实例的具体信息，也没有反射对象，这里保存的是一些类的元数据，只是一些“书签”信息。
> 
> 永久代并不意味着其中的数据就会永久保存，当Full GC时，老的元数据依旧会被丢弃回收。

### 控制并发

**-XX:ParallelGCThreads=N**

控制多线程垃圾回收器启动的线程数

> 以下参数会影响线程的数目：
> 
> * 使用`-XX:+UseParallelGC`、`-XX:+UseParNewGC`、`-XX:+UseG1GC`收集新生代空间
> 
> * 使用`-XX:+UseParallelOldGC`收集老年代空间
> 
> * CMS和G1收集器的“时空停顿”阶段（非Full GC）

### 垃圾回收

> 使用jstat工具查看gc信息。
> 
> 通过`jstat -options`列出所有选项
> 
> `jstat -gcutil process_id 1000`

### Throughput收集器

**-XX:MaxGCPauseMillis=N**

设定应用可承受的最大停顿时间

> 设定该标志的值，会影响Minor GC和Full GC，如果设置的值非常小，那么应用老年代最终就会非常小，会触发非常频繁的Full GC，对应用程序的性能将是灾难性的影响。缺省情况下，我们不设定该参数。

**-XX:GCTimeRatio=N**

设置程序在垃圾回收上花费多少时间

> ***ThroughputGoal* = 1 - $\frac {1}{1 + GCTimeRatio}$**
> 
> GCTimeRatiod的默认值是99，上述公式得出的解是0.99，表示只有1%的时间消耗在垃圾回收上。
> 
> 通过以下公式计算期望应用程序线程工作的时间：
> 
> ***GCTimeRatio* = $\frac {Throughput}{1 - Throughput}$**
> 
> 比如期望95%，带入后求得的GCTimeRatio是19。

### CMS收集器

#### 回收新生代空间

当堆的占用达到某个程度时，JVM会启动后台线程扫描堆，回收不用的对象，老年代空间不会进行压缩整理，会产生很多空间碎片。

##### 并发回收

1. 初始标记（CMS-initial-mark）：会暂停所有的应用程序线程。

2. 标记阶段（CMS-concurrent-mark-start）：应用程序线程可以持续运行，不会中断，这个阶段仅仅是标记，不会对堆的使用情况产生实质性的改变。

3. 预清理（CMS-concurrent-preclean-start）：与应用程序线程并发进行。

4. 重新标记（CMS-concurrent-abortable-preclean-start）（可中断预清理）：标记阶段所有的应用线程都会被暂停，如果新生代收集刚刚结束，紧接着又是一个标记阶段的话，应用线程就会遭遇2次连续的停顿操作，使用可中断预清理阶段就是希望尽量缩短停顿的长度，避免连续停顿。

5. 清除（CMS-concurrent-sweep-start）：回收线程与应用程序线程并发运行。

6. 并发重置（CMS-concurrent-reset-start）：并发运行的最后一个阶段，CMS垃圾回收的周期至此告终，老年代空间没有被引用的对象被回收。

以上是CMS回收的正常情况，但是现实世界并没有这么简单，我们还要额外查看另外的三种信息，出现这些信息表示垃圾收集器遇到了麻烦。

1. 并发模式失效（concurrent mode failure）：新生代发生垃圾回收，但是老年代又没有足够的空间容纳晋升的对象时，CMS就会退化成Full GC，所有的应用线程都会被暂停。

2. （promotion failed）老年代有足够的空间可以容纳晋升的对象，但是由于空闲空间的碎片化，导致晋升失败。

3. 只有一条Full GC的记录，不含任何常规并发垃圾回收的标志：永久代空间用尽，需要回收，Java8中，如果元空间需要调整，也会发生同样的情况。

#### 针对并发模式失效的调优

1. 给后台线程更多的运行机会
   
   更早地启动并发收集周期，比如在老年代空间占用达到60%时启动并发周期，会比70%完成垃圾收集的几率更大。
   
   `-XX:CMSInitiatingOccupancyFraction=N`和`-XX:+UseCMSInitiatingOccupancyOnly=N`，同时使用这两个参数能帮助CMS更容易地进行决策：如果同时设置这两个标志，那么CMS就只依据设置的老年代空间占用率来决定何时启动后台线程。
   
   如果开启了`UseCMSInitiatingOccupancyOnly`，`CMSInitiatingOccupancyFraction`的默认值就被置为70，即CMS会在老年代空间占用达到70%时启动并发收集周期。

2. 调整CMS后台线程
   
   每个CMS后台线程都会100%占用一颗CPU，如果并发模式失效，同时又有额外的CPU周期可用，可以设置`-XX:ConcGCThreads=N`，增加后台线程的数目。
   
   >  `ConcGCThreads = (3 + ParallelGCThreads) / 4`
   
   如果`ConcGCThreads`设置的偏大，垃圾收集会占用本来能用于运行应用程序的CPU周期，导致应用程序些微的停顿。
   
   而在一些配备了大量CPU的系统上，如果没有出现频繁的并发模式失败，可以考虑减少后台线程数，释放这部分CPU周期用于运行应用程序线程。

#### 永久代调优

Java7中的CMS垃圾收集线程不会处理永久代中的垃圾，如果永久代空间用尽，CMS会发起一次Full GC来回收其中的垃圾对象。

还可以开启`-XX:+CMSPermGenSweepingEnabled`标志（默认为false），开启后，永久代中的垃圾使用与老年代同样的方式进行垃圾收集：通过一组后台线程并发地回收永久代中的垃圾对象。`-XX:CMSInitiatingPermOccupancyFraction=N `参数可以指定回收空间占比，默认值为80%。

同时，我们还需要设置`-XX:+CMSClassUnloadingEnabled`，否则，永久代垃圾回收也只能释放少量的无效对象，类的元数据并不会被释放。

#### 增量式CMS垃圾收集

**在Java8中已经不推荐使用，在Java9中移除。**

### G1垃圾收集器

>  **首先收集垃圾最多的分区**

主要有4种情况会触发Full GC，这些日志是一个信号，我们需要进一步调优才能提升性能：

1. 并发模式失效
   
   G1垃圾收集启动标记周期，但老年代在周期完成之前就被填满了，这种情况下，G1收集器会放弃标记周期。
   
   ```
   51.408: [GC concurrent-mark-start]
   65.473: [Full GC 4095M->1395M(4096M), 6.1963770 secs]
    [Times: user=7.87 sys=0.00, real=6.20 secs]
   71.669: [GC concurrent-mark-abort]
   ```
   
   发生这种失败意味着堆的大小应该增加了，或者收集器的后台处理应该更早开始，或是需要调整周期，让它运行得更快。

2. 晋升失败
   
   G1收集器完成了标记阶段，开始启动混合式垃圾回收，清理老年代的分区，不过，老年代空间在垃圾回收释放出足够内存之前就会被耗尽。日志中，这种情况的现象通常是混合式GC之后紧接着一次Full GC。
   
   ```
   2226.224: [GC pause (mixed)
           2226.440: [SoftReference, 0 refs, 0.0000060 secs]
           2226.441: [WeakReference, 0 refs, 0.0000020 secs]
           2226.441: [FinalReference, 0 refs, 0.0000010 secs]
           2226.441: [PhantomReference, 0 refs, 0.0000010 secs]
           2226.441: [JNI Weak Reference, 0.0000030 secs]
                   (to-space exhausted), 0.2390040 secs]
   ....
       [Eden: 0.0B(400.0M)->0.0B(400.0M)
           Survivors: 0.0B->0.0B Heap: 2006.4M(2048.0M)->2006.4M(2048.0M)]
       [Times: user=1.70 sys=0.04, real=0.26 secs]
   2226.510: [Full GC (Allocation Failure)
           2227.519: [SoftReference, 4329 refs, 0.0005520 secs]
           2227.520: [WeakReference, 12646 refs, 0.0010510 secs]
           2227.521: [FinalReference, 7538 refs, 0.0005660 secs]
           2227.521: [PhantomReference, 168 refs, 0.0000120 secs]
           2227.521: [JNI Weak Reference, 0.0000020 secs]
                   2006M->907M(2048M), 4.1615450 secs]
       [Times: user=6.76 sys=0.01, real=4.16 secs]
   ```
   
   这种失败通常意味着混合式收集需要更快速地完成垃圾收集；每次新生代垃圾收集需要处理更多老年代的分区。

3. 疏散失败
   
   进行新生代垃圾收集时，Survivor空间和老年代中没有足够的空间容纳所有的幸存对象。
   
   ```
   60.238: [GC pause (young) (to-space overflow), 0.41546900 secs]
   ```
   
   这条日志表明堆已经几乎完全用尽或者碎片化了。解决这个问题最简单的方式是增加堆的大小。

4. 巨型对象分配失败
   
   目前没有工具可以很方便地专门诊断这种类型的失败，尤其是从标准垃圾收集日志中进行诊断。如果发生了莫名其妙的Full GC，可以排查一下是否是因为分配巨型对象导致的。

#### G1垃圾收集器调优

G1垃圾收集器调优的目标之一是尽量简单，所以最主要的调优只通过`-XX:MaxGCPauseMillis=N`。使用G1垃圾收集器时，该标志有一个默认值：200毫秒，如果G1收集器发生时空停顿的时长超过该值，G1收集器就会尝试各种方式进行弥补。

如果减少了参数值，为了达到停顿时间的目标，新生代的大小会相应减小，不过新生代垃圾收集的频率会更加频繁。除此之外，混合式GC收集的老年代分区数也会减少，而这会增大并发模式失败发生的机会。

1. 调整G1垃圾收集的后台线程数
   
   `ConcGCThreads = (ParallelGCThreads + 2) / 4`

2. 调整G1垃圾收集器运行的频率
   
   G1垃圾收集周期通常在堆的占用达到参数` -XX:InitiatingHeapOccupancyPercent=N `设定的比率时启动，默认情况下该参数的值为45。
   
   如果这个值设置的过高，应用程序会频繁的Full GC，如果设置的过低，应用程序又会以超过实际需要的节奏进行大量的后台处理。
   
   并发周期结束之后，检查下堆的大小，确保`InitiatingHeapOccupancyPercent`的值大于此时堆的大小。

3. 调整G1收集器的混合式垃圾收集周期
   
   第一个因素是有多少分区被发现大部分是垃圾对象。通过`-XX:G1MixedGCLiveThresholdPercent=N`来对调节分区的垃圾占用比，如果分区的垃圾占用比达到配置，这个分区就被标记为可以进行垃圾回收。
   
   第二个因素是G1垃圾收集回收分区时最大混合式GC周期数，通过参数`-XX:G1MixedGCCountTarget=N`进行调节。默认值为8，减少该值可以解决晋升失败的问题，代价是混合式GC周期的停顿时间会更长。如果混合式GC的停顿时间过长，可以增大这个参数的值，减少每次混合式GC周期的工作量。不过调整之前我们需要确保增大值之后不会对下一次G1并发周期带来太大的延迟，否则可能会导致并发模式失败。
   
   第三个因素是GC停顿可忍受的最大时长，通过`MaxGCPauseMillis`参数设定，增大该参数值，能在每次混合式GC中收集更多的老年代分区，而这反过来又能帮助G1收集器在更早的时候启动并发周期。

## 高级调优

### 晋升及Survivor空间

首次新生代垃圾收集时，对象被从Eden空间移动到Survivor0，紧接着的下一次垃圾收集中，活跃对象会从Survivor0和Eden空间移动到Survivor1，这之后，Eden空间和Survivor0空间被完全清空。下一次的垃圾回收会将活跃对象从Survivor1和Eden移回到Survivor0。

如果新生代收集时，目标Survivor空间被填满，Eden空间剩下的活跃对象会直接进入老年代，或者对象在Survivor空间经历的GC周期数超过了上限，这些超过上限的对象也会被移动到老年代。这个上限值被称为晋升阈值，可以通过`-XX:InitialTenuringThreshold=N`标志来设置初始的晋升阈值，JVM最终会在1和最大晋升阈值（`-XX:MaxTenuringThreshold=N`）之间选择一个合适的值。

Survivor空间的初始大小由`-XX:InitialSurvivorRatio=N`标志决定。

```
survivor_space_size = new_size / (initial_survivor_ratio + 2)
```

JVM可以增大Survivor空间的大小直到其最大上限（`-XX:MinSurvivorRatio=N`）。

```
maximum_survivor_space_size = new_size / (min_survivor_ratio + 2)
```

这个参数值是分母，值越小，Survivor空间的容量最大。

为了保持Survivor空间的大小为某个固定值，我们可以使用`SurvivorRatio`参数，同时关闭`UseAdaptiveSizePolicy`（需要注意的是，关闭自适应调节大小会同时影响新生代和老年代）。

JVM依据垃圾回收之后Survivor空间的占用情况判断是否需要增加或减少Survivor空间的大小。通过标志`-XX:TargetSurvivorRatio=N`设置，默认为50%。

避免因为Survivor空间过小而导致对象直接晋升到老年代，如果大量的短期对象最终填满老年代，会导致频繁的Full GC。

如果增大Survivor空间的大小，内存由新生代的Eden空间划分到Survivor空间，会导致在Minor GC之前能分配的对象数目会更少。因此不推荐采用这种方式。

另一种可能是增大新生代的大小，这会导致老年代空间变得更小，应用程序可能会频繁发生Full GC。

推荐先增大堆的大小，同时减小存活率（这种调优适合其大多数对象在几个GC周期之后就不再存活）。

## 堆内存最佳实践

### 堆分析

##### 打印堆直方图

`jmap -histo process_id`

> 包含被回收的对象

`jcmd port GC.class_histogram`

`jmap -histo:live process_id`

> 触发一次Full GC，不包含被回收的对象

##### 堆转储

`jcmd process_id GC.heap_dump /path/to/heap_dump.hprof`

`jmap -dump:live,file=/path/to/heap_dump.hprof process_id`

然后通过几种工具查看转储出来的hprof文件：jhat/jvisualvm/mat

### 内存溢出错误

1. 原生内存不足
   
   如果Error消息中提到了原生内存的分配，那对堆进行调优解决不了问题，需要看一下错误中提到的是那种原生内存问题。

2. 永久代或元空间内存不足
   
   应用使用的类太多：增加永久代的大小
   
   类加载器的内存泄漏：如果编写的应用会创建并丢弃大量类加载器，要确保类加载器本身能正确丢弃，尤其要确保没有线程将其上下文加载器设置成一个临时的类加载器。

3. 堆内存不足
   
   增加堆的大小，同时检查程序是否存在内存泄漏：持续分配新对象，却没有让其他对象退出作用域。
   
   `-XX:+HeapDumpOnOutOfMemoryError`：打开自动堆转储
   
   `-XX:HeapDumpPath=<path>`：设置堆转储文件写入的位置
   
   `-XX:+HeapDumpAfterFullGC`：在运行一次Full GC后生成一个堆转储文件
   
   `-XX:+HeapDumpBeforeFullGC`：在运行一次Full GC之前生成一个堆转储文件

4. 达到GC的开销限制
   
   JVM认为在执行GC上花费了太多时间。***必须同时满足下列所有条件时***，就会抛出该错误：
   
   1. 花费在Full GC上的时间超出了`-XX:GCTimeLimit=N`
   
   2. 一次Full GC回收的内存量少于`-XX:GCHeapFreeLimit=N`
   
   3. 上面两个条件连续5次Full GC都成立（如果连续4次Full GC周期都成立，JVM在第五次时会释放所有的软引用）
   
   4. `-XX:+UseGCOverhead-Limit`标记的值为true（默认为true）

### 减少内存使用

1. 减少对象大小

2. 延迟初始化（需要线程安全的对象，应该使用双重检查锁）

3. 尽早清理：在一个方法中创建局部变量；对于一个长期存活的类会缓存以及丢弃对象引用，一定要仔细检查，需要时应该手动将引用设置为null

## 线程与同步的性能

### 线程同步

#### 同步的代码

$Speedup = \frac {1}{(1-P)+\frac {P}{N}}$

* P是程序并行运行部分所花的时间

* N是所用到的线程数（假定每个线程总有CPU可用）

从这个等式中就可以看出，随着P值的降低（也就是有更多的代码是串行执行的），引入多个线程所带来的性能优势也会随之下降。

**锁定对象的开销**

如果某个锁没有被争用，那这个方面的开销会相当小。synchronized关键字和CAS指令之间有轻微的差别。非竞争的synchronized锁被称为非膨胀锁，获取非膨胀锁的开销在几百纳秒的数量级，CAS则更小。

如果存在竞争，开销会更高。当存在多个线程访问某个同步锁时，这个锁会变成膨胀的。随着线程数增加，成本也会增加，但每个线程所花的时间是固定的。而对于CAS指令的代码，竞争时的开销是无法预测的。

**避免同步**

* 在每个线程中使用不同的对象

* 使用基于CAS的替代方案

**指导原则**

* 如果访问的不是存在竞争的资源，那么基于CAS的保护要稍快于传统的同步。

* 如果访问的资源存在轻度或适度的竞争，那么基于CAS的保护要快于传统的同步。

* 随着所访问资源的竞争越来越剧烈，在某一时刻，传统的同步就会成为更高效的选择（这只会出现在运行着大量线程的非常大型的机器上）。

* 当被保护的值有多个读取，但不会被写入时，基于CAS的保护不会受竞争的影响。