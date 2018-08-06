# ConrrentHashMap实现原理
1.7 segment 分段锁
1.8 直接锁Node，put直接时候使用Synchronized，说明Synchronized性能提升后CAS性能相差无几
size计算：
1.7：
先采用不加锁的方式，连续计算元素的个数，最多计算3次：
1、如果前后两次计算结果相同，则说明计算出来的元素个数是准确的；
2、如果前后两次计算结果都不同，则给每个Segment进行加锁，再计算一次元素的个数；

1.8：
通过累加baseCount和CounterCell数组中的数量，即可得到元素的总个数；


http://www.importnew.com/23610.html
