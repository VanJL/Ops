# Q&A

记录一些与内存有关的疑问



Q1：

pgpgin/s 和 pgpgout/s 这两个指标代表着内存的换进与换出，但是我的疑问就是这里的 in 到底是放进内存还是把内存里面的数据放进到 磁盘 中呢？

A：



Q2：

关于这个内存交换的两个指标：pgpgin/s 和 pgpgout/s 。我看好像一个时间段，基本上是一直处于 pgpgin/s > pgpgout/s 的一个状态，这是否说明内存的剩余量还比较充足够用？

就像是一个泳池，一边放水进去，一边排水出去，但是进水量大于排水量的一个状态。

A：

