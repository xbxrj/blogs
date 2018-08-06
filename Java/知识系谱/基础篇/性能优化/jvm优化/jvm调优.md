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
问题：为什么需要分代，对象的生命周期不一样，希望大部分的对象在新生代就进行了回收(为垃圾回收服务)
1. eden、survivor的比例默认8:1:1
2. 年轻代、老年代的比例默认1:2
3. 1.8之后永久代被元空间替代。元空间可以自动扩容

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
4. 本地方法栈中引用的对象
## 回收算法

## 垃圾回收器 hostpot版本实现

## JVM内存分配策略
1. 优先分配eden区
大多数情况对象在eden区分配，在eden区没有足够空间进行分配的时候，执行一次minor GC(young GC)
2. 过大的对象直接晋升老年代

-XX:PretenureSizeThreshold
3. 长期存活的对象晋升老年代
4. 动态对象年龄判断
5. 空间分配担保
补充说明：
对象分配TLAB thread local allaction  buffer



### 垃圾回收器关系图

## 什么时间节点回收

## 如何查看当前的垃圾回收器

## 如何查看GC日志

# JDK自带监控工具

## 常用命令：JStack、JMap、jps、[排查问题](http://guafei.iteye.com/blog/1815222)

## 常用工具：zipkin zbaix falon

## JVM常用配置

# 常见线上问题

# 实战
[Java应用的GC优化](https://tech.meituan.com/jvm_optimize.html)
[美团Java应用的GC优化](https://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651747273&idx=1&sn=7f947064a41eeecb6816a5d0838581ae&chksm=bd12aa848a65239289d5c39264e89bd175f377f6554bfe93b37ad6498cf13deff356333c5398&mpshare=1&scene=1&srcid=1229bnD25a2zpI3DBKQxsI8T#rd)
[故障重现](https://blog.csdn.net/u010602357/article/details/54286346)
[Java GC 日志详解](https://blog.csdn.net/wanglha/article/details/48713217)

### 附录
[VM参数](http://www.oracle.com/technetwork/java/javase/tech/vmoptions-jsp-140102.html)