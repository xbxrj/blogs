# 一、JVM基础知识
## 1.1、JVM运行时数据区
看运行时数据区图

图中对各个模块存放的内容已经有了大致的描述
### 1.1.1、方法区
1.7：常量、静态变量、类的元信息
1.8：改为元空间,String常量池移到堆中
### 1.1.2、其余的结构直接看图片中

## 1.2、JVM内存结构
看内存结构图
1. eden、survivor的比例默认8:1:1
2. 年轻代、老年代的比例默认1:2
3. 1.8之后永久代被元空间替代。元空间可以自动扩容

**问题：** 为什么需要分代？

对象的生命周期不一样，希望大部分的对象在新生代就进行了回收(为垃圾回收服务)

# 二、垃圾回收
## 2.1、什么样的对象需要被回收
### 2.1.1、判断算法
#### 2.1.1.1、引用计数法
##### 1、算法实现

给对象中添加一个引用计数器，每当一个地方引用这个对象时，计数器值+1；当引用失效时，计数器值-1，计数值为0的对象就是不可能再被使用的。
##### 2、问题
有一类情况无法回收垃圾。A、B相互引用，计数器都是大于0，无法回收。
#### 2.1.1.2、可达性分析法 （java虚拟机中使用）
##### 1、算法实现
通过GC Roots对象作为起始点，向下搜索，搜索过的路径称为引用链，当一个对象没有任何引用链时(GC Roots到对象是不可达的)，就是这个对象是“垃圾”（不再使用的），需要回收

##### 2、哪些对象是GC Roots对象
1. 虚拟机栈局部变量区中引用的对象
2. 类静态属性（方法区/元空间）引用的对象
3. 常量池（方法区/元空间）中引用的对象
4. 本地方法栈中JNI引用的对象
```
补充说明：
对象第一次被标记，会进行一次筛选，筛选条件为是否执行执行该对象的finalize()方法，如果对象覆盖了finalize()并且没有执行过的，则对象被放置到F-Queue的队列中，虚拟机自动建立的、优先级低的Finalizer线程去执行。所以对象可以在finalize()中拯救并且只能拯救自己一次。
```
### 2.1.2、引用
#### 2.1.2.1、强引用
代码中普遍存在的类似"Object obj = new Object()"这类的引用，只要强引用还存在，垃圾收集器永远不会回收掉被引用的对象。
#### 2.1.2.2、弱引用
描述有些还有用但并非必需的对象。在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围进行二次回收。如果这次回收还没有足够的内存，才会抛出内存溢出异常。Java中的类SoftReference表示软引用。
#### 2.1.2.3、软引用
描述非必需对象。被弱引用关联的对象只能生存到下一次垃圾回收之前，垃圾收集器工作之后，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。Java中的类WeakReference表示弱引用。
#### 2.1.2.4、虚引用
这个引用存在的唯一目的就是在这个对象被收集器回收时收到一个系统通知，被虚引用关联的对象，和其生存时间完全没关系。Java中的类PhantomReference表示虚引用。

## 2.2、回收算法 
算法的描述都是字面上的意思，有兴趣详细介绍可看附录中“Java垃圾回收（GC）机制详解”
### 2.2.1、标记-清除算法
第一代算法
问题：效率不高、空间碎片
### 2.2.2、标记-整理算法
### 2.2.3、复制算法
实现简单、高效，不用考虑碎片
问题：要分出一块做复制区间

## 2.3、垃圾回收器 hostpot版本实现
除CMS的垃圾回收器，其余的详细介绍可看附录中“Java垃圾回收（GC）机制详解”
### 2.3.1、Serial
### 2.3.2、ParNew
### 2.3.3、Parallel Scavenge
吞吐量优先
1. -XX:MaxGCPauseMillis 最大垃圾收集停顿时间
2. -XX:GCTimeRatio 吞吐量大小。
3. -XX:+UseAdaptiveSizePolicy 打开不需要手动指定新生代大小、Eden区和Survivor参数等细节参数，虚拟机会根据当前系统的运行情况性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量
### 2.3.4、Serial Old
### 2.3.5、Parallel Old
### 2.3.6、CMS Concurrent Mark Sweep
1. Serial Old是CMS备用预案  Concurrent Mode Failusre时使用
在并发清除的过程中，老年代没有足够空间的时候可能出现Concurrent Mode Failusre，退化成Serial Old
避免方式：[concurrent mode failure问题排查](https://blog.csdn.net/yangguosb/article/details/79857844)
2. -XX:CMSInitiatingOccupancyFraction=<value>来设置，该值代表老年代堆空间的使用率。比如，value=75意味着第一次CMS垃圾收集会在老年代被占用75%时被触发。通常CMSInitiatingOccupancyFraction的默认值为68
3. -XX:+UseCMSCompactAtFullCollection （空间碎片整理）
4. -XX:CMSFullGCsBeforeCompaction=n （n次fullGC后进行空间的碎片整理）

### 2.3.7、G1

### 2.3.8、垃圾回收器关系图
看目录下垃圾回收器关系图图片
## 2.4、JVM内存分配策略
1. 优先分配eden区
大多数情况对象在eden区分配，在eden区没有足够空间进行分配的时候，执行一次minor GC(young GC)
2. 过大的对象直接晋升老年代
-XX:PretenureSizeThreshold设置多大的对象直接进入老年代，没有默认值，只对Serial和ParNew生效。防止大对象在eden、survivor来回复制。
3. 长期存活的对象晋升老年代
如果对象在Eden出生并经过第一次Minor GC后仍然存活，并且能被Survivor容纳的话，将被移动到Survivor空间中，并且对象年龄设为1。每“熬过”一次Minor GC，年龄就增加1岁，当它的年龄增加到一定程度，就将会被晋升到老年代中。
-XX:MaxTenuringThreshold 设置年龄，最大和默认15，CMS默认6（JDK1.8是6，JDK1.7是4） 
4. 动态对象年龄判断
如果在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无须等到MaxTenuringThreshold中要求的年龄。
5. 空间分配担保
-XX:+HandlePromotionFailure 1.5以前是false
minor GC(young GC)之前，老年代最大的连续空间是否大于年轻代所有对象总空间
a. 大于：直接minor GC
b. 不大于，判断老年代最大的连续空间是否大于历次晋升到老年代的平均空间大小：
1. 大于：直接minor GC
2. 不大于：HandlePromotionFailure为false（不允许担保失败）full GC。HandlePromotionFailure为true（允许担保失败）minor GC。
如果出现了HandlePromotionFailure失败，那就只好在失败后重新发起一次Full GC
```
补充说明：
1.当空间分配担保的full gc 针对CMS是直接使用Serial Old进行回收的
2.对象分配TLAB thread local allaction  buffer/栈上分配
```

## 2.5、什么时间节点回收 ==========(回头有时间完善)============
### 2.5.1、安全点
### 2.5.2、安全域

## 2.6、如何查看当前的垃圾回收器
1. jps查看jvm进程号，jmap -heap jvm进程号查看
2. ps -ef|grep java可以查看到启动的配置信息

## 2.7、如何查看GC日志
查看附件中“Java GC 日志详解”

# 三、JDK自带监控工具

## 3.1、常用命令：jps、jstack、jmap、jstat
[排查问题](http://guafei.iteye.com/blog/1815222)
1. jps获取java进程id
2. jstack运行时栈的信息，线程的情况
3. jmap内存信息，dump使用hprof二进制形式,输出jvm的heap内容到文件
4. jstat -gcutil <pid> 1000 每1秒查看统计gc时，heap情况

### 3.1.1、CPU使用过高，怎么排查是哪行代码出现问题
1. jstack <pid> > a.txt
2. top ，shift+h 切换线程模式
3. print "%x \n" <线程id> ，获取16进制
4. vi a.txt /<16进制的线程id> 查看相应的日志信息


## 3.2、常用工具：zipkin zbaix falon

## 3.3、JVM常用配置
查看附录中“VM参数”

# 四、常见线上问题
[线上异常排查总结](https://blog.csdn.net/freeiceflame/article/details/78006812)

# 实战
[Java应用的GC优化](https://tech.meituan.com/jvm_optimize.html)
[美团Java应用的GC优化](https://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651747273&idx=1&sn=7f947064a41eeecb6816a5d0838581ae&chksm=bd12aa848a65239289d5c39264e89bd175f377f6554bfe93b37ad6498cf13deff356333c5398&mpshare=1&scene=1&srcid=1229bnD25a2zpI3DBKQxsI8T#rd)
[故障重现](https://blog.csdn.net/u010602357/article/details/54286346)


### 附录
[Java GC 日志详解](https://blog.csdn.net/wanglha/article/details/48713217)
[Java垃圾回收（GC）机制详解](https://www.cnblogs.com/xiaoxi/p/6486852.html)
[VM参数](http://www.oracle.com/technetwork/java/javase/tech/vmoptions-jsp-140102.html)
[排查问题](http://guafei.iteye.com/blog/1815222)